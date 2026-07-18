---
tags: [estadistica, probabilidad, machine-learning, validacion, metricas]
aliases: [Estadistica y Metricas de ML, Fundamentos Estadisticos ML]
---
# Parte 5 — Ciencia de Datos: Estadística, Probabilidad y Fundamentos de Validación

> Esta es la parte que le da **rigor** a todo lo anterior. De poco sirve tener el pipeline perfecto ([[03-Arquitectura-Empresarial-de-Datos-y-ML]]) si el modelo que entrena al final no se valida correctamente. Nota: el usuario indicó que faltan más partes — esta cubre estadística/probabilidad/EDA/feature engineering/validación/métricas; temas como álgebra lineal para DL, arquitecturas de redes neuronales específicas, NLP, o MLOps avanzado quedan para partes futuras.

---

## Tabla de Contenidos

1. [[#Estadística|Estadística]]
2. [[#Probabilidad|Probabilidad]]
3. [[#EDA (Exploratory Data Analysis / Análisis Exploratorio de Datos)|EDA (Exploratory Data Analysis / Análisis Exploratorio de Datos)]]
4. [[#Feature Engineering|Feature Engineering]]
5. [[#Feature Selection|Feature Selection]]
6. [[#Validación y Cross-Validation|Validación y Cross-Validation]]
7. [[#Bias y Variance (Sesgo y Varianza)|Bias y Variance (Sesgo y Varianza)]]
8. [[#Overfitting y Underfitting|Overfitting y Underfitting]]
9. [[#Regularización|Regularización]]
10. [[#Métricas de Clasificación|Métricas de Clasificación]]
	1. [[#Métricas de Clasificación#Matriz de Confusión (la base de todo lo demás)|Matriz de Confusión (la base de todo lo demás)]]
	2. [[#Métricas de Clasificación#Precision (Precisión)|Precision (Precisión)]]
	3. [[#Métricas de Clasificación#Recall (Sensibilidad / Exhaustividad)|Recall (Sensibilidad / Exhaustividad)]]
	4. [[#Métricas de Clasificación#F1-Score|F1-Score]]
	5. [[#Métricas de Clasificación#Curva ROC y AUC|Curva ROC y AUC]]
11. [[#Métricas de Regresión (Predicción de Valores Numéricos)|Métricas de Regresión (Predicción de Valores Numéricos)]]
	1. [[#Métricas de Regresión (Predicción de Valores Numéricos)#RMSE (_Root Mean Squared Error_)|RMSE (_Root Mean Squared Error_)]]
	2. [[#Métricas de Regresión (Predicción de Valores Numéricos)#MAE (_Mean Absolute Error_)|MAE (_Mean Absolute Error_)]]
	3. [[#Métricas de Regresión (Predicción de Valores Numéricos)#MAPE (_Mean Absolute Percentage Error_)|MAPE (_Mean Absolute Percentage Error_)]]
	4. [[#Métricas de Regresión (Predicción de Valores Numéricos)#SMAPE (_Symmetric Mean Absolute Percentage Error_)|SMAPE (_Symmetric Mean Absolute Percentage Error_)]]
12. [[#Cómo Elegir la Métrica Correcta (Resumen Práctico)|Cómo Elegir la Métrica Correcta (Resumen Práctico)]]
13. [[#Ver también|Ver también]]
14. [[#Pendiente para próximas partes|Pendiente para próximas partes]]

---

## Estadística

**Qué es:** La disciplina que estudia cómo **recolectar, resumir e interpretar datos**, y cómo hacer afirmaciones confiables sobre un fenómeno a partir de una muestra limitada de él.

Se divide en dos grandes ramas:

- **Estadística descriptiva:** resume lo que _ya observaste_ (medias, medianas, desviaciones estándar, distribuciones). No intenta generalizar más allá de los datos que tienes.
- **Estadística inferencial:** usa una **muestra** para hacer afirmaciones sobre una **población** completa que no puedes observar por completo (ej. "encuestamos a 1,000 clientes de 500,000 y con eso inferimos qué opina el 90% de todos").

**Por qué importa para Data Science:** Casi todo lo que haces —desde un simple promedio en un dashboard hasta la validación de un modelo de ML— descansa sobre supuestos estadísticos. Sin entender estadística, es fácil confundir ruido con señal.

---

## Probabilidad

**Qué es:** La rama de las matemáticas que **cuantifica la incertidumbre** — qué tan probable es que ocurra un evento, dado lo que sabemos.

**Por qué importa:** Es el lenguaje matemático subyacente de casi todo el ML: un modelo de clasificación no dice "esto ES spam", dice "esto tiene 87% de probabilidad de ser spam". Conceptos como el **Teorema de Bayes** (actualizar una creencia a partir de nueva evidencia) son la base de algoritmos enteros (Naive Bayes) y de cómo pensar correctamente sobre métricas como precision/recall en contextos de clases desbalanceadas.

**Relación estadística-probabilidad:** La probabilidad te dice, si conoces el proceso que genera los datos, qué esperar. La estadística hace lo inverso: observas los datos y tratas de inferir el proceso que los generó. Son las dos caras de la misma moneda.

---

## EDA (Exploratory Data Analysis / Análisis Exploratorio de Datos)

**Qué es:** El proceso, generalmente el primer paso real de cualquier proyecto, de **explorar un dataset antes de modelarlo**: revisar distribuciones, valores faltantes, outliers, correlaciones entre variables, y formular hipótesis iniciales — típicamente con estadística descriptiva y visualizaciones.

**Por qué existe como paso formal:** Modelar sin haber explorado antes es la causa número uno de errores costosos — entrenar un modelo sobre datos con valores faltantes mal manejados, escalas inconsistentes, o fugas de información (_data leakage_) que nadie detectó. El EDA existe para **conocer tus datos antes de confiar en ellos**.

**Qué revela típicamente un buen EDA:**

- Valores faltantes (¿son aleatorios o siguen un patrón sospechoso?).
- Outliers (¿son errores de captura o información legítima?).
- Distribución de cada variable (¿está sesgada? ¿necesita transformación?).
- Correlaciones entre variables (¿hay redundancia? ¿hay una relación demasiado perfecta con el target, señal de _data leakage_?).
- Balance de clases (si es un problema de clasificación).

---

## Feature Engineering

**Qué es:** El proceso de **crear, transformar o combinar variables** para hacerlas más útiles para un modelo — ver la definición base en [[01-Fundamentos-y-Panorama-General#9. ¿Qué significa feature?]].

**Por qué existe:** Los datos crudos rara vez están en la forma óptima para que un modelo aprenda. Ejemplos típicos:

- Extraer "día de la semana" y "es fin de semana" de una fecha cruda.
- Combinar "ingresos" y "número de dependientes" en "ingreso per cápita del hogar".
- Codificar variables categóricas en formato numérico (_one-hot encoding_, _target encoding_).
- Escalar variables numéricas para que estén en rangos comparables (_normalización_, _estandarización_).

**Por qué sigue importando incluso con Deep Learning:** El Deep Learning es célebre por poder aprender features automáticamente a partir de datos crudos (imágenes, texto) — pero en datos **tabulares** (el tipo de dato más común en la empresa, incluyendo forecasting y WFM), el feature engineering manual todavía suele superar a los enfoques automáticos, porque el conocimiento del negocio humano sigue aportando información que el modelo, por sí solo, no puede inferir de los datos.

---

## Feature Selection

**Qué es:** El proceso de **elegir cuáles features usar** de entre todas las disponibles, descartando las que no aportan valor predictivo o que incluso perjudican al modelo.

**Por qué existe (más allá de "menos es más"):**

1. **Reduce overfitting** (ver abajo) — más features irrelevantes dan al modelo más oportunidades de "memorizar ruido".
2. **Reduce costo computacional** y tiempo de entrenamiento/inferencia.
3. **Mejora la interpretabilidad** — un modelo con 8 features bien elegidas es mucho más fácil de explicar a un stakeholder de negocio que uno con 200.
4. Evita la **maldición de la dimensionalidad**: a medida que agregas más dimensiones (features), los datos se vuelven más "dispersos" en ese espacio, y el modelo necesita exponencialmente más datos para aprender patrones confiables.

---

## Validación y Cross-Validation

**Validación:** El proceso de medir qué tan bien generaliza un modelo a datos que **no vio durante el entrenamiento**, típicamente separando los datos en un conjunto de entrenamiento (train) y uno de prueba (test), o agregando un tercer conjunto de validación intermedio para ajustar hiperparámetros sin "contaminar" el test final.

**Por qué es indispensable:** Medir el error de un modelo sobre los **mismos datos con los que se entrenó** es engañoso — un modelo suficientemente complejo puede memorizar perfectamente los datos de entrenamiento (error ≈ 0) y aun así ser inútil con datos nuevos. La validación es la única forma honesta de estimar cómo se comportará el modelo en el mundo real.

**Cross-Validation (validación cruzada):** En vez de hacer una sola división train/test, se divide el dataset en _k_ partes ("folds"), y se repite el proceso de entrenar con _k-1_ partes y validar con la restante, _k_ veces (rotando cuál parte queda fuera cada vez). El resultado final es el **promedio** de las _k_ mediciones.

**Por qué existe (en vez de una sola división train/test):** Una sola división puede ser "afortunada" o "desafortunada" por azar (¿qué pasa si justo los casos más difíciles cayeron todos en el test?). Cross-validation da una estimación **mucho más robusta y menos dependiente del azar** de cómo se comportará el modelo, usando todos los datos tanto para entrenar como para validar (en distintas rondas).

---

## Bias y Variance (Sesgo y Varianza)

Son los dos tipos fundamentales de error que puede tener un modelo, y **están en tensión entre sí** (el famoso _bias-variance tradeoff_):

- **Bias (sesgo):** el error que viene de que el modelo es **demasiado simple** para capturar el patrón real de los datos (ej. ajustar una línea recta a una relación que en realidad es curva). Un modelo con alto bias falla de forma sistemática, incluso en los datos de entrenamiento — esto es **underfitting**.
- **Variance (varianza):** el error que viene de que el modelo es **demasiado sensible** a las particularidades exactas de los datos de entrenamiento — captura ruido, no solo señal. Un modelo con alta varianza funciona excelente en entrenamiento pero mal en datos nuevos — esto es **overfitting**.

```
Modelo muy simple  →  Alto bias, baja varianza  →  UNDERFITTING
Modelo muy complejo →  Bajo bias, alta varianza  →  OVERFITTING
```

El objetivo de todo el proceso de modelado (elegir la complejidad correcta, la [[#Regularización|regularización]] correcta, suficientes datos) es encontrar el **punto intermedio** que minimiza el error total.

---

## Overfitting y Underfitting

**Overfitting (sobreajuste):** El modelo "memorizó" los datos de entrenamiento, incluyendo su ruido específico, en vez de aprender el patrón general subyacente. Síntoma clásico: **excelente desempeño en entrenamiento, mal desempeño en test/validación**.

**Underfitting (subajuste):** El modelo es demasiado simple o no se entrenó lo suficiente para siquiera capturar el patrón en los datos de entrenamiento. Síntoma clásico: **mal desempeño tanto en entrenamiento como en test** — el modelo ni siquiera es bueno con los datos que ya vio.

**Por qué esta distinción es la más importante de todo el diagnóstico de modelos:** te dice **en qué dirección actuar**. Si tienes overfitting, necesitas simplificar el modelo, conseguir más datos, o regularizar más. Si tienes underfitting, necesitas un modelo más complejo, mejores features, o entrenar más tiempo — acciones opuestas. Diagnosticar mal esto (por ejemplo, agregar complejidad a un modelo que ya tiene overfitting) empeora el problema.

---

## Regularización

**Qué es:** Un conjunto de técnicas que **penalizan la complejidad del modelo durante el entrenamiento**, para combatir directamente el overfitting. Las más comunes en modelos lineales: **L1 (Lasso)**, que puede llevar coeficientes exactamente a cero (haciendo selección de features implícita), y **L2 (Ridge)**, que reduce todos los coeficientes proporcionalmente sin llevarlos a cero. En redes neuronales: _dropout_ (apagar aleatoriamente neuronas durante el entrenamiento), _early stopping_ (detener el entrenamiento antes de que empiece a sobreajustar).

**Por qué existe:** Es la herramienta directa para mover un modelo desde el lado de "alta varianza/overfitting" del espectro bias-variance hacia el punto óptimo, sin tener que reducir manualmente el número de features o la complejidad del modelo.

---

## Métricas de Clasificación

### Matriz de Confusión (la base de todo lo demás)

Antes de las métricas, hay que entender los cuatro resultados posibles al clasificar (ej. "¿es fraude?"):

|                    | Predicho: Positivo      | Predicho: Negativo      |
| ------------------ | ----------------------- | ----------------------- |
| **Real: Positivo** | Verdadero Positivo (TP) | Falso Negativo (FN)     |
| **Real: Negativo** | Falso Positivo (FP)     | Verdadero Negativo (TN) |

### Precision (Precisión)

```python
Precision = TP / (TP + FP)
```

**Qué mide:** De todo lo que el modelo _predijo_ como positivo, ¿qué porcentaje realmente lo era? **Por qué importa:** cuando el costo de un **falso positivo** es alto (ej. marcar una transacción legítima como fraude y bloquear al cliente).

### Recall (Sensibilidad / Exhaustividad)

```python
Recall = TP / (TP + FN)
```

**Qué mide:** De todos los casos _realmente_ positivos, ¿qué porcentaje detectó el modelo? **Por qué importa:** cuando el costo de un **falso negativo** es alto (ej. no detectar un caso real de fraude, o no detectar una enfermedad).

**La tensión precision-recall:** Casi siempre hay un trade-off — puedes subir el recall (detectar más casos positivos) bajando el umbral de decisión, pero eso típicamente baja la precision (más falsos positivos también). La elección correcta depende del **costo relativo de cada tipo de error en tu problema específico de negocio** — no hay una respuesta universal.

### F1-Score

```python
F1 = 2 × (Precision × Recall) / (Precision + Recall)
```

**Qué es:** La media armónica entre precision y recall — un solo número que penaliza fuertemente si cualquiera de las dos es muy baja (a diferencia de un promedio simple). **Por qué existe:** para cuando necesitas un solo número que resuma el balance entre ambas métricas, útil especialmente con **clases desbalanceadas**, donde el _accuracy_ simple (% de aciertos totales) puede ser muy engañoso (ej. si el 99% de las transacciones no son fraude, un modelo que siempre predice "no fraude" tiene 99% de accuracy y es completamente inútil).

### Curva ROC y AUC

**Curva ROC (Receiver Operating Characteristic):** Grafica la Tasa de Verdaderos Positivos (recall) contra la Tasa de Falsos Positivos, a medida que se varía el umbral de decisión del modelo de 0 a 1. Muestra visualmente el trade-off completo entre sensibilidad y especificidad en todos los umbrales posibles, no solo en uno.

**AUC (Area Under the Curve):** El área bajo esa curva ROC, resumida en un solo número entre 0 y 1. Un AUC de 0.5 significa que el modelo no es mejor que adivinar al azar; un AUC de 1.0 significa clasificación perfecta.

**Por qué existen:** A diferencia de precision/recall/F1 (que dependen de haber fijado _un_ umbral de decisión específico, ej. 0.5), el AUC evalúa **la capacidad del modelo de separar las clases en general**, independientemente de qué umbral termines usando en producción — útil para comparar modelos entre sí antes de decidir el umbral óptimo para el negocio.

---

## Métricas de Regresión (Predicción de Valores Numéricos)

### RMSE (_Root Mean Squared Error_)

$$
RMSE = \sqrt{\frac{1}{n}\sum^n_{i=1}(y_{real}-y_{pred})^2}
$$

```python
import numpy as np

rmse = np.sqrt(np.mean((y_real - y_pred) ** 2))
```

**Qué mide:** El error promedio, en las mismas unidades que la variable original, pero **penalizando fuertemente los errores grandes** (porque eleva al cuadrado antes de promediar). **Cuándo usarla:** cuando los errores grandes son desproporcionadamente más costosos que varios errores pequeños (muy relevante, por ejemplo, en forecasting de demanda de personal — un error grande de sub-dotación un día puede ser mucho más costoso que varios días con error pequeño).

### MAE (_Mean Absolute Error_)

$$
MAE = \frac{1}{n}\sum^n_{i=1}\lvert y_{real} - y_{pred}\lvert
$$

```python
from sklearn.metrics import mean_absolute_error

mae = mean_absolute_error(y_real, y_pred)
```

**Qué mide:** El error promedio absoluto, sin elevar al cuadrado — todos los errores pesan de forma **lineal**, no se penalizan desproporcionadamente los grandes. **Cuándo usarla:** cuando quieres una medida de error más fácil de interpretar directamente ("en promedio nos equivocamos por X unidades") y no quieres que unos pocos outliers dominen la métrica.

### MAPE (_Mean Absolute Percentage Error_)

$$
MAPE = \frac{1}{n}\sum^n_{i=1}\lvert\frac{y_{real} - y_{pred}}{y_{real}}\lvert
$$

```python
from sklearn.metrics import mean_absolute_percentage_error

mape = mean_absolute_percentage_error(y_real, y_pred)
```

**Qué mide:** El error, pero expresado como **porcentaje** del valor real, en vez de en unidades absolutas. **Por qué existe:** las unidades absolutas (RMSE, MAE) no son comparables entre series con escalas muy distintas — un MAE de 50 es excelente si estás prediciendo miles de llamadas diarias, pero pésimo si predices un volumen de 60. El MAPE normaliza esto, siendo muy popular en forecasting de negocio precisamente por esta interpretabilidad ("nos equivocamos en promedio un 8%").

> [!WARNING] **La limitación conocida del MAPE (importante en tu contexto de forecasting)**
> Se rompe cuando el valor real (`y_real`) es cero o cercano a cero (división por un número muy pequeño dispara el error hacia el infinito), y penaliza de forma asimétrica: predecir de más y predecir de menos no se penalizan igual matemáticamente.

### SMAPE (_Symmetric Mean Absolute Percentage Error_)

$$
SMAPE = \frac{100\%}{n}\sum^n_{i=1}\frac{|y_{pred} - y_{real}|}{(|y_{real}| + |y_{pred}|)\div2}
$$

```python
import numpy as np

numerador = np.abs(y_real - y_pred)
denominador = (np.abs(y_real) + np.abs(y_pred)) / 2.0

smape = numerador / denominador
```

**Por qué nació:** Específicamente para corregir las dos limitaciones del MAPE mencionadas arriba — al usar el promedio de ambos valores (real y predicho) en el denominador en vez de solo el real, el SMAPE es más estable cuando hay valores cercanos a cero y trata de forma más simétrica los errores de sobre-predicción y sub-predicción. Es muy usado en competencias y benchmarks de forecasting (ej. la competencia M4).

---

## Cómo Elegir la Métrica Correcta (Resumen Práctico)

La pregunta nunca es "¿cuál métrica es la mejor?" en abstracto — es **"¿cuál métrica refleja mejor el costo real de los errores en mi problema de negocio?"**:

- **Clases muy desbalanceadas** (fraude, churn raro) → Precision/Recall/F1, no accuracy simple.
- **Necesitas comparar modelos sin fijar un umbral todavía** → AUC-ROC.
- **Los errores grandes son desproporcionadamente costosos** (forecasting de dotación de personal) → RMSE.
- **Quieres una cifra de error fácil de comunicar a negocio, robusta a outliers** → MAE.
- **Necesitas comparar el error entre series de escalas muy distintas, o comunicar en términos de "% de error"** → MAPE (con cuidado si hay valores cercanos a cero) o mejor, SMAPE.

---

## Ver también

- [[01-Fundamentos-y-Panorama-General]]
- [[02-Roles-y-Carreras-en-DS-ML]]
- [[03-Arquitectura-Empresarial-de-Datos-y-ML]]
- [[04-Ingenieria-de-Datos]]

## Pendiente para próximas partes

- Álgebra lineal y cálculo aplicados a Deep Learning
- Arquitecturas de redes neuronales (CNNs, RNNs, Transformers)
- NLP y LLMs (embeddings de texto, RAG, agentes)
- MLOps avanzado (CI/CD para ML, experiment tracking, feature stores en detalle)
- Series de tiempo y forecasting avanzado (ARIMA, Prophet, modelos de deep learning para series de tiempo)
