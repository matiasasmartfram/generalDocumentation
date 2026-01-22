---
layout: default
title: "Documentación: Gestión Manual de Estado de Sucursales"
subtitle: "Documentación: Consulta y Gestion Manual de Estado de Tiendas en BO"
---

## **1. Funcionamiento de Endpoints e Interfaz de Usuario**

Esta sección describe el propósito de la funcionalidad, cómo interactúa el usuario con la nueva interfaz y cómo esta se comunica con los endpoints del backend.

### **1.1 Resumen Ejecutivo**

La nueva funcionalidad ofrece un control manual y en tiempo real sobre el estado de disponibilidad (Abierto/Cerrado) de las sucursales en las plataformas de delivery **PedidosYa, Uber Eats, Rappi**. Permite a los operadores consultar el estado actual y forzar la apertura de una sucursal.

### **1.2 Interfaz de Usuario y Simulación**

Para gestionar el estado, se ha añadido una nueva sección en la página de detalles de cada sucursal. A continuación se muestra una recreación interactiva de esta interfaz.

#### **Plataformas \***

| Plataforma | Acción Global | Status |
| :--- | :--- | :--- |
| **Global** | `Enviar Open a todos` | `Actualizar TODO` |

| Plataforma | Acciones | Estado |
| :--- | :--- | :--- |
| **Rappi** | `Open Rappi` | `Error` + `Refresh` |
| **Toque** | — | — |
| **I+D** | — | — |
| **PedidosYa** | `Open PedidosYa` | `Off` + `Refresh` |
| **PediGrido** | — | — |
| **UberEats** | `Open UberEats` | `Error` + `Refresh` |

### **1.3 Análisis de Endpoints**

#### **Endpoint de Consulta: `POST /api/branches/getStatus`**

Se utiliza para verificar y devolver el estado de disponibilidad (abierto o cerrado) de una o más sucursales en las plataformas especificadas. Se usa el método `POST` para permitir el envío de un array de consultas en el cuerpo de la solicitud.

**Request Body:**

```json
[
  { "platform": 1, "branchId": 123 },
  { "platform": 2, "branchId": 123 }
]
```

**Response Body:**

```json
[
  { "platform": 1, "closed": false },
  { "platform": 2, "closed": true }
]
```

#### **Endpoint de Acción: `PUT /api/branches/sendOpen`**

Se utiliza para enviar una solicitud explícita a una plataforma para que marque una sucursal como "Abierta" u "Online". El cuerpo de la solicitud es idéntico al de `getStatus`.

## **2. Cambios Técnicos en el Concentrador (Backend)**

Esta sección profundiza en la arquitectura y el flujo de código de las nuevas funciones implementadas en el controlador `api/src/controllers/branch.js`.

### **2.1 Estructura y Patrones Comunes**

Ambas funciones, `getStatus` y `sendOpen`, comparten una estructura de base común diseñada para ser robusta y escalable:

*   **Declaración Asíncrona:** Son funciones `async`, permitiendo el uso de `await` para manejar operaciones de I/O de forma limpia.
*   **Carga de Configuración Dinámica:** Al inicio, cargan desde la base de datos la configuración de cada plataforma, evitando configuraciones hardcodeadas.
*   **Gestión Centralizada de Autenticación:** Los tokens se obtienen de variables globales gestionadas por procesos de fondo (cron jobs).
*   **Procesamiento en Bucle:** Iteran sobre el array `branchForms` para procesar múltiples consultas en una sola llamada.
*   **Manejo de Errores Detallado:** Cada llamada a una API externa está envuelta en un bloque `try...catch` para asegurar la resiliencia.

### **2.2 Implementación de `getStatus()`**

<div class="note">
  <p><strong>Objetivo:</strong> Consultar y devolver el estado de disponibilidad de una sucursal en una o varias plataformas.</p>
</div>

```javascript
// api/src/controllers/branch.js

const getStatus = async (req, res) => {
  try {
    // 1. Inicialización y carga de configuraciones
    let result = [];
    let peya = await platforms.findOne({ name: 'PedidosYa' });
    let uber = await platforms.findOne({ name: 'UberEats' });
    let rappi = await platforms.findOne({ name: 'Rappi' });
    // ... (Definición de headers y URLs base)

    let branchForms = req.body;
    let branch = await model.findOne({ branchId: branchForms[0].branchId });

    // 2. Iteración sobre las consultas solicitadas
    for (const branchForm of branchForms) { 
      try {
        // 3. Lógica específica por plataforma
        if (branchForm.platform === 1) { // PedidosYa
          // ... (Lógica para obtener el identificador y construir la URL)
          const statuspos = await axios.get(urlAvailability, headersConfig3);   
          if(statuspos.data[0].availabilityState === "CLOSED"){
            result.push({platform: 1, closed: true });   
          } else {
            result.push({platform: 1, closed : false });
          }
        }
        // ... (Bloques análogos para UberEats y Rappi)
      } catch (error) {
        result.push({platform: branchForm.platform, closed : "Error: "+ error?.message });
        Log.saveError(error, { /* ... contexto del error ... */ });               
      }      
    }
    // 4. Envío de la respuesta final
    return res.status(200).json(result).end();
  } catch (error) { /* ... */ }
};
```

