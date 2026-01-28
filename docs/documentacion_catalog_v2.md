---
layout: default
title: "Integración de Catálogo: SmartCloud ↔ SmartPedidos"
subtitle: "Arquitectura de integración de datos para la sincronización de catálogos"
---

## **1. Objetivo y Contexto**

El objetivo es automatizar la distribución de cambios de precios, productos, promociones/combos e imagenes. Se busca unificar todo de modo que cuando un administrador en SmartCloud presione **"Publicar Lista de Precios"**, el sistema automáticamente compile toda la información necesaria y la envíe a SmartPedidos.

### **1.1. Requerimientos para SmartCloud**

Para lograr este payload unificado, SmartCloud debe contar con las siguientes lógicas:

*   **Tipificación de Listas de Precio:** Capacidad de asignar **Canal** (Ej: SmartPedidos) y **SubCanal** (Ej: PedidosYa) en la configuración de la lista.
*   **Imágenes por Plataforma:** Selector de **Plataforma** en el módulo Multimedia para filtrar la imagen correcta (Ej: Tag "PedidosYa" con fondo blanco 4:3).

### **1.2. Flujo de Proceso de Publicación**

El proceso es un modelo **PUSH** disparado por evento:

1.  **Disparador:** Usuario hace clic en "Publicar" sobre una Lista de Precios.
2.  **Resolución de Contexto:** SmartCloud identifica la Plataforma (Ej: PedidosYa) y los **BranchIds** afectados.
3.  **Compilación de Datos:**
    *   **Catalog:** Obtiene items de la lista de precios (GetPrices) y resuelve sus **imageUrl** según el tag de plataforma.
    *   **Relations:** Obtiene los combos/promos válidos.
4.  **Envío:** SmartCloud realiza un **POST** al webhook de SmartPedidos con el JSON unificado.

### **1.3. Especificación del JSON Unificado (Payload)**

Este JSON es el **entregable final**.

**Estructura:** Se da un contexto de envio (context) y se separan los items unitarios (catalog) de las estructuras compuestas/promocionales (relations).

#### **Ejemplo de Estructura JSON**

