# Barreras para la actividad física – PulseUp

Proyecto de ciencia de datos de **Santiago Varela Peña** (SENA, Tecnología en Análisis y Desarrollo de Software, Ficha 3223873).

## Descripción

PulseUp es una aplicación móvil para el seguimiento de hábitos saludables. Este proyecto analiza datos abiertos para entender por qué las personas no hacen actividad física, de modo que la aplicación ofrezca funcionalidades que ataquen esas barreras.

## Propósito

Identificar las barreras más frecuentes para la actividad física y los factores asociados a la inactividad, y traducirlos en decisiones de diseño para PulseUp.

## Pregunta analítica

> ¿Qué barreras impiden a los adultos realizar actividad física de forma regular, qué factores sociodemográficos y de salud se asocian con la inactividad, y cuáles de esas barreras puede abordar una aplicación móvil de hábitos saludables?

## Datasets

| Dataset | Fuente | Formato | Uso en el proyecto |
|---|---|---|---|
| BRFSS 2023 | CDC de EE. UU. | `.XPT` (SAS) | Factores asociados a la inactividad |
| Actividad física en Bogotá D.C. | Secretaría Distrital de Salud – Datos Abiertos Colombia | `.csv` | Contexto local |
| Eurobarómetro 97.3 (ZA7888) | Comisión Europea – GESIS | `.dta` (Stata) | Barreras declaradas |

Los datos no están en el repositorio. Las instrucciones de descarga están en [`data/raw/README.md`](data/raw/README.md).

## Estructura

```
pulseup-ciencia-datos/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   └── raw/                      # datos crudos (no se suben a GitHub)
│       └── README.md             # cómo descargar cada dataset
├── notebooks/
│   └── diagnostico_calidad.ipynb # EV05: carga, inspección y diagnóstico
└── reports/
    └── informe_diagnostico.md    # EV05: informe técnico
```

## Cómo reproducir el diagnóstico

Requisitos: Python 3.10 o superior y Git.

```bash
# 1. Clonar el repositorio
git clone https://github.com/<tu-usuario>/pulseup-ciencia-datos.git
cd pulseup-ciencia-datos

# 2. Crear y activar el entorno virtual
python -m venv venv
venv\Scripts\activate          # Windows
# source venv/bin/activate     # macOS / Linux

# 3. Instalar dependencias
pip install -r requirements.txt

# 4. Descargar los datasets en data/raw/ (ver data/raw/README.md)

# 5. Abrir el notebook y ejecutar todas las celdas
jupyter notebook notebooks/diagnostico_calidad.ipynb
```

El notebook se debe ejecutar desde la carpeta `notebooks/`, porque busca los datos en `../data/raw/`.

## Evidencias

| Evidencia | Contenido |
|---|---|
| GA1-220501094-01-AA1-EV05 | Diagnóstico inicial de calidad de datos: `notebooks/diagnostico_calidad.ipynb` y `reports/informe_diagnostico.md` |
