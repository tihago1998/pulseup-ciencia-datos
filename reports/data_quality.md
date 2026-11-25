# Informe técnico: diagnóstico inicial de calidad de datos

**Autor:** Santiago Varela Peña · ADSO, Ficha 3223873 · SENA
**Notebook asociado:** [`notebooks/data_quality.ipynb`](../notebooks/data_quality.ipynb)

Todas las cifras de este informe provienen de las salidas del notebook (pandas 3.1.1).

## 1. Proyecto y pregunta analítica

**Proyecto:** Barreras para la actividad física – PulseUp.

**Pregunta analítica:** ¿Qué barreras impiden a los adultos realizar actividad física de forma regular, qué factores sociodemográficos y de salud se asocian con la inactividad, y cuáles de esas barreras puede abordar una aplicación móvil de hábitos saludables?

## 2. Descripción de los datasets

| Dataset | Fuente | Formato | Contenido | Relación con la pregunta |
|---|---|---|---|---|
| BRFSS 2023 | CDC de EE. UU. | `.XPT` (SAS) | Encuesta telefónica a adultos sobre salud y factores de riesgo. Se seleccionaron 14 variables (13 disponibles en 2023): sociodemográficas, actividad física, salud, IMC y peso muestral. | Factores asociados a la inactividad |
| Actividad física en Bogotá D.C. | Secretaría Distrital de Salud | `.csv` (latin-1, separado por `;`) | Porcentaje de personas según su frecuencia de actividad física, por localidad, en 2017 y 2021. | Contexto local |
| Eurobarómetro 97.3 (ZA7888) | Comisión Europea / GESIS | `.dta` (Stata) | Encuesta en los países de la UE que incluye el módulo de deporte y actividad física, con las razones para no practicarla. | Barreras declaradas |

## 3. Resultados de la inspección estructural

| Dataset | Filas | Columnas | Tipos de datos | Observaciones |
|---|---|---|---|---|
| BRFSS 2023 | 433 323 | 13 | `float64`: 13 | Subconjunto de 13 variables; `SLEPTIM1` no existe en 2023. Las variables categóricas se leen como números. |
| Bogotá | 168 | 4 | `str`: 3, `int64`: 1 | Datos agregados: 21 localidades × 4 categorías × 2 años. `Porcentaje` es texto con coma decimal. |
| Eurobarómetro 97.3 | 26 569 | 501 | `int8`: 322, `float64`: 166, `int16`: 8, `str`: 3, `int32`: 2 | Microdatos por encuestado. Respuestas cargadas como códigos numéricos porque `d9b` tiene etiquetas duplicadas. |

## 4. Valores faltantes, tipos de datos e inconsistencias

**BRFSS 2023**

- **Faltantes:** `_BMI5` 40 535 (9,35 %), `INCOME3` 8 075 (1,86 %), `EMPLOY1` 2 968 (0,68 %); `EDUCA`, `GENHLTH`, `ADDEPEV3` y `EXERANY2` tienen menos de 10 cada una.
- **Códigos de "no sabe / no responde":** `INCOME3` (77/99) 78 548 registros (18,13 %), `_AGEG5YR` (14) 7 779 (1,80 %), `EMPLOY1` (9) 4 713 (1,09 %), `ADDEPEV3` (7/9) 2 584 (0,60 %), `EDUCA` (9) 2 316 (0,53 %), `GENHLTH` (7/9) 1 258 (0,29 %), `_TOTINDA` (9) 1 251 (0,29 %) y `EXERANY2` (7/9) 1 249 (0,29 %). Pandas los trata como valores válidos.
- **Tipos:** las 13 columnas son `float64`, aunque 10 son categóricas.
- **Inconsistencias:** `_BMI5` va de 1 202 a 9 984 por sus dos decimales implícitos. Hay 252 filas idénticas, pero el subconjunto no tiene identificador: pueden ser personas distintas con las mismas respuestas.

**Bogotá**

- **Faltantes y duplicados:** ninguno.
- **Tipos:** `Porcentaje` es texto con coma decimal; todos sus valores son convertibles a número y están entre 0 y 100.
- **Inconsistencias:** el total de la ciudad ("Bogotá D.C.") aparece como una localidad más.
- **Cobertura y coherencia:** solo hay dos años (2017 y 2021, 84 registros cada uno); la suma de las 4 categorías no es exactamente 100,0 en 18 de las 42 combinaciones localidad-año (de 99,9 a 102,6; el máximo corresponde al total "Bogotá D.C." en 2021).