```json
{
    "traceKey": "guid-evento-12345",
    "publishDate": "2025-11-30T10:00:00Z",
    "context": {
        "priceListId": 2,
        "platformId": 1,
        "platformName": "PedidosYa",
        "targetBranchIds": [
            5542,
            3365,
            2835
        ]
    },
    "catalog": [
    {
      "type": "item",
      "data": {
        "id": 1181,
        "priceListId": 912,
        "item": {
          "id": 5,
          "name": "Cerveza Corona",
          "description": "Cerveza Corona",
          "forSale": true,
          "unitForPackage": 24,
          "unitaryItem": null,
          "unitaryItemQuantity": 0,
          "rawMaterialQuantity": 0.0,
          "bussinessUnit": 0.0,
          "fractional": false,
          "generic": null,
          "composition": null,
          "group": {
            "id": 4,
            "name": "Bebidas Con Alcohol",
            "description": "Cervezas - Fernet - Vino",
            "subCategoryId": 3,
            "subCategoryName": "Bebidas",
            "categoryId": 1,
            "categoryName": "Productos de Venta",
            "isKds": false,
            "financialModify": 0
          },
          "skus": [
            {
              "id": 31,
              "code": "5",
              "skuType": {
                "id": 6,
                "name": "SKU_SmartPedidos",
                "description": "SKU SmartPedidos"
              },
              "itemId": 5
            }
          ],
          "defaultImage": null
        },
        "itemId": 5,
        "publishedPrice": 250.0,
        "newPrice": 250.0,
        "internalTax": 0.0,
        "ivaCloudId": "3ebc1610-78b4-4a99-a151-d37d625b538d",
        "enabled": true,
        "isKds": false,
        "financialModify": 0,
        "imageUrl": "https://smartpedidos-catalogimages-weiss.s3.us-east-2.amazonaws.com/CORONA_ok.jpg"
      }
    },
    {
      "type": "item",
      "data": {
        "id": 1180,
        "priceListId": 912,
        "item": {
          "id": 112,
          "name": "Corona S/Alcohol",
          "description": "CORONA S/ALC.",
          "forSale": true,
          "unitForPackage": 24,
          "unitaryItem": null,
          "unitaryItemQuantity": 0,
          "rawMaterialQuantity": 0.0,
          "bussinessUnit": 0.0,
          "fractional": false,
          "generic": null,
          "composition": {
            "id": 179,
            "items": [],
            "generic": []
          },
          "group": {
            "id": 4,
            "name": "Bebidas Con Alcohol",
            "description": "Cervezas - Fernet - Vino",
            "subCategoryId": 3,
            "subCategoryName": "Bebidas",
            "categoryId": 1,
            "categoryName": "Productos de Venta",
            "isKds": false,
            "financialModify": 0
          },
          "skus": [
            {
              "id": 103,
              "code": "112",
              "skuType": {
                "id": 6,
                "name": "SKU_SmartPedidos",
                "description": "SKU SmartPedidos"
              },
              "itemId": 112
            }
          ],
          "defaultImage": null
        },
        "itemId": 112,
        "publishedPrice": 250.0,
        "newPrice": 250.0,
        "internalTax": 0.0,
        "ivaCloudId": "3ebc1610-78b4-4a99-a151-d37d625b538d",
        "enabled": true,
        "isKds": false,
        "financialModify": 0,
        "imageUrl": "https://smartpedidos-catalogimages-weiss.s3.us-east-2.amazonaws.com/CORONA_ok.jpg"
      }
    }
    ],
   "relations": [
    {
      "type": "promo",
      "data": {
        "id": 14,
        "cloudCodeId": "274a9ab1-a1f0-45fd-8edd-851f591725a8",
        "deleted": false,
        "name": "Combo 1 Smack",
        "description": "Combo 1 (burger, fernet y side a elección)",
        "quantity": 0,
        "imageUrl": null,
        "validSinceDate": "2025-12-21T00:00:00+00:00",
        "validToDate": "2025-12-27T00:00:00+00:00",
        "monday": true,
        "tuesday": true,
        "wednesday": true,
        "thursday": true,
        "friday": true,
        "saturday": true,
        "sunday": true,
        "holiday": true,
        "validSinceMinute": 0,
        "validToMinute": 1440,
        "groups": [
          {
            "name": "Hamburguesa",
            "id": 318,
            "amount": 1,
            "details": [
              {
                "id": 1290,
                "articleId": 60
              }
            ],
            "promotionGroupApply": [
              {
                "promotionApplyId": 138,
                "promotionApply": {
                  "include": true,
                  "franchiseId": null,
                  "franchiseeId": null,
                  "cityId": null,
                  "provinceId": null,
                  "countryId": 235,
                  "regionId": null,
                  "priceListId": null
                },
                "promotionValue": 820
              }
            ],
            "type": "FixedPrice",
            "hasAdditionals": false,
            "defaultArticleId": 60,
            "multipleSelection": false
          },
          {
            "name": "Side",
            "id": 319,
            "amount": 1,
            "details": [
              {
                "id": 1291,
                "articleId": 12
              }
            ],
            "promotionGroupApply": [
              {
                "promotionApplyId": 138,
                "promotionApply": {
                  "include": true,
                  "franchiseId": null,
                  "franchiseeId": null,
                  "cityId": null,
                  "provinceId": null,
                  "countryId": 235,
                  "regionId": null,
                  "priceListId": null
                },
                "promotionValue": 0
              }
            ],
            "type": "FixedPrice",
            "hasAdditionals": false,
            "defaultArticleId": 12,
            "multipleSelection": false
          },
          {
            "name": "Bebidas",
            "id": 320,
            "amount": 1,
            "details": [
              {
                "id": 1292,
                "articleId": 133
              },
              {
                "id": 1293,
                "articleId": 135
              },
              {
                "id": 1294,
                "articleId": 411
              },
              {
                "id": 1295,
                "articleId": 134
              },
              {
                "id": 1296,
                "articleId": 4
              },
              {
                "id": 1297,
                "articleId": 129
              },
              {
                "id": 1298,
                "articleId": 19
              },
              {
                "id": 1299,
                "articleId": 413
              },
              {
                "id": 1300,
                "articleId": 419
              },
              {
                "id": 1301,
                "articleId": 414
              },
              {
                "id": 1302,
                "articleId": 415
              },
              {
                "id": 1303,
                "articleId": 416
              },
              {
                "id": 1304,
                "articleId": 417
              },
              {
                "id": 1305,
                "articleId": 418
              },
              {
                "id": 1306,
                "articleId": 421
              }
            ],
            "promotionGroupApply": [
              {
                "promotionApplyId": 138,
                "promotionApply": {
                  "include": true,
                  "franchiseId": null,
                  "franchiseeId": null,
                  "cityId": null,
                  "provinceId": null,
                  "countryId": 235,
                  "regionId": null,
                  "priceListId": null
                },
                "promotionValue": 0
              }
            ],
            "type": "FixedPrice",
            "hasAdditionals": false,
            "defaultArticleId": 0,
            "multipleSelection": false
          }
        ],
        "appliesTo": [
          {
            "include": true,
            "franchiseId": null,
            "franchiseeId": null,
            "cityId": null,
            "provinceId": null,
            "countryId": 235,
            "regionId": null,
            "priceListId": null
          }
        ],
        "franchiseAppliesTo": [],
        "suscriptions": [],
        "createdDate": "2025-12-16T18:11:59.5634569+00:00",
        "activatedDate": "2025-12-20T17:38:41.6831374+00:00",
        "deactivatedDate": null,
        "deactivatedNote": null,
        "mandatoryForAll": true,
        "promotionType": 1,
        "relationCodes": []
      }
    }
    ]
}
```

