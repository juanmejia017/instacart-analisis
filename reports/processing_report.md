# Reporte de procesamiento de datos – Instacart

**Actividad:** GA1-220501094-01-AA1-EV06 – Limpieza, análisis y estructuración de datos con Pandas
**Aprendiz:** Juan Diego Mejía Llano
**Cuaderno:** `notebooks/pandas_labs.ipynb`

**Pregunta orientadora:** ¿Cómo preparar y estructurar los datos de Instacart para obtener información confiable sobre los hábitos de compra?

---

## 1. Limpieza y estandarización (Lección 2)

### Decisiones de limpieza

| Problema | Columna / tabla | Decisión | Justificación |
|---|---|---|---|
| 15 pedidos duplicados | `orders` | Eliminar | Todos ocurren el miércoles a las 2 a. m.: fallo del sistema, no pedidos reales |
| Tipos `int64` / `float64` | `orders`, `order_prods` | Corregir | Tipos más pequeños (`int8`, `int16`, `int32`) y `Int16` nullable donde hay nulos |
| 28.817 nulos | `days_since_prior_order` | Conservar | Son primeros pedidos: no existe pedido anterior |
| 1.258 nulos | `product_name` | Rellenar con `'Unknown'` | El producto existe y aparece en pedidos |
| 836 nulos | `add_to_cart_order` | Conservar | Pedidos de más de 64 productos: el sistema solo registró hasta la posición 64 |
| 403 espacios dobles y 16 espacios invisibles | `product_name` | Corregir | Se estandarizó el texto con `.str.replace`, `.str.strip` y `.str.title` |
| 141 nombres repetidos | `product_name` | Conservar y documentar | Tienen `product_id` distintos y 268 de 280 aparecen en pedidos |

### Informe de cambios del pipeline

| Tabla | Paso | Filas | Nulos | Memoria (MB) |
|---|---|---|---|---|
| orders | 0 crudo | 478.967 | 28.819 | 23,0 |
| orders | 1 sin duplicados | 478.952 | 28.817 | 23,0 |
| orders | 2 tipos | 478.952 | 28.817 | 7,2 |
| products | 0 crudo | 49.694 | 1.258 | 3,1 |
| products | 1 nombres | 49.694 | 0 | 3,1 |
| order_prods | 0 crudo | 4.545.007 | 836 | 145,4 |
| order_prods | 1 tipos | 4.545.007 | 836 | 54,5 |

La validación final con `assert` se superó y el pipeline produce exactamente las mismas tablas que la limpieza manual.

### Hallazgos del Reto 2

- `order_prods` no tiene duplicados exactos ni pares `order_id` + `product_id` repetidos; `reordered` solo contiene 0 y 1.
- `aisles` y `departments` no necesitan limpieza de texto.
- La estandarización cambió 6.321 nombres de productos.
- Los productos `'Unknown'` aparecen en 9.547 filas de 8.254 pedidos; se conservan porque son compras reales.

---

## 2. Filtrado, ordenamiento y agrupación (Lección 3)

### Perfil de hábitos de compra por segmento

| Segmento | Clientes | % clientes | Pedidos | Espera mediana (días) |
|---|---|---|---|---|
| Ocasional (1 pedido) | 55.357 | 35,2 % | 55.357 | 16 |
| Regular (2 a 4) | 71.553 | 45,4 % | 191.649 | 11 |
| Frecuente (5 o más) | 30.527 | 19,4 % | 231.946 | 6 |

### Principales hallazgos

- La hora pico es las 10 a. m. (40.578 pedidos); la combinación más activa es lunes a las 10 a. m. (7.881).
- El domingo es el día con más pedidos (84.090) y clientes (57.661).
- El 19,4 % de los clientes (frecuentes) hace casi la mitad de los pedidos y vuelve cada 6 días.
- Todos los segmentos compran sobre todo en la tarde, pero los frecuentes compran más en la mañana (35,2 % frente a 30,0 % de los ocasionales).
- El 9,31 % de los pedidos (44.577) se hace exactamente 7 días después del anterior: ciclo de compra semanal.
- La tasa de recompra baja con la posición en el carrito: 0,679 en la posición 1 y 0,617 en la 5.

---

## 3. Combinación de datasets (Lección 4)

### Validación de la tabla maestra

- La tabla maestra parte de `order_prods` (tabla de hechos) y se une con `products`, `aisles`, `departments` y `orders`.
- Todas las uniones usan `how='left'` y `validate='m:1'`; la tabla mantiene 4.545.007 filas en cada paso y termina con 14 columnas.
- La auditoría con `indicator=True` mostró 28.906 pedidos (6 %) sin productos registrados; con `how='inner'` habrían desaparecido sin aviso.

### Principales hallazgos

- Producto más pedido: Banana (66.050 veces, tasa de recompra 0,844).
- Departamento con más artículos: `produce` (1.328.302); `dairy eggs` tiene la mayor tasa de recompra entre los tres primeros (0,671).
- Hay 18 productos líderes de fidelidad, uno por departamento (entre productos con al menos 1.000 pedidos). El más fiel es Whole Organic Omega 3 Milk (0,863).
- Pasillos con mayor tasa de recompra: `milk` (0,781), `water seltzer sparkling water` (0,731) y `fresh fruits` (0,718); la más baja es `spices seasonings` (0,152).
- En la madrugada, `beverages` entra al top 3 de departamentos en lugar de `snacks`.
- El alcohol se compra sobre todo el viernes (3.814 artículos).

---

## 4. Archivos generados (`data/processed/`)

| Archivo | Contenido | ¿En GitHub? |
|---|---|---|
| `instacart_maestra.parquet` | Tabla maestra (4.545.007 × 14) | No: pesa ~104 MB y supera el límite de 100 MB de GitHub. Se regenera ejecutando el cuaderno |
| `resumen_departamentos.csv` | Artículos y tasa de recompra por departamento | Sí |
| `ficha_productos.parquet` | Una fila por producto (45.573 × 7) | Sí |
| `lideres_por_departamento.csv` | Producto con mayor tasa de recompra de cada departamento | Sí |

---

## 5. Conclusión

Con la limpieza, el análisis y la combinación de las cinco tablas se obtuvo una tabla maestra validada que permite responder con confianza preguntas sobre los hábitos de compra: cuándo compran los clientes, con qué frecuencia vuelven y qué productos recompran. Cada decisión de limpieza quedó justificada y cada unión se verificó para no crear ni perder registros.
