# Barreras para la actividad física – PulseUp

Proyecto de ciencia de datos de **Santiago Varela Peña** (SENA, Tecnología en Análisis y Desarrollo de Software, Ficha 3223873).

## Descripción

PulseUp es una aplicación móvil para el seguimiento de hábitos saludables. Este proyecto analiza datos abiertos para entender por qué las personas no hacen actividad física, de modo que la aplicación ofrezca funcionalidades que ataquen esas barreras.

## Propósito

Identificar las barreras más frecuentes para la actividad física y los factores asociados a la inactividad, y traducirlos en decisiones de diseño para PulseUp.

## Pregunta analítica

> ¿Qué barreras impiden a los adultos realizar actividad física de forma regular, qué factores sociodemográficos y de salud se asocian con la inactividad, y cuáles de esas barreras puede abordar una aplicación móvil de hábitos saludables?

## Fuentes de datos

| Dataset | Fuente | Formato | Uso en el proyecto |
|---|---|---|---|
| BRFSS 2023 | CDC de EE. UU. | `.XPT` (SAS) | Factores asociados a la inactividad |
| Actividad física en Bogotá D.C. | Secretaría Distrital de Salud – Datos Abiertos Colombia | `.csv` | Contexto local |
| Eurobarómetro 97.3 (ZA7888) | Comisión Europea – GESIS | `.dta` (Stata) | Barreras declaradas |

Los datos originales no están en el repositorio. Las instrucciones de descarga están en [`data/raw/README.md`](data/raw/README.md).

## Estructura

```
pulseup-ciencia-datos/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   ├── raw/                                      # datos originales, sin modificar (no se publican)
│   │   └── README.md                             # cómo descargar cada dataset
│   └── processed/                                # datos limpios y estructurados
│       ├── brfss_clean.parquet
│       ├── brfss_state_inactivity.csv
│       ├── bogota_physical_activity_clean.csv
│       ├── eurobarometer_barriers_indicators.csv
│       ├── project_indicators.csv
│       └── eurobarometer_sport_clean.parquet     # no se publica (licencia GESIS)
├── notebooks/
│   ├── data_quality.ipynb                        # diagnóstico de calidad
│   └── data_preparation.ipynb                    # limpieza, análisis y estructuración
└── reports/
    └── data_quality.md                           # informe del diagnóstico
```

## Transformaciones realizadas (`data_preparation.ipynb`)

Cada paso es una función reutilizable; las funciones se encadenan con `pipe()` y se validan con `assert`. El notebook registra filas, nulos y memoria después de cada paso y compara la calidad antes y después.

**Lección 2 – Limpieza y estandarización**

| Dataset | Transformaciones |
|---|---|
| BRFSS 2023 | Identificador por registro (las filas idénticas son personas distintas y se conservan); códigos de "no sabe / no responde" (7/9, 77/99, 14) convertidos a faltantes; exclusión de los registros sin respuesta válida en actividad física; IMC dividido entre 100; códigos traducidos a categorías en español; ingresos sin respuesta agrupados como "No informa"; variables derivadas `inactivo`, `grupo_edad` y `categoria_imc`; se descartan `_TOTINDA` y `_AGEG5YR` por redundantes. |
| Bogotá | Nombres de columnas estandarizados; texto limpio; porcentaje convertido de texto con coma decimal a número; frecuencias con etiquetas cortas y ordenadas; marca del total de la ciudad. |
| Eurobarómetro | Selección del módulo de deporte (`qb1`, `qb14_1` a `qb14_13`), país, sexo y edad; códigos traducidos con los metadatos del archivo; barreras como variables booleanas; códigos de país unificados. |

**Lección 3 – Filtrado, ordenamiento y agrupación**

Filtros con condiciones booleanas, `isin()`, `between()` y `query()`; resúmenes con `groupby()` y `agg()`; tablas dinámicas con `pivot_table()` y `unstack()`. Indicadores calculados:

1. Porcentaje de adultos inactivos (ponderado) por edad y sexo.
2. Brecha de inactividad por nivel de ingresos.
3. Inactividad según salud general, categoría de IMC y depresión.
4. Barreras más mencionadas en Europa y su variación por edad.
5. Localidades de Bogotá sin actividad física y su cambio entre 2017 y 2021.

**Lección 4 – Combinación y estructuración**

Los tres datasets describen poblaciones distintas y no comparten identificadores, por lo que no se unen fila por fila. Se hacen solo combinaciones con sentido, validadas con `validate` e `indicator`:

- `merge()` del BRFSS con una tabla de estados (código FIPS → nombre y región).
- `merge()` de las barreras con una tabla de funcionalidades de PulseUp que indica si la aplicación puede abordarlas.
- `concat()` de los indicadores de las tres fuentes en `project_indicators.csv`.

Los resultados se exportan a `data/processed/` y se verifican leyéndolos de nuevo.

## Cómo ejecutar

Requisitos: Python 3.10 o superior.

**En local (Jupyter o VSCode)**

```bash
python -m venv venv
venv\Scripts\activate            # Windows
# source venv/bin/activate       # macOS / Linux
pip install -r requirements.txt
# descargar los datasets en data/raw/ (ver data/raw/README.md)
cd notebooks
jupyter nbconvert --to notebook --execute --inplace data_quality.ipynb
jupyter nbconvert --to notebook --execute --inplace data_preparation.ipynb
```

**En Google Colab**

1. Sube los datasets a Google Drive en la carpeta `MyDrive/pulseup-datasets/`.
2. Abre el notebook desde **Archivo → Abrir cuaderno → GitHub** (`tihago1998/pulseup-ciencia-datos`).
3. Ejecuta todas las celdas y autoriza el acceso a Drive. Los archivos procesados se guardan en `MyDrive/pulseup-datasets/processed/`.

## Ramas

| Rama | Uso |
|---|---|
| `main` | Versión estable |
| `develop` | Integración de las actividades |
| `feature/project-data-preparation` | Preparación, limpieza y estructuración de datos |

## Evidencias

| Evidencia | Contenido |
|---|---|
| GA1-220501094-01-AA1-EV05 (diagnóstico de calidad de datos) | `notebooks/data_quality.ipynb` y `reports/data_quality.md` |
| GA1-220501094-01-AA1-EV07 | `notebooks/data_preparation.ipynb` y `data/processed/` |