#### **Diccionario de Datos del Payload**

##### **A. Cabecera (context)**

Define el alcance de la actualización.

*   **platformId:** (Integer) ID interno de mapeo (1=PeYa, 2=Rappi).
*   **targetBranchIds:** (Array) Lista de IDs de sucursales de SmartPedidos donde impactará este catálogo.

##### **B. Productos Unitarios (catalog)**

Array que contiene **únicamente** objetos `type: "item"`.

Representa el inventario de productos.

*   **imageUrl:** Debe venir populado con la URL pública final. SmartCloud **debe haber realizado el filtrado previo** seleccionando la imagen correspondiente a la plataforma del contexto.

##### **C. Relaciones y Promociones (relations)**

Array que contiene objetos `type: "promo"` o estructuras compuestas.

Representa Combos, Descuentos o Menús que referencian a los items del `catalog` pero tienen lógica de agrupación y precios propia.

---

## **2. Requerimientos para SmartCloud**

**Solicitado por: Matias Avila**

### **2.1. Tipificación de Listas de Precios (Canal y SubCanal)**

#### **Situación Actual**

Las listas de precios poseen una clasificación genérica (un solo nivel) y la asignación al POS es restrictiva (relación 1 a 1).

#### **Modificación Solicitada**

**Jerarquía de Canales:** Debe soportar dos niveles de clasificación:

*   **Nivel 1 - Canal :** Define el comportamiento inicial (Minorista, Mayorista, Plataformas).
*   **Nivel 2 - SubCanal - :** Define el destino específico dentro de cada Canal (PedidosYa, Salon, Ambulante…).

**Asignación Múltiple en POS (1:N):** Transformar la configuración del POS para permitir múltiples listas de precios simultáneas.

#### **Justificación**

Esta tipificación habilita la logica de ruteo y formato de Multimedia (Imagenes) entre otros.

#### **Detalle Funcional y de Interfaz (UI/UX)**