**Eurobarómetro 97.3**

- **Faltantes:** 163 de las 501 columnas tienen faltantes; las 20 columnas con más ausentes son todas `region_*` (entre 96,12 % y 98,11 %), porque cada una solo aplica a un país.
- **Duplicados:** ninguno.
- **Tipos:** las respuestas son códigos numéricos; su significado está en los metadatos del archivo.
- **Inconsistencias:** buscar el texto "DK" en los datos daría falsos positivos, porque "DK" es también el código ISO de Dinamarca en `isocntry`. Al identificar los códigos especiales con las etiquetas de valor, 167 de las 501 variables tienen registros "no sabe / no responde / no aplica".
- **Módulo de deporte:** en `qb1`, 11 161 encuestados (42,01 %) responden que nunca practican deporte; las barreras más mencionadas en `qb14` son falta de tiempo (10 729; 40,38 %), falta de motivación (6 531; 24,58 %) y discapacidad o enfermedad (4 435; 16,69 %) (conteos sin ponderar).

## 5. Principales hallazgos de calidad

| Dataset | Problema identificado | Evidencia encontrada | Posible impacto | Acción propuesta |
|---|---|---|---|---|
| BRFSS 2023 | Códigos de "no sabe / no responde" como números | `INCOME3` 77/99: 78 548 registros (18,13 %) | Distorsionan promedios y porcentajes | Convertirlos a faltantes; en ingresos, crear la categoría "No informa" |
| BRFSS 2023 | Valores faltantes en el IMC | `_BMI5`: 40 535 (9,35 %) | Menos registros para el análisis del IMC | Conservar; analizar con los casos disponibles |
| BRFSS 2023 | Tipos inadecuados | 13 columnas `float64` | Interpretación errónea y más memoria | Convertir a categorías y enteros |
| BRFSS 2023 | Escala del IMC | Rango de 1 202 a 9 984 | IMC 100 veces mayor | Dividir entre 100 |
| BRFSS 2023 | Filas idénticas sin identificador | 252 duplicados | Eliminarlas borraría personas reales | Conservar y crear un identificador |
| BRFSS 2023 | Variable de sueño ausente | `SLEPTIM1` no existe en 2023 | No se puede analizar el sueño | Usar salud general, IMC y depresión |
| Bogotá | Porcentaje como texto | `str` con coma decimal | Impide cálculos | Convertir a número |
| Bogotá | Total mezclado con localidades | Fila "Bogotá D.C." | Doble conteo | Separar la fila total |
| Eurobarómetro | Exceso de columnas | 501 columnas | Más memoria y confusión | Seleccionar el módulo de deporte |
| Eurobarómetro | Respuestas codificadas | 322 columnas `int8` | Datos ilegibles | Recodificar con los metadatos |
| Eurobarómetro | Faltantes estructurales | Las 20 columnas con más ausentes son `region_*` (96,12 % a 98,11 %) | No son errores | Excluir esas columnas |

## 6. Acciones de limpieza y preparación propuestas

1. **BRFSS:** convertir los códigos especiales en faltantes, dividir `_BMI5` entre 100, convertir los códigos a categorías con etiquetas en español, crear un identificador por registro, crear la categoría "No informa" en ingresos y excluir los registros sin respuesta válida en `EXERANY2`, porque sin ella no se puede saber si la persona es activa.
2. **Bogotá:** renombrar las columnas, convertir `Porcentaje` a número, estandarizar las categorías de frecuencia y separar el total de la ciudad.
3. **Eurobarómetro:** seleccionar `qb1`, `qb14_1` a `qb14_13`, país, sexo y edad; traducir los códigos con los metadatos y convertir las barreras en variables booleanas.
4. **Todos:** validar los resultados con `assert` y comparar la calidad antes y después.

## 7. Conclusiones

Los tres datasets se cargan correctamente y sirven para responder la pregunta, pero ninguno puede analizarse directamente. El BRFSS es la fuente más grande y su principal riesgo son los códigos especiales: el 18 % de los registros de ingresos no son valores reales. El archivo de Bogotá es pequeño y limpio, pero es agregado y requiere conversión de tipos. El Eurobarómetro es el único que pregunta directamente por las barreras, pero hay que reducirlo de 501 a unas pocas columnas y traducir sus códigos. Las limitaciones más importantes son la ausencia de la variable de sueño en el BRFSS 2023 y que las fuentes con datos por persona no son colombianas. Las acciones propuestas se implementan en `notebooks/data_preparation.ipynb`.