### **2.3 Implementación de `sendOpen()`**

<div class="note">
  <p><strong>Objetivo:</strong> Enviar una solicitud explícita a una plataforma para marcar una sucursal como abierta.</p>
</div>

```javascript
// api/src/controllers/branch.js

const sendOpen = async (req, res) => {
  try {
    // 1. Inicialización y carga de configuraciones (similar a getStatus)
    // ...

    // 2. Iteración sobre las acciones solicitadas
    for (const branchForm of branchForms) {
      // 3. Lógica específica por plataforma
      if (branchForm.platform === 1) { // PedidosYa
        // 3a. Obtener datos necesarios para la actualización (leer para escribir)
        const statuspos = await axios.get(urlAvailability, headersConfig3);
        let body = {
          "availabilityState": "OPEN",
          "platformKey": statuspos.data[0].platformKey,
          // ...
        };
        // 3b. Enviar la actualización
        await axios.put(urlAvailability, body, headersConfig3);
      }

      if (branchForm.platform === 4) { // UberEats
        let body = { "status": "ONLINE", "reason": "STORE OPEN" };
        await axios.post(urlAvailability, body, headersConfig);
      }

      if (branchForm.platform === 2) { // Rappi
        let body = { "stores": [rappiNumber], "is_enabled": true };
        await axios.put(url, body, options);
      }
    }
    // 4. Envío de la respuesta final
    return res.status(200).json({}).end();
  } catch (error) { /* ... */ }
};
```

## **3. Cambios Técnicos en el BackOffice (Frontend)**

Esta sección detalla los cambios realizados en el código de Angular para integrar la nueva interfaz y su lógica.

### **3.1 Archivo de Servicio: `branches.service.ts`**

<div class="note">
  <p><strong>Propósito:</strong> Se añaden los métodos para comunicarse con los nuevos endpoints del backend.</p>
</div>

```typescript
// ...
   public getBranch(id: number): Observable<Branch> { /* ... */ }
 
+  public sendOpen(branchForms: any): Observable<any> {
+    return this.http.put<any>(environment.apiUrl + `/branches/sendOpen`, branchForms);  
+  }
+
+  public getStatus(branchForms: any): Observable<any> {
+    return this.http.post<any>(environment.apiUrl + `/branches/getStatus`, branchForms);
+  }
+
   public getBranches(): Observable<Branch[]> { /* ... */ }
// ...
```

*   **`sendOpen(...)`:** Nuevo método que realiza una petición `PUT` a `/branches/sendOpen` para enviar la orden de apertura.
*   **`getStatus(...)`:** Nuevo método que realiza una petición `POST` a `/branches/getStatus` para solicitar el estado actual de las plataformas.

### **3.2 Archivo de Lógica del Componente: `branch.component.ts`**

<div class="note">
  <p><strong>Propósito:</strong> Se añade toda la lógica para manejar el estado en la interfaz, llamar a los servicios y actualizar la vista.</p>
</div>

#### **Nuevas propiedades de la clase**

```typescript
// ...
   public rappiActive: boolean;
+  public platformsOpenClosed = ["Rappi","UberEats","PedidosYa"]
+  public platformMap = { 1: 'PedidosYa', 2: 'Rappi', 4: 'UberEats' };
+  public branchForms: { [platformName: string]: { closed: boolean | string } } = {};
// ...
```

*   **`platformsOpenClosed`:** Array que define qué plataformas son compatibles con esta funcionalidad.
*   **`platformMap`:** Objeto para traducir IDs de plataforma a nombres legibles.
*   **`branchForms`:** Objeto que almacenará dinámicamente el estado de cada plataforma para que la vista lo pueda leer.

#### **Llamada inicial en `ngOnInit`**

```typescript
// ...
   this.isLoaded = Promise.resolve(false);
   this.generateNewsDashboard(1, branch);
+  this.getStatus(this.dataPlatform);
// ...
```

*   Se añade una llamada a `this.getStatus()` para que, al cargar la página, se obtenga inmediatamente el estado de todas las plataformas.

#### **Nuevos Métodos**

*   **`sendOpen(platforms)` y `sendOpenFilter(platformName)`:** Lógica para preparar y enviar la solicitud de apertura al servicio, ya sea para todas las plataformas o para una sola.
*   **`getStatus(platforms)` y `getStatusFilter(platformName)`:** Lógica para preparar y enviar la solicitud de estado. Al recibir la respuesta, itera sobre los resultados y actualiza el objeto `this.branchForms`, lo que provoca que la interfaz gráfica se actualice con los indicadores de estado correctos.

### **3.3 Archivo de Plantilla Visual: `branch.component.html`**

<div class="note">
  <p><strong>Propósito:</strong> Se añaden los nuevos botones e indicadores visuales a la interfaz de usuario. (Punto 1.2)</p>
</div>
