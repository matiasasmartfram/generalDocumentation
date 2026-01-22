---
layout: default
title: "Documentación: Actualización de la Colección 'news'"
subtitle: "Adición del Objeto 'tenant'"
---

## **1. Adición del Objeto 'tenant'**

A partir de ahora, la colección **`news`** incluirá un nuevo objeto llamado **`tenant`** dentro de la estructura del objeto **`order`**. Este cambio se aplicará a todos los documentos de la colección.

Este nuevo objeto contendrá información sobre el tenant. La estructura del objeto **`tenant`** es la siguiente:

```json
"tenant": {
    "name": "grido",
    "orgId": "org_g4qPlLbxcJZx5e7U",
    "tenantId": "d3186bc6d7b2"
}
```

<div class="note">
    <h3>&#9432; Nota Importante</h3>
    <p>Los valores de los atributos <strong><code>name</code></strong>, <strong><code>orgId</code></strong> y <strong><code>tenantId</code></strong> cambiarán dinámicamente en cada documento según el tenant al que corresponda la <strong><code>news</code></strong>.</p>
</div>

## **2. Ejemplo de Documento 'news' Actualizado**

A continuación, un ejemplo de un documento de la colección **`news`** para ilustrar la ubicación y el formato del nuevo objeto **`tenant`**.

```json
{
    "_id": {
        "$oid": "68c010e9810d2f81f1bed140"
    },
    "order": {
        "id": 4100552,
        ...
        "pickupOnShop": true,
        "pickupDateOnShop": "2019-09-23T16:40:32Z",
        "preOrder": false,
        "observations": "observacion",
        "ownDelivery": false,
        // Nuevo objeto agregado
        "tenant": {
            "name": "grido",
            "orgId": "org_g4qPlLbxcJZx5e7U",
            "tenantId": "d3186bc6d7b2"
        },
        ...
    },
    ...
}
```

<div style="text-align: center; margin-top: 3em; padding-top: 1em; border-top: 1px solid #eee; font-family: 'Source Code Pro', monospace; font-weight: 600; color: #7f8c8d;">
    { smart Pedidos }
</div>