La implementación impacta en dos módulos clave del sistema:

**A. Configuración de la Lista de Precios Manager**

En la pestaña "General", en Canal Comercial, se habilita el selector de Sub-Canal con las siguientes opciones definidas:

*   Canal - Minorista :
    *   Sub Canal - Salón
    *   Sub Canal - Ambulante
*   Canal - Plataformas Delivery:
    *   Sub Canal - PedidosYa
    *   Sub Canal - Rappi
    *   Sub Canal - UberEats
*   Canal - Mayorista:
    *   Sub Canal - Gastronómico
    *   Sub Canal - Corner
    *   Sub Canal - Store
    *   Sub Canal - Food Truck

🔗 *[Ver Mockup: SmartCloud - Edición Lista de Precios](https://matiasasmartfram.github.io/spdocumentation/mockup_catalog_listadeprecios_manager.html)*

Nota: El mockup detalla la interacción de los selectores y visualización de los mismos.

**B. Asignacion de la Lista de Precios POS (Editar Punto de Venta)**

En la sección "Datos Pos", se reemplaza el campo único por un Configuración de Listas de Precios.

*   **Selector Jerárquico:** Flujo de selección Canal -> SubCanal -> Lista.
*   **Grilla de Asignaciones:** Tabla que visualiza todas las listas activas en el POS, mostrando claramente el Canal y SubCanal de cada una mediante badges identificatorios.
*   **Regla de Unicidad de SubCanal:** Un POS puede tener múltiples listas asignadas, pero nunca puede repetir la combinación {Canal + SubCanal}.
*   **Mensaje de Error:** "Ya existe una lista asignada para el Canal '{Canal}' y SubCanal '{SubCanal}'. Debe eliminar la existente antes de asignar una nueva."

🔗 *[Ver Mockup: SmartCloud - Edición Lista de Precios POS](https://matiasasmartfram.github.io/spdocumentation/mockup_catalog_listadeprecios_POS.html)*

Nota: El mockup muestra la visualizacion de asignaciones y los mensajes de validación.

### **2.2. Gestión de Multimedia de Imágenes**

#### **Situación Actual**

Existe una relación rígida de 1 a 1 entre Ítem e Imagen. El sistema no valida requisitos (resolución, fondo, formato), lo que provoca que una misma imagen se envíe a todos los canales.

#### **Modificación Solicitada**

*   **Relación 1 a N:** Permitir la carga de múltiples imágenes por ítem, clasificadas mediante un sistema de etiquetas basado en la taxonomía de Canal y SubCanal definida en el punto 2.1.
*   **Validación Mediante Observaciones:** Desplegar alertas visuales con los requisitos técnicos específicos al momento de asignar una imagen a una plataforma.
*   **Resolución Dinámica en Publicación:** Al generar el JSON de la Lista de Precios, el sistema debe inyectar dinámicamente la URL de la imagen que coincida con el canal de destino.

#### **Justificación**

Garantiza que cada canal reciba el activo digital optimizado para su interfaz, evitando el rechazo masivo de catálogos por parte de plataformas externas debido a incumplimiento de estándares gráficos.

#### **Detalle Funcional y de Interfaz**

El módulo de Multimedia se rediseña para gestionar asignaciones múltiples:

**A. Flujo de Carga y Asignación**

*   **Carga:** El usuario sube el archivo o ingresa la URL.
*   **Asignación:** Selecciona el Canal (ej: Plataformas Delivery) y SubCanal (ej: PedidosYa).
*   **Feedback:** Al seleccionar el SubCanal, el sistema muestra un cuadro informativo con las reglas de validación reducidas.
    *Ejemplo Visual: "Requisitos PedidosYa: JPG/PNG, Mín 1440x1080px, Relación 4:3."*
*   **Visualización:** Las imágenes cargadas muestran Chips o etiquetas indicando a qué canales están asignadas.

🔗 *[Ver Mockup: SmartCloud - Edición Lista de Precios Multimedia](https://matiasasmartfram.github.io/spdocumentation/mockup_catalog_item_multimedia.html)*

Nota: El mockup ilustra la asignación de tags y la visualización de requisitos técnicos.

#### **Regla importante: Atributo “imageUrl” en JSON**

El atributo imageUrl del ítem se deberia resolver en el momento de la publicación (Push) o consulta (Pull) basándose en la configuración de la Lista de Precios.

**Resolución:**

1.  El sistema identifica el SubCanal de la Lista de Precios que se está procesando (ej: PedidosYa).
2.  Busca en la colección de imágenes del ítem aquella que tenga el Tag correspondiente a ese SubCanal.
3.  Si encuentra coincidencia, inyecta esa URL específica en el payload.
4.  Si no encuentra coincidencia específica, utiliza la imagen marcada como "Default/Principal".

**Ejemplo de Salida JSON (Payload):**

Para una Lista de Precios configurada como "Plataformas Delivery -> PedidosYa":

```json
{ 
    "id": 112,
    "name": "Corona S/Alcohol",
    "description": "CORONA S/ALC.",
    // Imagen resuelta por SmartCloud filtrando por tag de Sub-Canal "PedidosYa" 
    "imageUrl": "https://cdn.smartcloud.com/images/pedidosya/burger_1440x1080_white_bg.jpg", 
    "composition": {...} 
}
```

Para una Lista de Precios configurada como "Plataformas Delivery -> Rappi":

```json
{ 
    "id": 112,
    "name": "Corona S/Alcohol",
    "description": "CORONA S/ALC.",
    // Imagen resuelta por SmartCloud filtrando por tag de Sub-Canal "Rappi" 
    "imageUrl": "https://cdn.smartcloud.com/images/rappi/burger_4x3_white_bg.rappi.jpg", 
    "composition": {...} 
}
```

**Caso: PedidosYa**

Para el SubCanal "PedidosYa", el sistema debe instruir al usuario vía UI, al seleccionar Plataforma de Pedidos → Pedidos Ya:

*"Requisito Excluyente: Formato Horizontal (4:3), Mínimo 1440x1080px. Formatos JEPG/JPG/PNG."*

### **2.3. Integración de Promociones y Combos**

#### **Situación Actual**

SmartCloud no relacion a los Ítems y a las Promociones en la Lista de Precios.

*   Items: Se obtienen vía GetPrices (Lista de Precios).
*   Promociones: Se obtienen vía GetByPOSCode (Lógica de POS).

Esta separación obliga a realizar múltiples llamadas.

#### **Modificación Solicitada (Backend SmartCloud)**

El servicio de publicación de Lista de Precios debe actuar como un Agregador.

Al publicar una Lista de Precios, debe:

1.  **Identificar Promociones Vinculadas:** Buscar todas las promociones activas que tengan asignada la PriceList que se está publicando.
2.  **Transformar la Estructura:** Convertir el modelo de datos interno de Promociones (basado en GetByPOSCode) al esquema simplificado relations del JSON unificado.
    `{ "type": "promo", "data": {...`
3.  **Inyectar en Payload:** Anexar este array de promociones (objeto relations) al mismo JSON que contiene los ítems individuales.

---

## **3. Especificación Técnica Detallada**

Este JSON es el entregable final que SmartPedidos espera recibir. Contiene la Lista de Precios (Items) + Las Promociones/Combos

### **3.1. Estructura General del JSON**

El payload se divide en tres secciones: Cabecera (context), Productos Unitarios (catalog) y Relaciones (relations).

```json
{
  "traceKey": "guid-evento-12345",
  "publishDate": "2025-11-30T10:00:00Z",
  "context": { (Ver detalle 1.2)
    "priceListId": 2,
    "platformId": 1,
    "platformName": "PedidosYa",
    "targetBranchIds": [ 5542, 3365, 2835 ]
  },
  "catalog": [
    // Array de Items Unitarios (Ver detalle 1.3)
  ],
  "relations": [
    // Array de Promos y Combos (Ver detalle 1.4)
  ]
}
```

### **3.2. Contexto y Ruteo (context)**

Esta sección es para el enrutamiento del mensaje. Define qué plataforma recibe la data y a qué sucursales específicas debe impactar.

#### **Estructura del Objeto JSON**

```json
"context": {
  "priceListId": 2,
  "platformId": 1,            // Mapeado según tabla interna (Ej: 1 = PedidosYa)
  "platformName": "PedidosYa", // String legible
  "targetBranchIds": [ 5129, 5542 ] // IDs de SmartPedidos de los POS afectados
}
```

#### **3.2.1. Mapeo de Plataformas (platformId y platformName)**

El backend de SmartCloud debe implementar un Enum o Diccionario Estático para traducir el SubCanal seleccionado en la UI (Punto 2.1) a los códigos internos requeridos por la API de SmartPedidos.

**Tabla de Referencia (Hardcoded):**

| SubCanal (SmartCloud UI) | platformName (JSON) | platformId (JSON) |
| :--- | :--- | :--- |
| PedidosYa | PedidosYa | 1 |
| Rappi | Rappi | 2 |
| Uber Eats | UberEats | 4 |
| PaD | PaD | 5 |
| Croni | Croni | 6 |
| PediGrido | PediGrido | 7 |
| Mercado Pago | MercadoPago | 8 |
| Glovo | Glovo | 9 |
| Toque | Toque | 11 |
| I+D | I+D | 12 |

Regla de Desarrollo: Si se agrega una nueva integración en el futuro, se debe actualizar esta tabla de mapeo en el backend de SmartCloud antes de habilitar el SubCanal en la UI.

#### **3.2.2. Resolución de targetBranchIds**

Este array debe contener los IDs de autenticación ("User") que identifican a cada sucursal dentro del ecosistema de SmartPedidos.

**Fuente de Datos:**

*   Módulo: Franquicias > Puntos de Venta > Pestaña "Integraciones".
*   Campo Objetivo: El valor del input "User" ubicado dentro del bloque/toggle activado de Smart Pedidos.

**Algoritmo de Resolución (Paso a Paso):**

1.  Disparador: El usuario presiona "Publicar" en la Lista de Precios (ej: ID 2).
2.  Identificación de Alcance (Query 1):
    *   Buscar en la base de datos todos los Puntos de Venta (POS) que tengan asignada la PriceListId = 2 en su configuración de "Datos Pos".
    *   Resultado: Lista de objetos POS.
3.  Extracción de Credenciales (Query 2):
    *   Para cada POS de la lista anterior, consultar su configuración de Integraciones.
    *   Verificar si el toggle "Usar Smart Pedidos" está true.
    *   Si está activo, leer el valor del campo "User" (Ej: 5129).
4.  Inyección:
    *   Agregar el valor 5129 al array targetBranchIds.
    *   Repetir para todos los POS encontrados.

**Ejemplo de Flujo de Datos:**

Escenario: La "Lista Córdoba" se usa en 2 locales.

*   Local A (Centro): En Integraciones > Smart Pedidos > User tiene el valor 5129.
*   Local B (Cerro): En Integraciones > Smart Pedidos > User tiene el valor 5542.

Resultado en JSON:

```json
"targetBranchIds": [ 5129, 5542 ]
```

Si un POS usa la lista de precios pero NO tiene activa la integración con SmartPedidos (o el campo User está vacío), ese POS debe ser omitido del array targetBranchIds para evitar errores de ruteo.

### **3.3. Productos Unitarios (catalog)**

Esta sección contiene el inventario de productos en la Lista de Precios /api/v1/PriceLists/GetPrices/{PriceListId}. A diferencia del endpoint mencionado que retorna un array plano, el JSON de integración requiere una para estandarizar el contenido.

#### **3.3.1. Estructura del Objeto**

Cada elemento del array catalog debe ser un objeto con dos propiedades obligatorias que no existen en el modelo original:

*   type: Identificador de tipo de objeto. Para esta sección, el valor fijo es "item".
*   data: Objeto contenedor donde se vuelcan las propiedades del producto.

Esquema de Transformación:

```json
// Destino (Payload Unificado)
"catalog": [
  //item 1
  {
    "type": "item",       // Nuevo campo discriminador
    "data": { ... }       // Aquí van los datos mapeados de GetPrices del item
  },
  //item 2
  {
    "type": "item",       // Nuevo campo discriminador
    "data": { ... }       // Aquí van los datos mapeados de GetPrices del item
  }
]
```

#### **3.3.2. Construcción del Objeto data**

La todos los campos (name, description, publishedPrice, enabled) se copian directamente del objeto item original. Sin embargo, los siguientes campos requieren una lógica específica:

**A. Nuevo Atributo: imageUrl (Resolución Dinámica)**

El backend debe calcular este campo teniendo en cuenta las modificaiones solicitadas en “Gestion Multimedia de Imagenes (Link):

*   Leer el platformId del contexto (Ej: Id: 1 == "PedidosYa").
*   Buscar en la colección de multimedia del ítem la imagen que tenga el tag coincidente (Ej: Id: 1).
*   Resultado: Inyectar la URL encontrada en data.imageUrl.

**B.Ejemplo de JSON Resultante**

```json
"catalog": [
    {
      "type": "item",
      "data": {
        "id": 1181,
        "priceListId": 912,
        "item": {
          "id": 5,
          "name": "Cerveza Corona",
          "description": "Cerveza Corona",
        //etc... -> Demas campos (Copiados Exactamente de GetPrices)
        // Campos Nuevos
        "imageUrl": "https://cdn.smartcloud.com/images/peya/agua_gas_4x3.jpg", // Resuelto por Tag
      },
     {
      "type": "item",
      "data": {
        "id": 1180,
        "priceListId": 912,
        "item": {
          "id": 112,
          "name": "Corona S/Alcohol",
          "description": "CORONA S/ALC.",
        //etc... -> Demas campos (Copiados Exactamente de GetPrices)    
        // Campos Nuevos
        "imageUrl": "https://cdn.smartcloud.com/images/peya/agua_4x3.jpg", // Resuelto por Tag
      }
    }
]
```

### **3.4. Relaciones y Promociones (relations)**

Esta sección contiene las estructuras compuestas (Combos, Menús). El backend debe construir este array dinámicamente, obteniendo información de endpoints internos para generar el listado final.

#### **3.4.1. Lógica de Agregación y Filtrado (Backend)**

El servicio de publicación debe ejecutar la siguiente secuencia lógica para poblar este array:

*   Obtener Contexto: Leer el priceListId que se está publicando (Ej: 2) desde el contexto del evento.
*   Consultar Universo de Promociones: Ejemplo:/business/promotions/ para obtener el listado .
*   Filtrado : Iterar sobre las promociones y seleccionar únicamente aquellas que tengan asignado el priceListId (Ej: 2) actual en su configuración.
*   Estandarización de Formato:
    *   La data de la promoción seleccionada debe formatearse siguiendo el esquema del endpoint /api/v1/Promotion/GetByPOSCode.
    *   Se deben aplicar las transformaciones de Imagen.

#### **3.4.2. Estructura del Array relations**

El array relations es una colección de objetos. Cada objeto representa UNA promoción individual y debe estar encapsulado con el discriminador de tipo.

Esquema del Array:

```json
"relations": [
  // Promoción 1
  {
    "type": "promo",       // Discriminador fijo
    "data": { ... }        // Estructura idéntica a GetByPOSCode (Transformada)
  },
  // Promoción 2
  {
    "type": "promo",
    "data": { ... }
  },
  // Promoción N...
  {
    "type": "promo",
    "data": { ... }
  }
]
```

#### **3.4.3. Ejemplo de JSON Resultante (Objeto Transformado)**

A continuación se muestra cómo se ve un objeto dentro del array relations una vez procesado. Observar que data mantiene la estructura de GetByPOSCode pero con los valores (precios, imágenes, nombres) ya resueltos para la integración.

```json
"relations": [
    {
      "type": "promo",
      "data": {
        "id": 2,
        "cloudCodeId": "e3387769-02dd-4a81-aa4c-340b6627e674",
        "deleted": false,
        "name": "Combo 1 Peya",
        "description": "Combo 1 Peya",
        "quantity": 1,
        "imageUrl": null,
        "validSinceDate": "2025-09-25T00:00:00+00:00",
        "validToDate": "2027-04-30T00:00:00+00:00",
        "monday": true,
        "tuesday": true,
        "wednesday": true,
        "thursday": true,
        "friday": true,
        "saturday": true,
        "sunday": true,
        "holiday": true,
        "validSinceMinute": 0,
        "validToMinute": 1440,
        "groups": [
            {
                "name": "Hamburgesa",
                "id": 163,
                "amount": 1,
                "details": [
                    {
                        "id": 809,
                        "articleId": 9
                    },
                    {
                        "id": 810,
                        "articleId": 8
                    }
                ],
                "promotionGroupApply": [
                    {
                        "promotionApplyId": 65,
                        "promotionApply": {
                            "include": true,
                            "franchiseId": null,
                            "franchiseeId": null,
                            "cityId": null,
                            "provinceId": null,
                            "countryId": 11,
                            "regionId": null,
                            "priceListId": null
                        },
                        "promotionValue": 16900
                    }
                ],
                "type": "FixedPrice",
                "hasAdditionals": false,
                "defaultArticleId": 0,
                "multipleSelection": false
            },
            {
                "name": "Acompañamiento",
                "id": 164,
                "amount": 1,
                "details": [
                    {
                        "id": 811,
                        "articleId": 12
                    },
                    {
                        "id": 812,
                        "articleId": 29
                    }
                ],
                "promotionGroupApply": [
                    {
                        "promotionApplyId": 65,
                        "promotionApply": {
                            "include": true,
                            "franchiseId": null,
                            "franchiseeId": null,
                            "cityId": null,
                            "provinceId": null,
                            "countryId": 11,
                            "regionId": null,
                            "priceListId": null
                        },
                        "promotionValue": 0
                    }
                ],
                "type": "FixedPrice",
                "hasAdditionals": false,
                "defaultArticleId": 0,
                "multipleSelection": false
            },
            {
                "name": "Bebida",
                "id": 165,
                "amount": 1,
                "details": [
                    {
                        "id": 813,
                        "articleId": 4
                    },
                    {
                        "id": 814,
                        "articleId": 129
                    }
                ],
                "promotionGroupApply": [
                    {
                        "promotionApplyId": 65,
                        "promotionApply": {
                            "include": true,
                            "franchiseId": null,
                            "franchiseeId": null,
                            "cityId": null,
                            "provinceId": null,
                            "countryId": 11,
                            "regionId": null,
                            "priceListId": null
                        },
                        "promotionValue": 0
                    }
                ],
                "type": "FixedPrice",
                "hasAdditionals": false,
                "defaultArticleId": 0,
                "multipleSelection": false
            }
        ],
        "appliesTo": [
            {
                "include": true,
                "franchiseId": null,
                "franchiseeId": null,
                "cityId": null,
                "provinceId": null,
                "countryId": 11,
                "regionId": null,
                "priceListId": null
            }
        ],
        "franchiseAppliesTo": [],
        "suscriptions": [],
        "createdDate": "2025-09-25T17:59:10.1880684+00:00",
        "activatedDate": "2025-12-12T18:40:19.7666985+00:00",
        "deactivatedDate": null,
        "deactivatedNote": null,
        "mandatoryForAll": true,
        "promotionType": 1,
        "relationCodes": []
    },
    {
    "type": "promo",
      "data": {....}
    }
]
```
