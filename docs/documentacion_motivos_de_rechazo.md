---
layout: default
title: "Guía de Implementación: API de Motivos de Rechazo para Cloud"
subtitle: "Especificación del Endpoint y Lógica de Consumo"
---

Este documento describe cómo utilizar el nuevo endpoint de la API para obtener una lista de los motivos de rechazo de pedidos y tiempos de entrega.

## **1. Especificación del Endpoint**

Para obtener la lista completa de motivos de rechazo y tiempos de entrega, se debe realizar una petición al siguiente endpoint:

| **Método** | `GET` |
| --- | --- |
| **Ruta** | `/api/branches/rejected-messages` |
| **Autenticación** | Requerida. Se debe incluir un Bearer Token (JWT) válido en la cabecera `/api/branches/login` . |

---

## **2. Formato de la Respuesta (200 OK)**

La respuesta exitosa devolverá un objeto JSON con tres propiedades principales: `rejectedMessages`, `deliveryTimes` y `branchTimeout`.

```json
{
  "rejectedMessages": [
    {
      "id": 1,
      "platformId": 7,
      "name": "Sin Producto/Variedad",
      "descriptionES": "Sin Producto/Variedad",
      "descriptionPT": "Sin Producto/Variedad",
      "forRestaurant": false,
      "forLogistics": true,
      "forPickup": true
    }
    // ... más motivos
  ],
  "deliveryTimes": [
    {
      "id": 1,
      "description": "10-15 min",
      "maxMinutes": 15,
      "minMinutes": 10,
      "name": "10-15",
      "order": 1,
      "platformId": 7
    }
    // ... más tiempos de entrega
  ],
  "branchTimeout": 20
}
```

### **2.1. Detalle del Objeto `rejectedMessages`**

Cada objeto dentro del array `rejectedMessages` representa un motivo de rechazo y tiene la siguiente estructura:

| **Campo** | **Tipo** | **Descripción** |
| --- | --- | --- |
| `id` | Number | Identificador numérico del motivo de rechazo. |
| `platformId` | Number | ID de la plataforma a la que pertenece el motivo (Rappi, PedidosYa, etc.). |
| `name` | String | Nombre corto del motivo de rechazo. |
| `descriptionES` | String | Descripción en español. |
| `descriptionPT` | String | Descripción auxiliar (No utilizada). |
| `forPickup` | Boolean | `true` si el motivo es aplicable a órdenes de **TakeAway (retiro en local)**. |
| `forLogistics` | Boolean | `true` si el motivo es aplicable a órdenes con **Delivery de la Plataforma**. |
| `forRestaurant` | Boolean | `true` si el motivo es aplicable a órdenes con **Delivery Propio del restaurante**. |

### **2.2. Detalle del Objeto `deliveryTimes`**

Contiene la lista de los posibles tiempos de entrega a configurar. Actualmente no se utiliza de manera directa (posible deprecacion).

---

## **3. Guía de Implementación para el Consumidor (Cloud)**

### **3.1. Sincronización de Datos**

Se recomienda que el POS llame a este endpoint `GET /api/branches/rejected-messages` para sincronizar y actualizar su lista local de motivos. La respuesta del endpoint debe **reemplazar completamente** la lista existente en el POS para asegurar consistencia.

### **3.2. Lógica de Filtrado en la Interfaz de Usuario**

Para mostrar al operador solo los motivos de rechazo relevantes, el POS debe implementar la siguiente lógica de filtrado basada en el tipo de orden que se está gestionando:

*   **Para órdenes de TakeAway (Retiro en Local):**
    *   Mostrar únicamente los motivos donde `forPickup` sea `true`.
*   **Para órdenes con Delivery de la Plataforma (ej. repartidor de Rappi):**
    *   Mostrar únicamente los motivos donde `forLogistics` sea `true`.
*   **Para órdenes con Delivery Propio (repartidor del local):**
    *   Mostrar únicamente los motivos donde `forRestaurant` sea `true`.

### **3.3. Lógica de Envío de Rechazo**

**No hay cambios en este flujo, se sigue utilizando el `id` como hasta ahora.**

---

## **4. Respuestas de Error**

| **Código** | **Descripción** |
| --- | --- |
| `401 Unauthorized` | Se devuelve si el token JWT no se proporciona, es inválido o ha expirado. |
| `400 Bad Request` | Se devuelve si ocurre un error en el servidor al procesar la solicitud. |
