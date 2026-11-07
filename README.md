# instacart-analisis

Proyecto de análisis de datos de **Instacart** con Python y Pandas, desarrollado en el programa **Análisis y Desarrollo de Software (ADSO)** del SENA.

**Aprendiz:** Juan Diego Mejía Llano

El proyecto parte de cinco archivos CSV crudos de Instacart y los lleva, por fases, hasta una tabla limpia, combinada y validada que permite estudiar los hábitos de compra de los clientes: cuándo compran, con qué frecuencia vuelven y qué productos recompran.

---

## Actividades

| Actividad | Descripción | Cuaderno | Reporte |
|---|---|---|---|
| GA1-220501094-01-AA1-EV04 | Fundamentos en Pandas: exploración y diagnóstico de calidad de los datos | `notebooks/data_quality.ipynb` | `reports/data_quality.md` |
| GA1-220501094-01-AA1-EV06 | Limpieza, análisis y estructuración de datos con Pandas (lecciones 2, 3 y 4) | `notebooks/pandas_labs.ipynb` | `reports/processing_report.md` |

**Pregunta orientadora (EV06):** ¿Cómo preparar y estructurar los datos de Instacart para obtener información confiable sobre los hábitos de compra?

---

## Estructura del proyecto

```
instacart-analisis/
├── README.md
├── .gitignore
├── requirements.txt
│
├── data/
│   ├── raw/                  <- los cinco CSV originales (no se suben a GitHub)
│   │   ├── instacart_orders.csv
│   │   ├── order_products.csv
│   │   ├── products.csv
│   │   ├── aisles.csv
│   │   └── departments.csv
│   └── processed/            <- archivos generados por pandas_labs.ipynb
│
├── notebooks/
│   ├── 00_entorno.ipynb      <- comprobación del entorno
│   ├── data_quality.ipynb    <- EV04: exploración y diagnóstico
│   └── pandas_labs.ipynb     <- EV06: lecciones 2, 3 y 4
│
└── reports/
    ├── data_quality.md       <- reporte de calidad de datos (EV04)
    └── processing_report.md  <- reporte de procesamiento (EV06)
```

---

## Datos

| Archivo | Una fila es | Clave |
|---|---|---|
| `instacart_orders.csv` | Un pedido: cliente, número de pedido, día (`order_dow`, 0 = domingo), hora y días desde el pedido anterior | `order_id` |
| `order_products.csv` | Un producto dentro de un pedido, su posición en el carrito y si es recompra (`reordered`) | `order_id` + `product_id` |
| `products.csv` | Un producto del catálogo con su pasillo y departamento | `product_id` |
| `aisles.csv` | Un pasillo | `aisle_id` |
| `departments.csv` | Un departamento | `department_id` |

Todos los archivos usan punto y coma (`;`) como separador.

```
orders ── 1:N ── order_products ── N:1 ── products ── N:1 ── aisles
                                                └── N:1 ── departments
```

---

## Requisitos

- Python 3.x
- Pandas, PyArrow y JupyterLab (versiones exactas en `requirements.txt`)
- Git

---

## Instrucciones de ejecución

1. Clonar el repositorio y entrar a la carpeta:
   ```powershell
   git clone https://github.com/juanmejia017/instacart-analisis.git
   cd instacart-analisis
   ```
2. Crear y activar el entorno virtual (Windows PowerShell):
   ```powershell
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```
3. Instalar las dependencias:
   ```powershell
   pip install -r requirements.txt
   ```
4. Copiar los cinco archivos CSV de Instacart en `data/raw/`.
5. Abrir Jupyter desde la raíz del proyecto:
   ```powershell
   jupyter lab
   ```
6. Ejecutar los cuadernos en orden con **Kernel → Restart Kernel and Run All Cells**:
   1. `notebooks/data_quality.ipynb`
   2. `notebooks/pandas_labs.ipynb`

`pandas_labs.ipynb` debe ejecutarse completo y en orden: cada lección usa las variables de la anterior. Tarda unos segundos en cargar `order_products.csv` (~90 MB) y en construir la tabla maestra.

---

## Contenido de `pandas_labs.ipynb`

| Lección | Tema | Laboratorio guiado | Reto |
|---|---|---|---|
| 2 | Limpieza y estandarización de datos crudos | Pipeline de limpieza con informe de cambios | 7 ejercicios |
| 3 | Filtrado, ordenamiento y agrupación | Perfil de hábitos de compra | 7 ejercicios |
| 4 | Combinación y estructuración de múltiples datasets | Tabla de análisis por producto | 7 ejercicios |

Cada ejercicio incluye el código ejecutado, su resultado y una interpretación breve.

---

## Archivos generados

Al ejecutar `pandas_labs.ipynb` se generan en `data/processed/`:

| Archivo | Contenido | En GitHub |
|---|---|---|
| `instacart_maestra.parquet` | Tabla maestra con las cinco tablas combinadas (4.545.007 filas × 14 columnas) | No (~104 MB, supera el límite de GitHub) |
| `resumen_departamentos.csv` | Artículos y tasa de recompra por departamento | Sí |
| `ficha_productos.parquet` | Una fila por producto: pedidos, clientes, tasa de recompra y posición media | Sí |
| `lideres_por_departamento.csv` | Producto con mayor tasa de recompra de cada departamento | Sí |

Los CSV originales de `data/raw/` y la tabla maestra no se versionan (ver `.gitignore`); se obtienen copiando los datos y ejecutando el cuaderno.

---

## Principales resultados

- **Limpieza:** se eliminaron 15 pedidos duplicados, se corrigieron los tipos de dato (la memoria de `orders` bajó de 23,0 a 7,2 MB y la de `order_products` de 145,4 a 54,5 MB), se rellenaron 1.258 nombres de producto con `'Unknown'` y se estandarizó el texto de 6.321 nombres.
- **Hábitos de compra:** la hora pico es las 10 a. m. y el domingo es el día con más pedidos. El 19,4 % de los clientes (frecuentes) hace casi la mitad de los pedidos y vuelve cada 6 días.
- **Tabla maestra:** las cinco tablas se combinaron sin crear ni perder filas (validación con `validate` y `assert`). 28.906 pedidos (6 %) no tienen productos registrados.
- **Productos:** Banana es el producto más pedido (66.050 veces); Whole Organic Omega 3 Milk es el de mayor tasa de recompra (0,863) entre los productos con al menos 1.000 pedidos.

El detalle de decisiones, cifras e interpretaciones está en `reports/processing_report.md`.

---

## Flujo de trabajo con Git

- `main`: versión estable.
- `develop`: integración de los avances.
- Ramas de trabajo por actividad (`feat/explorando-instacart`, `feature/pandas-data-processing`), que se integran a `develop` y luego a `main` mediante Pull Requests.
- Commits con convención *Conventional Commits* (`feat:`, `docs:`, `chore:`).
