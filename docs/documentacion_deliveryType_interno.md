---
layout: default
title: "Análisis Técnico: Tipo de Delivery"
subtitle: "Reglas para Pedigrido y PedidosYa"
---

Este documento detalla las reglas de negocio aplicadas para resolver la modalidad de entrega en las integraciones de **Pedigrido** (Interfaz ThirdParty) y **PedidosYa**, basándose en el mapeo de datos de entrada hacia el sistema interno.

## 1. Interfaz Pedigrido / ThirdParty
**Ubicación:** `api/src/platforms/interfaces/thirdParty.js` -> `orderMapper()`

La resolución en esta interfaz se basa en flags booleanos directos. El sistema aplica una jerarquía donde el retiro tiene prioridad sobre la logística de envío.

### 🛠️ Variables de Entrada
| Campo Original (`data.order`) | Atributo Interno | Definición |
| :--- | :--- | :--- |
| `ORDER.pickup` | `NEWS.pickupOnShop` | Indica si el cliente retira por el local. |
| `ORDER.logistics` | `NEWS.ownDelivery` | Indica si el local se encarga del envío (Delivery Propio). |

### 📊 Matriz de Resolución
| Combinación | `ORDER.pickup` | `ORDER.logistics` | `NEWS.pickupOnShop` | `NEWS.ownDelivery` | Resolución / Escenario |
| :---: | :---: | :---: | :---: | :---: | :--- |
| **1** | `true` | *(Cualquiera)* | ✅ **true** | ❌ **false** | **Retira Cliente (Take Away):** Retira el Cliente en el Local. |
| **2** | `false` | `true` | ❌ **false** | ✅ **true** | **Delivery Propio:** Logística a cargo del local. |
| **3** | `false` | `false` | ❌ **false** | ❌ **false** | **Delivery a Cargo de la Plataforma:** Logística a cargo de la Plataforma de Pedidos. |

> [!NOTE]
> **Regla de Prioridad:** En la interfaz ThirdParty, si `NEWS.pickupOnShop` es `true`, el sistema ignora el valor de `NEWS.ownDelivery` y marca la orden como **Retira Cliente (Take Away):**.

---

## 2. Interfaz PedidosYa
**Ubicación:** `api/src/platforms/interfaces/pedidosYa.js` -> `orderMapper()`

Para PedidosYa, el factor determinante no es un flag de logística, sino la **asignación de un repartidor** (Rider) por parte de la plataforma.

### 🛠️ Variables de Entrada
| Campo Original (`data.order`) | Atributo Interno | Definición |
| :--- | :--- | :--- |
| `ORDER.pickup` | `NEWS.pickupOnShop` | Objeto presente solo en órdenes de Retiro. |
| `ORDER.delivery.riderPickupTime` | `NEWS.ownDelivery` | **Clave:** Si es **vacío (null)**, el envío es propio. |

### 📊 Matriz de Resolución (Validada por Pruebas)
| Combinación | `ORDER.pickup` (Objeto) | `ORDER.delivery.riderPickupTime` | `NEWS.pickupOnShop` | `NEWS.ownDelivery` | Resolución / Escenario  |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **1** | **Existe** | **≠ null** | ✅ **true** | ❌ **false** | **Retira Cliente (Take Away):** Retira el Cliente en el Local. |
| **2** | **No existe** | **≠ null** | ❌ **false** | ❌ **false** | **Delivery a Cargo de la Plataforma:** Logística a cargo de la Plataforma de Pedidos. |
| **3** | **Existe** | **null** | ✅ **true** | ✅ **true** | **Delivery Propio:** Logística a cargo del local. |
| **4** | **No existe** | **null** | ❌ **false** | ✅ **true** | **Delivery Propio:** Logística a cargo del local. |

> [!NOTE]
> **Regla de Prioridad:** La variable `NEWS.ownDelivery` (Logística Propia) en PedidosYa depende **exclusivamente** de que `ORDER.delivery.riderPickupTime` sea **vacío (null)**, independientemente de si el objeto `ORDER.pickup` existe o no en el mensaje.

---

