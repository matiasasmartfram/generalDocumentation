---
layout: default
title: "Análisis de Logs de Apertura/Cierre - PediGrido"
subtitle: "Lógica de funcionamiento de los registros en openclosedlogs"
---

Este documento detalla la lógica de funcionamiento de los registros en la colección **`openclosedlogs`** para la plataforma **PediGrido**, diferenciando entre los procesos de tipo `Special` y `CRON`.

---

## **1. El Proceso "Special" (Detección por latido / Heartbeat)**

El tipo `Special` se encarga de monitorear la conexión en tiempo real de las tiendas basándose en el campo **`lastGetNews`**. Es un mecanismo reactivo a la actividad del agente en la sucursal.

### **Lógica de Ejecución**

*   **Frecuencia:** Se ejecuta cada **1 minuto exacto**.
*   **Momento:** En el segundo `00` de cada minuto (sincronizado con el reloj del servidor).
*   **Código de referencia (`api/src/controllers/branch.js`):**

```javascript
// Definición del Cron (Línea ~964)
const schedulePediGridoOffline = '*/1 * * * *';
cron.schedule(schedulePediGridoOffline, async () => await checkOfflinePOSpedigrido());

// Función de validación (Línea ~731)
async function checkOfflinePOSpedigrido() {
  const dateNow = new Date();
  // El umbral es "hace 1 minuto"
  const date3minutes = new Date(dateNow.getTime() - (1 * 60 * 1000)); 

  // CASO CIERRE: Si lastGetNews es más viejo que 1 minuto
  let branchesToClose = await model.find({
    'platforms.platform': pedigridoId,
    lastGetNews: { $lt: date3minutes }, 
    alreadyClosedPediGrido: { $in: [null, false] }
  });

  // CASO APERTURA: Si el reporte volvió y es más reciente que el umbral
  let branchesToOpen = await model.find({
    'platforms.platform': pedigridoId,
    lastGetNews: { $gt: date3minutes },
    alreadyClosedPediGrido: { $in: [null, true] }
  });
  
  // ... lógica de envío a API y creación de log ...
}
```

### **Ejemplo Práctico de Tiempos**

Si el **`lastGetNews`** de una tienda se registra a las **10:07:09 AM**:

1.  **Ejecución 10:08:00:**
    *   Umbral calculado: `10:07:00`.
    *   Como `10:07:09` es **mayor** que `10:07:00`, el sistema detecta la tienda como **ONLINE**.
    *   Si estaba cerrada, genera un log `Special` de tipo **"Open"**.
2.  **Ejecución 10:09:00 (Si no hubo más reportes):**
    *   Umbral calculado: `10:08:00`.
    *   Como `10:07:09` es **menor** que `10:08:00`, el sistema detecta la tienda como **OFFLINE**.
    *   Genera un log `Special` de tipo **"Closed"**.

---

## **2. El Proceso "CRON" (Sincronización por Horario)**

El tipo `CRON` no depende de la conexión de la tienda, sino del calendario de horarios de apertura configurados en el sistema de Grido.

### **Lógica de Ejecución**

*   **Frecuencia:** Se programa dinámicamente según los horarios de apertura (`apertura`) de cada sucursal.
*   **Margen de Cortesía:** El sistema añade **2 minutos** al horario de apertura para asegurar que el sistema local haya tenido tiempo de arrancar.
*   **Código de referencia (`api/src/controllers/branch.js`):**

```javascript
// Programación dinámica (Línea ~167)
let newMinuto = minuto + 2; // Añade 2 minutos al horario de la API
let newHora = hora;

// Se crea una expresión cron para el día y hora específica
const cronExp = `${m} ${h} * * ${dia}`;
jobsCron[jobKey] = cron.schedule(cronExp, async () => {
    await openClosedBranchesGrido(country);
});
```

### **Ejemplo Práctico**

*   Si una tienda tiene configurada su apertura a las **09:30:00**.
*   El sistema generará un proceso `CRON` para las **09:32:00**.
*   En ese momento, se forzará la apertura en la plataforma y se grabará el log con `type: "CRON"`.

---

## **3. Resumen de Diferencias**

| Característica | Proceso `Special` | Proceso `CRON` |
| :--- | :--- | :--- |
| **Disparador** | Latido de conexión (`lastGetNews`) | Horario de reloj (Calendario Grido) |
| **Frecuencia** | Cada 1 minuto (Loop constante) | Una vez por cada evento de horario |
| **Objetivo** | Detectar caídas de internet o cierres forzados | Asegurar apertura/cierre según contrato |
| **Identificador en Log** | `type: "Special"` | `type: "CRON"` |
| **Umbral de tiempo** | 1 minuto de inactividad | Hora exacta + 2 minutos |
