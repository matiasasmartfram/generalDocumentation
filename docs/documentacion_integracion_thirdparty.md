---
layout: default
title: Documento de Integración con Terceros
subtitle: Integración con Plataformas de Pedidos - SmartPedidos - Smartfran
---

## **Historial de Revisiones del Documento**

| Versión | Fecha | Modificado por | Descripción de Cambios |
| --- | --- | --- | --- |
| 0.0.1 | 15/08/2019 | A Kraupl | Versión inicial |
| 0.0.2 | 04/09/2019 | A Kraupl | Se agregan atributos de promociones al detalle del pedido. |
| 0.0.3 | 09/09/2019 | A. Kraupl | Correción del body Post Orders. Agrego unitPrice a la promoción. |
| 0.0.4 | 19/11/2019 | A. Kraupl | Respuestas de error. Nuevo BaseUrl. |
| 0.0.5 | 01/04/2020 | A.Kraupl | Se agrega campo `partial` al payment. Se agrega `email` y `dni` al user. |
| 0.0.6 | 06/07/2021 | F. Velez | Se agrega response para las nuevas plataformas. |
| 0.0.7 | 29/12/2023 | O. Joel | Actualización general. |
| 0.0.8 | 22/09/2025 | A. Matias | Eliminacion del apartado Busqueda de Datos (Deprecado). Actualizaciones Generales |

## **Tabla de Contenidos**

