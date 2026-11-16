# Análisis Predictivo 2026 Q2 – Predicción de `condition` (new / used) en MercadoLibre

Miranda Contieri – Legajo 67125

Competencia de Kaggle `ap-2026-q-2`. Se predice si una publicación de MercadoLibre es nueva o usada. Métrica: accuracy.

## Aclaración importante sobre la submission

Les escribo para comunicarles que, preparándome para entregar el tercer examen, me di cuenta de que tengo un problema: no tengo guardada la versión exacta del notebook que creó la submission con el mejor score privado. Cuando la creé estaba haciendo pruebas en el modo "edición" de Kaggle y no sé exactamente qué parámetros usé, ya que los últimos días estuve probando múltiples combinaciones.

Por esta razón, quiero dejar en claro que en la entrega **no incluyo el notebook que genera la submission ganadora, sino uno parecido**, que genera otra submission con un score similar.

| Submission | Score público | Score privado |
|---|---|---|
| Ganadora (no reproducible) | 0.94757 | 0.94822 |
| Guardada (la que replica esta entrega) | 0.94752 | 0.94800 |

Considerar la submission guardada como la enviada me dejaría en el puesto 14 del leaderboard, en lugar del 13.

Comprendo completamente si deciden deducir puntos de mi nota por este error de prolijidad. A futuro voy a documentar y guardar correctamente todas las versiones en las que trabaje.

## Contenido del repositorio

| Archivo / carpeta | Descripción |
|---|---|
| `notebook-xlm-roberta-base/` | Notebook y salidas (`bert_feats_train.csv`, `bert_feats_test.csv`) del modelo XLM-RoBERTa base. |
| `notebook-roberta-base-bne-v5-ap-2026/` | Notebook y salidas del modelo RoBERTa base BNE. |
| `notebook-bert-titulos-ap/` | Notebook y salidas del modelo BERT sobre títulos. |
| `01-entrenar-modelo-final.ipynb` | Construye las features, entrena el ensamble final (LightGBM + XGBoost + CatBoost + meta-modelo logístico) y guarda los modelos en `modelo_final/`. |
| `02-predeir-test.ipynb` | Carga los modelos guardados, los aplica al test y genera `submission.csv`. |
| `notebook11-ap-2026 (1).ipynb` | Notebook de trabajo completo, del que se extrajeron los dos notebooks anteriores. Se incluye solo como referencia. |
| `Miranda_Contieri_67125_Ap_2026.pptx` | Presentación. |

## Orden de ejecución

1. Los tres notebooks de BERT (en GPU). Cada uno genera `bert_feats_train.csv` y `bert_feats_test.csv`.
2. `01-entrenar-modelo-final.ipynb`. Usa los datos de la competencia y los CSV de BERT, y escribe `modelo_final/`.
3. `02-predeir-test.ipynb`. Usa `modelo_final/` y el test, y escribe `submission.csv`.

Los notebooks asumen el entorno de Kaggle: los datos en `/kaggle/input/competitions/ap-2026-q-2/` y los CSV de BERT en las rutas indicadas en la variable `BERT_DIRS` del notebook 01. Si se ejecutan en otro entorno, hay que ajustar esas rutas.

## Modelo final

- **Features:** tabulares (aplanado de campos anidados, limpieza, precios relativos), target encoding fuera de fold, texto (TF-IDF con regresión logística fuera de fold, vecinos más cercanos por título), vecinos en el tiempo y dentro del mismo vendedor, desvío de cada publicación respecto de su vendedor, target encoding de pares, y las predicciones fuera de fold de los tres modelos BERT.
- **Modelos:** LightGBM (6 semillas), XGBoost (3 semillas) y CatBoost (3 semillas), cada uno con 5 folds estratificados.
- **Mezcla:** regresión logística sobre los logits de las predicciones fuera de fold.
- **Accuracy en validación cruzada (OOF):** aprox. 0.946.

## Reproducibilidad

- El BERT entrenado en GPU no es idéntico bit a bit entre corridas, por limitaciones de no determinismo de la GPU. Por eso se incluyen en el repositorio los CSV de BERT con los que se generó la submission, y los notebooks 01 y 02 parten de esos archivos.
- LightGBM y XGBoost usan todos los núcleos (`n_jobs=-1`), por lo que pueden aparecer diferencias mínimas en las probabilidades. Es muy improbable que cambien etiquetas finales.
- El notebook 02 usa las matrices de features de test que el notebook 01 deja guardadas en `modelo_final/`, porque esas features dependen de ajustes hechos sobre el train.
