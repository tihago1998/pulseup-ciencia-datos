# Datos crudos

Los archivos de esta carpeta **no se publican en GitHub**: el BRFSS supera el límite de tamaño de GitHub y la licencia de GESIS no permite redistribuir el Eurobarómetro. Para reproducir el diagnóstico, descárgalos y guárdalos aquí con estos nombres:

| Dataset | Dónde descargarlo | Archivo que debe quedar en `data/raw/` |
|---|---|---|
| BRFSS 2023 (CDC) | https://www.cdc.gov/brfss/annual_data/annual_2023.html → *2023 BRFSS Data (SAS Transport Format)* | `LLCP2023.XPT` |
| Actividad física en Bogotá D.C. | https://www.datos.gov.co/dataset/Proporci-n-de-personas-que-realizan-actividad-f-si/gx62-57mq → **Exportar** → CSV | `actividad_fisica_bogota.csv` |
| Eurobarómetro 97.3 (ZA7888) | https://search.gesis.org → buscar **ZA7888** → descargar con una cuenta gratuita de GESIS (formato Stata) | `ZA7888_v1-0-0.dta` |

**Notas**

- El ZIP del BRFSS trae un archivo llamado `LLCP2023.XPT ` con un **espacio al final del nombre**. Renómbralo a `LLCP2023.XPT`.
- Si GESIS solo te da el formato SPSS (`.sav`), también funciona: el notebook detecta los dos formatos.