*   [Introducción](#intro)
*   [1. Datos de Configuración](#config)
*   [2. Proceso](#process)
*   [3. Peticiones (API SmartPedidos)](#requests)
*   [4. Callbacks](#responses)
*   [5. Solucion de Problemas](#problems)

---

## **Introducción**

Este documento contiene información referida a la integración de terceros como plataformas generadoras de pedidos hacia SmartPedidos. Contiene los endpoints necesarios para gestionar todo el tráfico desde la generación de un nuevo pedido hasta la gestión de sus estados.

---

## **Operación de ThirdParties**

### **1. Datos de Configuración**

*   **THIRD_PARTY_ID:** SOLICITAR
*   **THIRD_PARTY_SECRET:** SOLICITAR
*   **URL:** `{urlstg}/api/thirdParties`

### **2. Proceso**

Para poder operar con el servicio de gestión de pedidos de SmartFran, la plataforma tercera deberá loguearse con los datos de configuración, obtener un token y, posteriormente, realizar peticiones a los endpoints protegidos con dicho token. Las órdenes a ingresar deben indicar la sucursal (`branchId`) a la que será enviado el pedido.

### **3. Peticiones (API SmartPedidos)**

Las siguientes son las peticiones que nuestra API permite:

#### **3.1. Login**

El login es obligatorio para hacer cualquier otro tipo de petición. El token expira en 24 horas.

| **Endpoint** | `/login` |
| --- | --- |
| **Método** | `POST` |

**Body**

```json
{
  "thirdPartyId": "THIRD_PARTY_ID",
  "thirdPartySecret": "THIRD_PARTY_SECRET"
}
```

**Respuesta Exitosa (200 OK)**

```json
{
  "data": {
    "name": "THIRD_PARTY_NAME",
    "id": "Id del third party en el sistema de SmartFran"
  },
  "accessToken": "ACCESS_TOKEN"
}
```

**Respuesta de Error (401 Unauthorized)**

```json
{
  "message": "Login failed.",
  "error": "Login failed."
}
```

#### **3.2. Crear Órdenes**

Permite enviar órdenes a las sucursales, siempre que el cuerpo de la petición siga el formato especificado. El diccionario de campos ayuda a construir el body correctamente.

| **Endpoint** | `/orders` |
| --- | --- |
| **Método** | `POST` |
| **Header** | `Authorization: Bearer ACCESS_TOKEN` |

**Body de Ejemplo**

```json
[{
  "id": "62743329",
  "state": "PENDING",
  "preOrder": false,
  "registeredDate": "2019-07-23T16:42:32Z",
  "deliveryDate": "2019-07-23T17:19:32Z",
  "pickup": false,
  "pickupDate": null,
  "notes": "observacion",
  "logistics": false,
  "user": {
    "platform": "thirdParty",
    "name": "Daley",
    "lastName": "Paley",
    "id": 10455712,
    "email": "prueba@gmail.com",
    "dni": "99999999"
  },
  "address": {
    "description": "Obispo trejo 1420 esquina rew qwre",
    "phone": "4324232"
  },
  "details": [{
    "id": "4568487",
    "unitPrice": 220,
    "quantity": 1,
    "discount": 0,
    "name": "Promo Helado 2 Kilos por 1",
    "sku": "4568487",
    "notes": "Nota de sobre el producto.",
    "promotion": true,
    "optionGroups": [{
      "id": 49,
      "name": "Primer kilo",
      "sku": "64",
      "unitPrice": "110",
      "notes": "Granizado: 1 - Dulce de leche: 1 - Banana: 1 - Limon: 1"
    }, {
      "id": 50,
      "name": "Segundo kilo",
      "sku": "64",
      "unitPrice": "110",
      "notes": "Chocolate: 3 - Vainilla: 1"
    }]
  }, {
    "id": 51,
    "unitPrice": 240,
    "quantity": 1,
    "discount": 0,
    "name": "1 Kilo",
    "sku": "64",
    "notes": "Granizado. Cantidad: 4"
  }],
  "payment": [{
    "method": "Efectivo",
    "shipping": 50,
    "discount": 100,
    "voucher": "voucher:PEYA-F53A2ABB-9D5A-4496-97F6-7DED6710BE36",
    "online": false,
    "partial": 250,
    "paymentAmount": 5,
    "subtotal": 245
  }],
  "branchId": "15"
}]
```

**Diccionario del Body**

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `id` | String | Identificador que tiene la orden en el ThirdParty. |
| `state` | String | Estado de la orden. Valor inicial: "PENDING". |
| `preOrder` | Boolean | Será `true` si el local estaba cerrado cuando el cliente hizo el pedido. |
| `registeredDate` | String | Fecha y hora (ISO 8601) en que se realizó la orden en la plataforma del ThirdParty. |
| `deliveryDate` | String | Fecha y hora (ISO 8601) de entrega programada. |
| `pickup` | Boolean | Será `true` si el cliente retira el pedido en el local. |
| `pickupDate` | String \| null | Fecha y hora (ISO 8601) de retiro programado. |
| `notes` | String | Observaciones generales del cliente para toda la orden. |
| `logistics` | Boolean | Será `true` si el local tiene su propio delivery. |
| `user` | Object | Objeto que contiene la información del cliente. |
| `platform` | String | Nombre del thirdParty. |
| `name` | String | Nombre del cliente. |
| `lastName` | String | Apellido del cliente. |
| `id` | Number | Identificador del cliente en el ThirdParty. |
| `email` | String | Email del cliente. |
| `dni` | String | DNI del cliente. |
| `address` | Object | Objeto con la dirección de entrega. |
| `description` | String | Dirección del cliente. |
| `phone` | String | Número de teléfono del cliente. |
| `details` | Array<Object> | Array de objetos, donde cada objeto es un ítem del pedido. |
| `id` | String | Identificador del producto en el ThirdParty. |
| `unitPrice` | Number | Precio unitario del producto. |
| `quantity` | Number | Cantidad solicitada de este producto. |
| `discount` | Number | Descuento monetario aplicado a este producto. |
| `name` | String | Descripción completa del producto. |
| `sku` | String | Código identificador del producto (SKU). |
| `notes` | String | Nota específica para este producto. |
| `promotion` | Boolean | Indica si el detalle es del tipo promoción. Por defecto es `false`. |
| `optionGroups` | Array<Object> | Array que contiene los productos que conforman la promoción. |
| `payment` | Array<Object> | Array que contiene los métodos de pago (usualmente uno solo). |
| `method` | String | Método de pago (ej. "Efectivo"). |
| `shipping` | Number | Costo de envío. |
| `discount` | Number | Descuento total aplicado al pago. |
| `voucher` | String | Código del voucher de descuento, si aplica. |
| `online` | Boolean | Indica si el pago fue online (`true`) o se realiza en la entrega (`false`). |
| `partial` | Number | Monto con el que pagará el cliente (para calcular el vuelto). |
| `paymentAmount` | Number | Vuelto a entregar al cliente. |
| `subtotal` | Number | Precio total de la compra antes de descuentos (Suma de productos + Envío). |
| `branchId` | String | Identificador de la sucursal de destino en el sistema Smartfran. |

**Respuesta Exitosa (200 OK)**

```json
[{
  "id": 62743329,
  "state": "PENDING",
  "branchId": 15
}]
```

**Respuestas de Error**

**401 - No Autorizado (Token inválido, expirado o sin permisos)**

```json
{ "message": "Invalid token." }
{ "message": "The token has not access permission to the route." }
{ "message": "Expired token." }
```

**400 - Bad Request (Orden ya creada)**

```json
{ "error": "order/s 62743329 already exists" }
```

#### **3.3. Cancelar Orden**

Con esta petición puede, si por alguna razón se desea, indicar el rechazo de una orden.

| **Endpoint** | `/orders/cancel` |
| --- | --- |
| **Método** | `POST` |
| **Header** | `Authorization: Bearer ACCESS_TOKEN` |

**Body**

```json
{
  "id": "62743329", //String
  "branchId": "15" //String
}
```

**Respuesta Exitosa (200 OK)**

```json
{
  "id": "62743329",
  "state": "REJECTED"
}
```

**Respuestas de Error**

**401 - No Autorizado (Token inválido, expirado o sin permisos)**

```json
{ "message": "Invalid token." }
{ "message": "The token has not access permission to the route." }
{ "message": "Expired token." }
```

**404 - Not Found (Orden no encontrada)**

```json
{
  "message": "The order 62743330 could not be found.",
  "error": {}
}
```

**400 - Bad Request (Orden ya procesada)**

```json
{
  "error": "La orden 62743330 ya está procesada"
}
```

#### **3.4. Obtener Orden**

Permite consultar una orden mediante el ID con el que fue creada.

| **Endpoint** | `/orders/:id` (Ej: `/orders/62743329`) |
| --- | --- |
| **Método** | `GET` |
| **Header** | `Authorization: Bearer ACCESS_TOKEN` |
| **Body** | No posee. |

**Respuesta Exitosa (200 OK)**

```json
{
  "id": 62743329,
  "state": "REJECTED",
  "preOrder": false,
  "registeredDate": "2019-07-23T16:42:32Z",
  "deliveryDate": "2019-07-23T17:19:32Z",
  "pickup": false,
  "pickupDate": null,
  "notes": "observacion",
  "logistics": false,
  "user": {
    "platform": "thirdParty",
    "name": "Daley",
    "lastName": "Paley",
    "id": 10455712,
    "email": "daley.paley@gmail.com",
    "dni": 37897458
  },
  "address": {
    "description": "Obispo trejo 1420 esquina rew qwre",
    "phone": "4324232"
  },
  "details": [{
    "id": 4568487,
    "unitPrice": 220,
    "quantity": 1,
    "promo": 1,
    "promotion": 1,
    "discount": 0,
    "name": "Crocantino (10 porciones)",
    "sku": "4568487",
    "notes": "nota"
  }],
  "payment": [{
    "method": "Efectivo",
    "shipping": 50,
    "discount": 100,
    "voucher": "voucher:PEYA-F53A2ABB-9D5A-4496-97F6-7DED6710BE36",
    "online": false,
    "partial": 250,
    "paymentAmount": 5,
    "subtotal": 245
  }],
  "timestamp": 1563910952322,
  "branchId": "15"
}
```

**Respuestas de Error**

**401 - No Autorizado (Token inválido, expirado o sin permisos)**

```json
{ "message": "Invalid token." }
{ "message": "The token has not access permission to the route." }
{ "message": "Expired token." }
```

### **4. Callbacks**

Esta sección describe las peticiones que Smartfran realizará hacia la plataforma tercera para notificar cambios de estado y obtener datos de configuración.

#### **4.1 Manejo de Callbacks**

Es necesaria la implementacion de endpoints seguros ara las solicitudes POST que SmartPedidos envía a su plataforma para notificar cambios de estado.

**Autenticación en Callbacks**

Smartfran enviará los callbacks usando el método de autenticación configurado (token o credenciales básicas). Su endpoint debe validar la autenticación antes de procesar la solicitud.

**Manejo de Errores en Callbacks**

*   **Respuesta Exitosa**: Devuelva HTTP 200 OK para confirmar recepción. Smartfran no reintenta si recibe 200.
*   **Errores Temporales**: Si su sistema está temporalmente indisponible, devuelva 5xx. Smartfran puede reintentar (depende de la configuración).
*   **Errores Permanentes**: Para errores de validación (4xx), Smartfran no reintenta. Registre el error para depuración.
*   **Timeouts**: Las solicitudes tienen un timeout de 30 segundos. Procese rápidamente para evitar fallos.

#### **4.2 Validación**

Existen dos formas de validar los datos contra una plataforma nueva:

**1. Autenticación por Token**

Se registra un token en el Backoffice y se envía en el cuerpo (body) de la petición junto con los datos.

**Estructura y Llamada**

```javascript
// El body de la petición incluye el Token y el IdPedido
const body = {
  Token: "el_token_registrado",
  IdPedido: 1245
};

// Las cabeceras deben especificar el Content-Type
const headers = {
  'Content-Type': 'application/json'
};

// Llamada de ejemplo con Axios
axios.post(url, body, { headers: headers });
```

**Ejemplo Concreto**

```javascript
axios.post("https://.../test/CancelarPedido", 
  { 
    Token: "Test1112", 
    IdPedido: 1245 
  }, 
  { 
    headers: { 'Content-Type': 'application/json' } 
  }
);
```

**2. Autenticación por AuthData (Autenticación Básica)**

Se utilizan credenciales de usuario y contraseña que se envían directamente en la configuración de la petición.

**Estructura y Llamada**

```javascript
// Credenciales de autenticación
const auth = { 
  username: "nombre_de_usuario", 
  password: "la_contraseña"
};

// El body de la petición solo contiene los datos del pedido
const body = { 
  IdPedido: 1245 
};
              
// Llamada de ejemplo con Axios, pasando el objeto 'auth'
axios.post(url, body, { auth: auth });
```

**Ejemplo Concreto**

```javascript
axios.post("https://.../test/CancelarPedido",
  { 
    IdPedido: 1245 
  }, 
  { 
    auth: { 
      username: "SmartfranPlatform",
      password: "testpass"
    }
  }
);
```

#### **4.3 Notificación de Estados**

Para cada cambio de estado de un pedido, el Concentrador ejecutará una petición `POST` a un endpoint específico en la `urlBase` de la plataforma tercera. La autenticación puede ser por cualquiera de los dos métodos definidos anteriormente (Token o AuthData).

| Estado | Endpoint | Body Esperado | Disparador (Trigger) |
| --- | --- | --- | --- |
| **1. Recibido** | `urlBase + 'RecibirPedido'` | <pre><code class="language-javascript">{ "IdPedido": 123 }</code></pre> | Se ejecuta cuando la orden llega al POS. |
| **2. Visto** | `urlBase + 'VistarPedido'` | <pre><code class="language-javascript">{ "IdPedido": 123 }</code></pre> | Se ejecuta cuando el operador selecciona la orden en el POS. |
| **3. Confirmado** | `urlBase + 'ConfirmarPedido'` | <pre><code class="language-javascript">{ "IdPedido": 123, "Demora": 15 }</code></pre> | Se ejecuta cuando el operador confirma la orden en el POS. El campo `Demora` representa el ID del tiempo de entrega seleccionado. |
| **4. Cancelado** | `urlBase + 'CancelarPedido'` | <pre><code class="language-javascript">{ "IdPedido": 123, "IdMotivo": 1 }</code></pre> | Se ejecuta cuando el operador cancela la orden en el POS. |
| **5. Enviado** | `urlBase + 'EnviarPedido'` | <pre><code class="language-javascript">{ "IdPedido": 123 }</code></pre> | Se ejecuta cuando el operador despacha la orden desde el POS. |
| **6. Entregado** | `urlBase + 'EntregarPedido'` | <pre><code class="language-javascript">{ "IdPedido": 123 }</code></pre> | Se ejecuta cuando el operador marca la orden como entregada en el POS. |

---

## **5. Solución de Problemas**

Esta sección proporciona guías para resolver problemas comunes durante la integración con la API de Smartfran, incluyendo manejo de errores y callbacks.

### **Errores Comunes en la API**

| Código de Error | Descripción | Solución |
| --- | --- | --- |
| 401 - Unauthorized | Token inválido, expirado o sin permisos. | Verifique que el token sea válido y no haya expirado (24 horas). Asegúrese de que el thirdPartyId y thirdPartySecret sean correctos. |
| 400 - Bad Request (Orden ya creada) | La orden con el ID especificado ya existe. | Use un ID único para cada orden. Verifique el campo `id` en el body. |
| 404 - Not Found | Orden no encontrada para cancelar o consultar. | Confirme que el ID de la orden sea correcto y que la orden haya sido creada previamente. |
| 400 - Bad Request (Orden ya procesada) | Intento de cancelar una orden que ya está en proceso o completada. | Las órdenes solo pueden cancelarse si están en estado PENDING. Verifique el estado antes de intentar cancelar. |
| 400 - Insuficient parameters | Faltan parámetros requeridos en la solicitud. | Asegúrese de incluir todos los campos obligatorios: thirdPartyId/thirdPartySecret para login, id/branchId para cancelar. |
| 200 - Platform not active | La plataforma está desactivada en el backoffice. | Contacte al administrador de Smartfran para activar la plataforma. |
