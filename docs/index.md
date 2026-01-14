1. Inicialización de Contexto e Ingesta de Fuentes de Datos
Objetivo del Bloque:
Establecer los parámetros globales de la integración y realizar la ingesta inicial de los archivos de origen, normalizando su estructura para un procesamiento unificado.
Lógica de Negocio y Transformación:
Definición de Contexto de Integración:
Se configuran las variables de entorno que determinarán cómo se comportará la salida de datos. Esto incluye la definición de la Lista de Precios a utilizar (para asegurar que los precios coincidan con la estrategia comercial del canal de delivery), la identificación de la Plataforma Destino (ej. PedidosYa) y las Sucursales (Branch IDs) para las cuales se generará este catálogo.
Ingesta y Normalización (Wrapping):
El proceso lee dos fuentes de datos dispares provenientes del ERP:
Catálogo de Productos: Items individuales (ej. bebidas, platos principales).
Relaciones y Promociones: Estructuras compuestas (ej. combos, menús).
Para facilitar el procesamiento posterior, se aplica una técnica de "Envoltura" (Wrapping). Cada objeto crudo extraído del archivo original no se pasa directamente, sino que se encapsula en una nueva estructura estandarizada.
Impacto en la Estructura del JSON:
Los datos pasan de ser una lista plana de objetos heterogéneos a una lista de objetos tipados.
Antes (Raw ERP): { "id": 101, "name": "Hamburguesa" }
Después (Normalizado):
code
JSON
{
  "type": "item",       // Identificador del tipo de dato ("item" o "promo")
  "data": {             // Payload original encapsulado
     "id": 101,
     "name": "Hamburguesa",
     ...
  }
}
Resultado:
Se obtienen dos colecciones en memoria (catalog y relations) con una estructura homogénea, listas para ser validadas y transformadas sin perder la trazabilidad de su origen.





