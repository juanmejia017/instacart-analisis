# Reporte de calidad de datos – Instacart

**Actividad:** GA1-220501094-01-AA1-EV04 – Fundamentos en Pandas
**Aprendiz:** Juan Diego Mejía Llano
**Cuaderno:** `notebooks/data_quality.ipynb`

Diagnóstico de la estructura y calidad inicial de los cinco archivos de Instacart. En esta fase no se corrigió ni se eliminó ningún registro.

## Archivos analizados

| Archivo | Filas | Columnas | Valores ausentes | Una fila representa |
|---|---|---|---|---|
| `instacart_orders.csv` | 478.967 | 6 | 28.819 | Un pedido |
| `products.csv` | 49.694 | 4 | 1.258 | Un producto |
| `aisles.csv` | 134 | 2 | 0 | Un pasillo |
| `departments.csv` | 21 | 2 | 0 | Un departamento |
| `order_products.csv` | 4.545.007 | 4 | 836 | Un artículo dentro de un pedido |

Todos los archivos usan punto y coma (`;`) como separador.

## Relación entre las tablas

| Campo | Relaciona | Significado |
|---|---|---|
| `order_id` | `orders` ↔ `order_products` | Qué artículos tiene cada pedido |
| `product_id` | `products` ↔ `order_products` | Qué producto es cada artículo comprado |
| `aisle_id` | `products` ↔ `aisles` | A qué pasillo pertenece cada producto |
| `department_id` | `products` ↔ `departments` | A qué departamento pertenece cada producto |

## Problemas de calidad detectados

| Archivo | Problema | Detalle |
|---|---|---|
| `instacart_orders.csv` | Filas duplicadas | 15 filas completamente repetidas |
| `instacart_orders.csv` | Valores ausentes | 28.819 en `days_since_prior_order`; coinciden con los primeros pedidos (`order_number = 1`) |
| `instacart_orders.csv` | Tipo de dato | `days_since_prior_order` es `float64` aunque guarda días enteros, por los ausentes |
| `products.csv` | Valores ausentes | 1.258 productos sin `product_name` |
| `aisles.csv` | Categoría sin nombre | El pasillo 100 se llama `missing` |
| `departments.csv` | Categoría sin nombre | El departamento 21 se llama `missing` |
| `order_products.csv` | Valores ausentes | 836 en `add_to_cart_order`; por eso la columna es `float64` |

## Indicadores básicos

- 478.952 pedidos distintos y 157.437 clientes distintos.
- Hora con más pedidos: 10 a. m. (40.578 pedidos).
- Día con más pedidos: domingo (84.090).
- El 59,05 % de los artículos son recompras (2.683.838 de 4.545.007).
- Espera media entre pedidos: 11,10 días (mediana 7).

## Problemas que pasan a la etapa de limpieza

Estos hallazgos se resuelven en `notebooks/pandas_labs.ipynb` (Lección 2) y se documentan en `reports/processing_report.md`.
