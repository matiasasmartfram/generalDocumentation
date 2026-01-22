---
layout: default
title: "Especificación Técnica: Nuevos Atributos para CanjeApp"
subtitle: "Especificación Técnica para BI: Formato de Pedidos con Canje"
---

## **1. Objetivo de este Documento**

Este documento describe e ilustra la adición de nuevos atributos en las colecciones de MongoDB **orders** y **news**. Esta actualización da soporte a la nueva funcionalidad **CanjeApp**, permitiendo identificar y obtener detalles de los pedidos que incluyen un canje de puntos.

---

## **2. Resumen del Cambio**

<div class="note">
  <p><strong>Tipo de cambio:</strong></p>
  <ul>
      <li>Corresponde únicamente a una <strong>adición de nuevos atributos</strong>. No se modificarán ni eliminarán campos existentes.</li>
  </ul>
  <p><strong>Condición de Aplicación:</strong></p>
  <ul>
      <li>Los nuevos atributos aparecerán <strong>solo cuando</strong> el pedido provenga de la plataforma <strong>PediGrido (<code>plataformaId: 7</code>)</strong>.</li>
  </ul>
  <p><strong>Colecciones Afectadas:</strong></p>
  <ul>
      <li><code>orders</code> (Datos de entrada)</li>
      <li><code>news</code> (Datos de salida transformados)</li>
  </ul>
  <p>Para el resto de los pedidos (de otras plataformas Ej. PedidosYa, Rappi. etc.), la estructura de los documentos permanecerá sin cambios.</p>
</div>

---

## **3. Detalle de Nuevos Atributos**

A continuación se detallan los nuevos campos. Los JSON mostrados son ejemplos simplificados; todos los demás atributos existentes en las órdenes seguirán presentes.

### **3.1. Colección `orders` (Datos de Entrada)**

Estos atributos se agregarán al documento original del pedido cuando se reciba un canje desde PediGrido.

#### **Nuevos Atributos:**

*   `order.user.tipoIdentificacion` (String)
*   `order.user.numeroTarjetaLoyalty` (String)
*   `order.user.contieneCanje` (Boolean)
*   `order.user.puntosCanjeados` (Int)
*   `order.details[].canje` (Int - Valor 1 o 0)

#### **Ejemplo de Estructura `order` con Canje:**

```json
{
  "order": {
    "user": {
      "dni": "39690194",
      "tipoIdentificacion": "DNI",            // NUEVO ATRIBUTO
      "numeroTarjetaLoyalty": "1234567890",    // NUEVO ATRIBUTO
      "contieneCanje": true,                    // NUEVO ATRIBUTO
      "puntosCanjeados": 3000                   // NUEVO ATRIBUTO
    },
    "details": [
      {
        "name": "50% Descuento en Kilo por 3000 Puntos",
        "sku": "1641",
        "canje": 1,                             // NUEVO ATRIBUTO
        "promotion": false,
        "optionGroups": [
          { 
            "name": "Kilo de Helado", 
            "sku": "64",
            "notes": "Chocolate, Vainilla" 
          }
        ]
      }
    ]
    // ... resto de los atributos de la orden
  }
}
```

---

### **3.2. Colección `news` (Salida Transformada)**

Estos atributos se agregarán al documento transformado que se genera para el consumo final.

#### **Nuevos Atributos:**

*   `order.customer.tipoIdentificacion` (String)
*   `order.customer.numeroTarjetaLoyalty` (String)
*   `order.customer.contieneCanje` (Boolean)
*   `order.customer.puntosCanjeados` (Int)
*   `order.details[].canje` (Int - Valor 1 o 0)

#### **Ejemplo de Estructura `news` con Canje:**

```json
{
  "order": {
    "customer": {
      "dni": "39690194",
      "tipoIdentificacion": "DNI",              // NUEVO ATRIBUTO
      "numeroTarjetaLoyalty": "1234567890",      // NUEVO ATRIBUTO
      "contieneCanje": true,                      // NUEVO ATRIBUTO
      "puntosCanjeados": 3000                     // NUEVO ATRIBUTO
    },
    "details": [
      {
        "description": "50% Descuento en Kilo por 3000 Puntos",
        "sku": "1641",
        "price": 5000,
        "promo": 2,
        "groupId": 1,
        "canje": 1                                // NUEVO ATRIBUTO
      },
      {
        "description": "Kilo de Helado",
        "sku": "64",
        "price": 0,
        "optionalText": "Chocolate, Vainilla",
        "promo": 1,
        "groupId": 1,
        "canje": 1                                // NUEVO ATRIBUTO
      }
    ]
    // ... resto de los atributos de la orden
  }
}
```

---

Documento generado por el equipo de SmartPedidos | Fecha de Actualización: 11 de agosto de 2025
