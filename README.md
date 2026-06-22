# Grupo 2 - Series temporales y pronóstico de consumo eléctrico

Mini proyecto de forecasting para la asignatura **Aprendizaje Automático II**, dentro de la unidad de **Series Temporales y Pronóstico**.

El objetivo del proyecto es construir un flujo reproducible de predicción de consumo eléctrico a partir de datos históricos. El notebook desarrolla la preparación temporal, el análisis exploratorio, modelos univariantes, evaluación con backtesting, ajuste de hiperparámetros, incorporación de variables exógenas y comparación final de resultados.

El notebook principal del proyecto es `miniproyecto_series_temporales.ipynb`. Este archivo está pensado para ejecutarse dentro de **Google Colab**. Por ese motivo, los archivos generados durante la ejecución, como el dataset procesado, el modelo entrenado, los metadatos y otras salidas auxiliares, están pensados para guardarse en el **almacenamiento temporal de Colab**. 

## Problema de forecasting

Se busca predecir el consumo eléctrico de la **zona 1** para las próximas **24 horas**.

- **Variable temporal:** `fecha`
- **Variable objetivo original:** `consumo_zona_1`
- **Variable objetivo usada para modelar:** `y`
- **Frecuencia original:** registros cada 10 minutos
- **Frecuencia de trabajo:** horaria, usando promedio por hora
- **Horizonte de predicción:** 24 pasos horarios
- **Métrica principal:** MAE, error absoluto medio

## Dataset

El dataset trabajado en el notebook corresponde al archivo entregado por el docente y cargado como `alemania_consumo.csv`. Contiene 52.416 registros y 7 columnas. El rango temporal observado va desde `2017-01-01 00:00:00` hasta `2017-12-30 23:50:00`.

Columnas principales:

| Columna | Descripción dentro del proyecto |
|---|---|
| `fecha` | Marca temporal de cada observación. |
| `consumo_zona_1` | Consumo eléctrico de la zona 1. Es la variable objetivo. |
| `temperatura` | Variable meteorológica usada como exógena. |
| `humedad` | Variable meteorológica usada como exógena. |
| `velocidad_viento` | Variable meteorológica usada como exógena. |
| `flujo_difuso_general` | Variable de radiación usada como exógena. |
| `flujo_difuso` | Variable de radiación usada como exógena. |

Para alinear el trabajo con las pautas de entrega, se recomienda guardar el archivo original del docente en `data/raw/` y el dataset procesado horario en `data/processed/`.

## Estructura sugerida del repositorio

```text
miniproyecto-series-temporales/
├── README.md
├── requirements.txt
├── informe.pdf
├── notebooks/
│   └── miniproyecto_series_temporales.ipynb
├── data/
│   ├── raw/
│   │   └── tetuan_power_docente.csv
│   └── processed/
│       └── consumo_horario_procesado.csv
└── modelos/
    ├── mejor_modelo_tetuan.joblib
    └── mejor_modelo_tetuan.pkl
    └── mejor_modelo_tetuan_metadata.json
```


## Preparación de datos

El flujo aplicado en el notebook es el siguiente:

1. Carga del dataset.
2. Conversión de `fecha` a tipo datetime con formato `%m/%d/%Y %H:%M`.
3. Ordenamiento cronológico e indexación temporal.
4. Resampling horario mediante promedio.
5. Renombrado de `consumo_zona_1` como `y`.
6. Imputación de valores faltantes por bloque temporal:
   - interpolación temporal;
   - forward fill;
   - backward fill.
7. Separación temporal sin mezclar períodos:
   - train: 5.592 registros;
   - validación: 1.397 registros;
   - test: 1.747 registros.

## Modelado

Se trabajó con `skforecast` usando modelos recursivos y directos. La comparación inicial se hizo con Ridge y luego se ajustaron modelos más avanzados.

Modelos evaluados:

- Ridge univariante como modelo base.
- Random Forest univariante.
- LightGBM univariante.
- Random Forest con variables exógenas.
- LightGBM con variables exógenas.

Lags principales probados:

- `lags_24h`: 24 lags horarios.
- `lags_48h`: 48 lags horarios.
- `lags_diario_semanal`: `[1, 2, 3, 23, 24, 25, 47, 48, 49, 167, 168]`.

La evaluación se realizó con `TimeSeriesFold`, `backtesting_forecaster` y `grid_search_forecaster`, respetando el orden temporal de los datos.

## Resultados principales

| Modelo | Usa exógenas | Estrategia | MAE validación | MAE test |
|---|---:|---|---:|---:|
| RandomForest_univariante | No | Recursiva | 1221.84 | 1097.42 |
| LightGBM_univariante | No | Recursiva | 1232.77 | 1137.35 |
| RandomForest_exogenas | Sí | Recursiva | 1241.74 | 1085.57 |
| LightGBM_exogenas | Sí | Recursiva | 1238.49 | 1137.87 |

El mejor modelo final fue **RandomForest_exogenas**, con un **MAE en test de 1085.57**. Este modelo usa la variable objetivo histórica y las variables exógenas `temperatura`, `humedad`, `velocidad_viento`, `flujo_difuso_general` y `flujo_difuso`.

## Modelo guardado

El notebook guarda el modelo final y sus metadatos en:

```text
models/mejor_modelo_tetuan.joblib
models/mejor_modelo_tetuan.pkl
modelos/mejor_modelo_tetuan_metadata.json
```

Para reutilizarlo:

```python
import joblib

modelo = joblib.load("models/mejor_modelo_tetuan.joblib")
```

Como el modelo final usa variables exógenas, para predecir 24 horas hacia adelante se debe disponer de un DataFrame futuro con estas columnas:

```python
EXOG_COLS = [
    "temperatura",
    "humedad",
    "velocidad_viento",
    "flujo_difuso_general",
    "flujo_difuso"
]
```

## Ejecución en Google Colab

El archivo `miniproyecto_series_temporales.ipynb` está pensado para ejecutarse en **Google Colab**, no necesariamente en una instalación local de Jupyter. Los archivos que se generan durante el flujo del notebook están configurados para guardarse en el almacenamiento temporal de Colab.

Pasos recomendados:

1. Abrir `notebooks/miniproyecto_series_temporales.ipynb` en Google Colab.
2. Subir el archivo `alemania_consumo.csv` al entorno temporal de Colab o ajustar la ruta de carga según corresponda.
3. Ejecutar las celdas del notebook de arriba hacia abajo.
4. Durante la ejecución se generan archivos y carpetas temporales, por ejemplo:

```text
data/processed/consumo_horario_procesado.csv
modelos/mejor_modelo_tetuan.joblib
modelos/mejor_modelo_tetuan_metadata.json
```

Estos archivos quedan en el almacenamiento temporal de la sesión de Colab. Si se reinicia el entorno, se desconecta la sesión o se cierra Colab, pueden eliminarse. Por eso, para conservarlos, se deben descargar manualmente o copiar al repositorio antes de finalizar la sesión.

El archivo `requirements.txt` se incluye para documentar las librerías necesarias y facilitar la reproducción del proyecto si se desea ejecutarlo fuera de Colab.


## Conclusión

El proyecto cumple el flujo principal de forecasting solicitado: preparación temporal, análisis exploratorio, modelo base, backtesting, ajuste de hiperparámetros, comparación univariante/multivariante y guardado del modelo final. La incorporación de variables exógenas permitió obtener el menor error en test, aunque la mejora respecto al mejor modelo univariante fue moderada.
