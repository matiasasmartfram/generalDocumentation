---
layout: default
title: "Lógica de Cash on Delivery (COD) en PedidosYa"
subtitle: "Basada estrictamente en el código de api/src/platforms/interfaces/pedidosYa.js"
---

Esta documentación explica cómo funciona la lógica para aplicar Cash on Delivery (pago contra entrega) en los pedidos de PedidosYa, basada estrictamente en el código de `api/src/platforms/interfaces/pedidosYa.js`.

## **¿Qué es Cash Collection?**

Es un modelo donde el cliente paga en efectivo al repartidor de PedidosYa, pero la plataforma se encarga de recoger y conciliar el dinero. La sucursal no maneja el efectivo directamente, simplificando su contabilidad.

## **Lógica Principal: Clasificación del Tipo de Pago**

Esta parte decide si un pago es en efectivo o con tarjeta/crédito.

1.  **Limpieza de datos**: Se toman el tipo de pago (ej. "Cash" o "Efectivo") y el estado (ej. "paid"), y se ponen en minúsculas sin espacios para evitar errores.
2.  **Valor de pago al restaurante**: Se mira el campo `payRestaurant` en los precios. Si no existe, se usa "1". Indica si la sucursal paga por el servicio de entrega.
3.  **¿Es pago en efectivo?**
    *   Si el tipo incluye "cash" o "efectivo" Y `payRestaurant` no es "0" (la sucursal paga algo):
        *   Se marca como efectivo.
        *   Se calcula el total restando descuentos.
        *   Si ya está pagado: no hay vuelto que dar, y el pago cubre el total.
        *   Si no está pagado: el vuelto es el total (debe pagar), y no ha pagado nada aún.
    *   Si no cumple: se marca como crédito (tarjeta).
4.  **¿Es un pago online?**
    *   Un pago es online si ya está pagado, o si no es efectivo (no cumple las condiciones anteriores).
5.  **Si es online, se fuerza a crédito**: Aunque fuera efectivo, si es online, se cambia a crédito para manejarlo como digital.

## **Determinación de Entrega Propia (ownDelivery)**

Decide si el pedido usa entrega de la sucursal o de PedidosYa.

*   Es entrega propia si hay información de entrega Y no hay hora de recogida por repartidor de PedidosYa.
*   Pero si no hay recogida en tienda, NO es entrega propia.
*   Resultado: solo es propia si existe entrega, no hay riderPickupTime, Y existe pickup.

## **Ajuste Final en las Notas del Pedido**

Después de clasificar, se hace un último cambio:

*   Si las notas incluyen "Efectivo" Y la sucursal no paga nada (`payRestaurant` es "0"):
    *   Si NO es entrega propia: cambia "Efectivo" por **"Cash Collection"** en las notas.
    *   Si SÍ es entrega propia: deja las notas como están, pero marca el pago como offline y efectivo.

## **Ejemplos Corregidos**

| Escenario | ownDelivery | payRestaurant | Notas Originales | Resultado |
| :--- | :--- | :--- | :--- | :--- |
| PedidosYa + Cash Collection | false | "0" | "Medio de pago: Efectivo..." | Cambia a "Medio de pago: Cash Collection..." |
| Entrega Propia + Cobro normal | true | "1500" | "Medio de pago: Efectivo..." | Sin cambios |
| Entrega Propia + Intento COD | true | "0" | "Medio de pago: Efectivo..." | Sin cambios en notas; pago offline y efectivo |
| PedidosYa + No efectivo | false | "0" | Sin "Efectivo" | Sin cambios |

Esta lógica asegura que **Cash Collection** se active solo en pedidos de PedidosYa donde la sucursal no paga por entrega, evitando confusiones en entregas propias.
