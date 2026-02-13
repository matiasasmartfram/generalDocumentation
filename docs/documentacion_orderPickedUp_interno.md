---
layout: default
title: "Análisis de Variable: orderPickedUp"
subtitle: "Función, ubicación y comportamiento de la variable"
---

## **1. Propósito de la Variable**

La variable **`orderPickedUp`** ha sido diseñada para evitar que una sucursal pueda rechazar o cancelar una orden que ya ha sido retirada físicamente por un repartidor externo.

Esto previene conflictos donde el repartidor ya está en la tienda pero el sistema del punto de venta (POS) intenta anular el pedido, lo cual generaría una inconsistencia entre lo que sucede en la calle y lo que figura en los sistemas de la plataforma.

## **2. Ubicación en la Base de Datos**

La variable se encuentra en la colección **`news`** de MongoDB. Se ubica directamente en la **raíz del documento**, al mismo nivel que campos como `order`, `branchId` o `traces`.
El campo existe para todas las órdenes en la colección `news`, sin importar la plataforma.

### **Ejemplo de Estructura JSON**

A continuación, se muestra el formato de una "novedad" (news). Observa que **`orderPickedUp`** se encuentra resaltado en la estructura principal:

```json
{
  "_id": { "$oid": "698cec3ec3f1d235da83c572" },
  "order": {
    "id": "1770843198714",
    "platformId": 1,
    "statusId": 5,
    "totalAmount": 4500
    // ... otros campos de la orden
  },
  "branchId": 50,
  "extraData": {
    "platform": "PedidosYa",
    "country": "Uruguay"
  },
  
  "orderPickedUp": false,  // <--- UBICACIÓN: Raíz del documento news
  
  "traces": [
    { "entity": "PLATFORM", "update": { "typeId": 1 } },
    { "entity": "BRANCH", "update": { "typeId": 10 } }
  ],
  "typeId": 14,
  "updatedAt": { "$date": "2026-02-12T19:45:26.172Z" }
}
```

## **3. Funcionamiento en PedidosYa**

Actualmente, **PedidosYa** es la plataforma que gatilla la actualización de este campo:

1.  **Detección**: Cuando PedidosYa envía una notificación de actualización (`updateOrder`) con el estado `"order_picked_up"`.
2.  **Actualización**: El controlador `peya.js` identifica este estado y ejecuta un `findOneAndUpdate` poniendo `orderPickedUp: true`.
3.  **Validación de Bloqueo**:
    *   Cuando la sucursal intenta enviar un **Rechazo** (`BranchRejectStrategy`), el sistema consulta esta variable.
    *   Si `orderPickedUp === true`, el sistema registra un log de advertencia y **aborta el rechazo**, evitando que se envíe la cancelación a la plataforma.

## **4. Plataforma y Escalabilidad**

Aunque el flujo de actualización automática (vía Webhook) está activo inicialmente para **PedidosYa**, la infraestructura es genérica:

*   **Modelo**: El campo existe para todas las órdenes en la colección `news`, sin importar la plataforma.
*   **Protección Activa**: La lógica que impide el rechazo (`BranchRejectStrategy`) ya está configurada para chequear este campo en cualquier orden.
*   **Futuras Integraciones**: Si se requiere el mismo comportamiento para **UberEats** o **Rappi**, solo se debe añadir la lógica de "escucha" en sus respectivos controladores para que actualicen el campo cuando reciban el evento de retiro (Picked Up).

---

*Documentación generada para el equipo de desarrollo de SmartFran.*
