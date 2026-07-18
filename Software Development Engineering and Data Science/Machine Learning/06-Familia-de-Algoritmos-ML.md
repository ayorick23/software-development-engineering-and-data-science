---
tags: [machine-learning, algoritmos, deep-learning, nlp, computer-vision, llm]
aliases: [Familia de Algoritmos ML, Algoritmos de Machine Learning]
---
# Parte 6 — Machine Learning: Toda la Familia de Algoritmos

> Continúa de [[05-Ciencia-de-Datos-Estadistica-y-Fundamentos-ML]]. Aquí el objetivo no es enseñarte matemáticamente cada algoritmo, sino darte el **mapa de cuándo usar cada familia** y por qué existe cada una — la pregunta que realmente te vas a hacer en el trabajo es "tengo este problema, ¿qué familia de algoritmos le queda?", no "¿cómo funciona el gradiente de XGBoost por dentro?".

---

## Tabla de Contenidos

1. [[#Cómo pensar la elección de un algoritmo|Cómo pensar la elección de un algoritmo]]
2. [[#Regresión|Regresión]]
3. [[#Árboles de Decisión|Árboles de Decisión]]
4. [[#Bosques (Random Forest)|Bosques (Random Forest)]]
5. [[#Boosting (Gradient Boosting: XGBoost, LightGBM, CatBoost)|Boosting (Gradient Boosting: XGBoost, LightGBM, CatBoost)]]
6. [[#Clustering|Clustering]]
7. [[#Detección de Anomalías (Anomaly Detection)|Detección de Anomalías (Anomaly Detection)]]
8. [[#Series Temporales (Time Series / Forecasting)|Series Temporales (Time Series / Forecasting)]]
9. [[#Sistemas de Recomendación|Sistemas de Recomendación]]
10. [[#NLP (Natural Language Processing / Procesamiento de Lenguaje Natural)|NLP (Natural Language Processing / Procesamiento de Lenguaje Natural)]]
11. [[#Computer Vision (Visión por Computadora)|Computer Vision (Visión por Computadora)]]
12. [[#LLMs (Large Language Models)|LLMs (Large Language Models)]]
13. [[#RL (Reinforcement Learning / Aprendizaje por Refuerzo)|RL (Reinforcement Learning / Aprendizaje por Refuerzo)]]
14. [[#Tabla Resumen: Problema → Familia de Algoritmos|Tabla Resumen: Problema → Familia de Algoritmos]]
15. [[#Ver también|Ver también]]

---
## Cómo Pensar la Elección de un Algoritmo

Antes de la lista, la pregunta marco que ordena todo lo demás:

1. **¿Qué tipo de problema es?** (predecir un número, una categoría, agrupar sin etiquetas, detectar rarezas, generar contenido...)
2. **¿Qué tipo de dato tengo?** (tabular, texto, imagen, series de tiempo, grafo de interacciones)
3. **¿Cuánta data tengo?** (pocos algoritmos de Deep Learning funcionan bien con cientos de filas; brillan con millones)
4. **¿Necesito interpretabilidad?** (¿un regulador o un gerente necesita entender _por qué_ el modelo decidió algo?)
5. **¿Cuánto presupuesto de cómputo/latencia tengo en producción?**

Con esas cinco respuestas, la familia correcta casi siempre se reduce a 1-2 candidatas razonables.

---

## Regresión

**Qué resuelve:** Predecir un **valor numérico continuo** (precio, demanda, temperatura, ingresos).

**Algoritmos típicos:** Regresión lineal, regresión polinomial, regresión Ridge/Lasso (con [[05-Ciencia-de-Datos-Estadistica-y-Fundamentos-ML#Regularización|regularización]]), regresión logística (a pesar del nombre, es para **clasificación**, no regresión — ver más abajo).

**Cuándo usarla:** Es casi siempre el **punto de partida** para cualquier problema de predicción numérica — rápida de entrenar, altamente interpretable (cada coeficiente te dice el efecto de cada variable), y sorprendentemente competitiva cuando la relación entre variables es razonablemente lineal. Úsala primero como _baseline_ antes de saltar a algo más complejo; si un modelo mucho más sofisticado apenas la supera, puede que no valga la pena la complejidad extra.

![[Pasted image 20260714102213.png]]

**Ejemplo con Regresión Lineal:**

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.linear_model import LinearRegression

# 1. Datos de ejemplo: Horas de estudio vs Calificación obtenida
X = np.array([[1], [2], [3], [4], [5]])  # Variable independiente (Horas)
y = np.array([2, 4, 5, 4, 5])           # Variable dependiente (Calificación)

# 2. Crear y entrenar el modelo
modelo = LinearRegression()
modelo.fit(X, y)

# 3. Hacer una predicción (ej. ¿cuál sería la nota estudiando 6 horas?)
horas_nuevas = np.array([[6]])
prediccion = modelo.predict(horas_nuevas)
print(f"Predicción para 6 horas: {prediccion[0]:.2f}")

# 4. Visualizar los resultados
plt.scatter(X, y, color='blue', label='Datos reales')
plt.plot(X, modelo.predict(X), color='red', label='Línea de regresión')
plt.title('Regresión Lineal Simple')
plt.xlabel('Horas de estudio')
plt.ylabel('Calificación')
plt.legend()
plt.show()
```

---

## Árboles de Decisión (_Decision Tree_)

**Qué resuelve:** Clasificación o regresión mediante una secuencia de **preguntas tipo "si/entonces"** sobre las features (ej. "¿antigüedad > 2 años? → ¿quejas > 3? → churn: sí").

**Por qué existen:** Son la forma más natural de capturar **relaciones no lineales e interacciones entre variables** sin tener que especificarlas a mano (a diferencia de la regresión lineal, que asume que cada variable afecta al resultado de forma independiente y proporcional).

**Cuándo usarlos:** Cuando necesitas máxima **interpretabilidad** (puedes literalmente dibujar el árbol y explicárselo a un gerente de negocio paso a paso) y sospechas que hay interacciones no lineales importantes. Su gran debilidad, sin embargo, es que un solo árbol tiende a sobreajustarse fácilmente ([[05-Ciencia-de-Datos-Estadistica-y-Fundamentos-ML#Overfitting y Underfitting|overfitting]]) — de ahí nacen los siguientes dos puntos.

![[Pasted image 20260714101813.png]]

```python
import pandas as pd
from sklearn.tree import DecisionTreeClassifier
from sklearn.tree import plot_tree
import matplotlib.pyplot as plt

# 1. Datos de ejemplo
datos = {
    'Edad': [25, 45, 35, 50, 23, 37, 40, 28],
    'Ingresos': [30000, 80000, 60000, 90000, 25000, 65000, 75000, 40000],
    'Compra': [0, 1, 1, 1, 0, 0, 1, 0] # 0 = No, 1 = Sí
}
df = pd.DataFrame(datos)

# 2. Separar características (X) y variable objetivo (y)
X = df[['Edad', 'Ingresos']]
y = df['Compra']

# 3. Crear y entrenar el modelo
modelo = DecisionTreeClassifier(max_depth=3, random_state=42)
modelo.fit(X, y)

# 4. Hacer una predicción
# Ejemplo: ¿Compra una persona de 30 años con 70,000 de ingresos?
prediccion = modelo.predict([[30, 70000]])
print(f"¿Comprará? (0=No, 1=Sí): {prediccion[0]}")

# 5. Visualizar el árbol de decisión
plt.figure(figsize=(10, 6))
plot_tree(modelo, feature_names=['Edad', 'Ingresos'], class_names=['No', 'Sí'], filled=True)
plt.show()
```

---

## Bosques (_Random Forest_)

**Qué resuelve:** El mismo tipo de problema que un árbol individual, pero mucho más robusto.

**Por qué nació:** Un solo árbol de decisión es inestable — un pequeño cambio en los datos de entrenamiento puede producir un árbol completamente distinto (alta varianza). Random Forest resuelve esto entrenando **cientos de árboles distintos**, cada uno sobre una muestra aleatoria de los datos y de las features (técnica llamada _bagging_), y promediando sus predicciones. El "ruido" de árboles individuales se cancela al promediar muchos.

**Cuándo usarlo:** Es uno de los algoritmos más usados en la práctica para datos **tabulares** — funciona bien casi "out of the box", con poco ajuste de hiperparámetros, es robusto a outliers, y maneja bien tanto variables numéricas como categóricas. Pierde algo de la interpretabilidad directa de un solo árbol (ya no puedes "dibujarlo"), aunque se compensa con técnicas de importancia de features.

![[Pasted image 20260714102001.png]]

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn import tree
import matplotlib.pyplot as plt
import pandas as pd
from sklearn.metrics import accuracy_score, confusion_matrix, ConfusionMatrixDisplay

# 1. Cargar el dataset (ejemplo clásico: flores Iris)
iris = load_iris()
X = pd.DataFrame(iris.data, columns=iris.feature_names)
y = iris.target

# 2. Dividir los datos en entrenamiento (80%) y prueba (20%)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 3. Crear y entrenar el modelo Random Forest
# Usaremos un bosque con 100 árboles de decisión
modelo = RandomForestClassifier(n_estimators=100, random_state=42)
modelo.fit(X_train, y_train)

# 4. Hacer predicciones y evaluar el modelo
predicciones = modelo.predict(X_test)
precision = accuracy_score(y_test, predicciones)
print(f"Precisión del modelo: {precision * 100:.2f}%\n")

# 5. Visualizar la Matriz de Confusión
cm = confusion_matrix(y_test, predicciones, labels=modelo.classes_)
disp = ConfusionMatrixDisplay(confusion_matrix=cm, display_labels=iris.target_names)
disp.plot(cmap=plt.cm.Blues)
plt.title("Matriz de Confusión del Random Forest")
plt.show()

# 6. Visualizar uno de los árboles que componen el "bosque"
# Extraemos el primer árbol (índice 0)
plt.figure(figsize=(15, 10))
tree.plot_tree(modelo.estimators_[0], 
               feature_names=iris.feature_names, 
               class_names=iris.target_names, 
               filled=True, 
               rounded=True)
plt.title("Visualización de un Árbol de Decisión dentro del Random Forest")
plt.show()
```

---

## Boosting (Gradient Boosting: XGBoost, LightGBM, CatBoost)

**Qué resuelve:** Igual que Random Forest — clasificación/regresión sobre datos tabulares — pero con una filosofía distinta.

**Por qué nació y en qué se diferencia de Random Forest:** Random Forest entrena árboles **en paralelo, de forma independiente** (bagging) y promedia. Boosting entrena árboles **en secuencia**, donde cada árbol nuevo se enfoca específicamente en **corregir los errores que cometieron los árboles anteriores**. Este enfoque secuencial y "orientado al error" suele producir modelos con mejor desempeño predictivo puro que Random Forest, a costa de ser más sensible a overfitting si no se regula bien (por eso estos algoritmos traen muchos hiperparámetros de control).

**Cuándo usarlo:** Es, en la práctica actual (2024-2026), **el algoritmo dominante en competencias de datos tabulares** (Kaggle) y en producción real para forecasting, scoring de crédito, detección de fraude, churn — prácticamente cualquier problema tabular donde el desempeño predictivo importa más que la interpretabilidad directa. Ver [[07-Librerias-de-Data-Science-y-ML]] para las diferencias específicas entre XGBoost, LightGBM y CatBoost.

![[bagging-vs-boosting-datascientest-1.webp]]

```python
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
from sklearn.datasets import load_wine

# 2. Cargar los datos de ejemplo
datos = load_wine()
X, y = datos.data, datos.target

# 3. Dividir los datos en entrenamiento y prueba (80% entrenar, 20% probar)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 4. Crear y entrenar el modelo de Boosting
# n_estimators=100 significa que usaremos 100 árboles secuenciales
modelo = GradientBoostingClassifier(n_estimators=100, learning_rate=0.1, random_state=42)
modelo.fit(X_train, y_train)

# 5. Hacer predicciones y evaluar el modelo
predicciones = modelo.predict(X_test)
precision = accuracy_score(y_test, predicciones)

print(f"Precisión del modelo: {precision * 100:.2f}%")
```

---

## Clustering

**Qué resuelve:** Agrupar datos en **grupos con características similares, sin etiquetas previas** (aprendizaje **no supervisado** — no le dices al algoritmo cuáles son los grupos correctos, él los descubre).

**Algoritmos típicos:** K-Means (el más común — agrupa en _k_ grupos definiendo centroides), DBSCAN (agrupa por densidad, detecta outliers naturalmente, no requiere definir _k_ de antemano), clustering jerárquico (construye un árbol de agrupaciones a distintos niveles de granularidad).

**Cuándo usarlo:** Segmentación de clientes sin categorías predefinidas ("¿qué grupos naturales de comportamiento existen en mi base de clientes?"), detección de patrones exploratorios, o como paso previo de EDA antes de construir un modelo supervisado. La diferencia clave con clasificación: en clustering **no sabes de antemano cuáles son los grupos** — el algoritmo te los revela.

![[Pasted image 20260714103632.png]]

**Ejemplo con KMeans:**

```python
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.datasets import make_blobs

# 1. Crear datos de ejemplo (3 grupos naturales)
# 'X' contiene las coordenadas y 'y' las etiquetas reales (solo para referencia)
X, y = make_blobs(n_samples=300, centers=3, cluster_std=0.60, random_state=42)

# 2. Configurar y entrenar el modelo K-Means
# Le decimos al algoritmo que queremos encontrar exactamente 3 clusters
kmeans = KMeans(n_clusters=3, n_init='auto', random_state=42)
kmeans.fit(X)

# Obtener las etiquetas de los clusters asignadas a cada punto
labels = kmeans.labels_

# Obtener las coordenadas de los centroides (el centro de cada grupo)
centroids = kmeans.cluster_centers_

# 3. Visualizar los resultados
plt.figure(figsize=(8, 6))
# Dibujar los puntos de datos coloreados por su cluster
# X[:, 0] es el eje X, X[:, 1] es el eje Y
plt.scatter(X[:, 0], X[:, 1], c=labels, cmap='viridis', marker='o', s=50, edgecolor='k')
# Dibujar los centroides de los clusters en color rojo
plt.scatter(centroids[:, 0], centroids[:, 1], c='red', marker='x', s=200, label='Centroides')
plt.title('Ejemplo de Clustering con K-Means')
plt.xlabel('Característica 1')
plt.ylabel('Característica 2')
plt.legend()
plt.grid(True)
plt.show()
```

---

## Detección de Anomalías (Anomaly Detection)

**Qué resuelve:** Identificar observaciones que se **desvían significativamente del patrón normal** de los datos — sin necesariamente tener ejemplos etiquetados de "anomalía" (a menudo las anomalías son tan raras que no hay suficientes ejemplos para entrenar un clasificador supervisado tradicional).

**Algoritmos típicos:** Isolation Forest (aísla anomalías midiendo qué tan fácil es "separarlas" del resto con particiones aleatorias), One-Class SVM, Autoencoders (redes neuronales que aprenden a "reconstruir" datos normales; cuando algo no se reconstruye bien, es sospechoso de ser anómalo).

![[3_anomaly-detection-algorithm.avif|597]]

**Cuándo usarlo:** Detección de fraude cuando los casos de fraude etiquetados son extremadamente escasos, detección de fallas en sensores/equipos industriales, detección de comportamiento anómalo en redes/ciberseguridad, control de calidad. La clave conceptual: mientras la clasificación tradicional necesita ejemplos abundantes de ambas clases, la detección de anomalías está diseñada específicamente para el caso donde una clase es rarísima o casi inexistente en los datos etiquetados.

**Ejemplo con Isolation Forest:**

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.ensemble import IsolationForest

# 1. Generamos datos de ejemplo (normales y atípicos)
np.random.seed(42)

# Datos normales (agrupados cerca de 0)
X_normal = 0.3 * np.random.randn(500, 2)

# Añadimos datos anómalos (alejados de 0)
X_outliers = np.random.uniform(low=-4, high=4, size=(20, 2))

# Unimos ambos conjuntos
X = np.r_[X_normal, X_outliers]

# 2. Entrenamos el modelo Isolation Forest
# contamination='auto' o le indicamos el % esperado de anomalías (ej. 0.04)
clf = IsolationForest(contamination=0.04, random_state=42)
clf.fit(X)

# 3. Predecimos las anomalías
# Devuelve 1 para datos normales y -1 para anomalías
predictions = clf.predict(X)

# 4. Visualización de los resultados
plt.figure(figsize=(10, 6))

# Separamos los puntos normales y anómalos para graficarlos con distintos colores
normal_mask = predictions == 1
anomaly_mask = predictions == -1

plt.scatter(X[normal_mask, 0], X[normal_mask, 1], c='blue', label='Datos normales', alpha=0.6)
plt.scatter(X[anomaly_mask, 0], X[anomaly_mask, 1], c='red', edgecolor='black', label='Anomalías', alpha=0.8)

plt.title("Detección de Anomalías con Isolation Forest")
plt.xlabel("Característica 1")
plt.ylabel("Característica 2")
plt.legend()
plt.grid(True)
plt.show()
```

---

## Series Temporales (Time Series / Forecasting)

**Qué resuelve:** Predecir valores futuros de una variable que se observa **secuencialmente en el tiempo**, donde el orden temporal y la dependencia entre observaciones consecutivas son la esencia del problema (muy relevante para tu contexto de forecasting de demanda/dotación en WFM).

**Algoritmos típicos:**

- **Clásicos estadísticos:** ARIMA/SARIMA (modelan la serie en función de sus propios valores pasados y errores pasados), modelos de suavizado exponencial (Holt-Winters).
- **Modelos modernos "de negocio":** Prophet (de Meta, diseñado para ser robusto a estacionalidad, feriados y datos faltantes con mínima configuración manual).
- **Enfoque de ML supervisado:** convertir el problema en tabular (usando valores pasados como features — _lags_) y aplicar Random Forest/Boosting.
- **Deep Learning:** RNNs/LSTMs, y más recientemente arquitecturas basadas en Transformers adaptadas a series de tiempo, útiles cuando hay múltiples series relacionadas y volumen de datos muy grande.

![[forecasting_multi-step.gif]]

**Cuándo usar cuál:** ARIMA/Holt-Winters para series individuales, con patrones claros y relativamente simples, donde la interpretabilidad estadística importa. Prophet cuando hay fuerte estacionalidad/feriados y quieres algo robusto con poco tuning manual. ML supervisado (boosting con lags) cuando tienes múltiples variables externas (features exógenas) que influyen en la serie, no solo el pasado de la propia serie — el caso típico en forecasting de WFM (feriados, campañas, clima, etc. además del histórico). Deep Learning cuando tienes cientos/miles de series relacionadas y datos suficientes para justificar la complejidad.

**Ejemplo con ARIMA:**

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from statsmodels.tsa.arima.model import ARIMA

# 1. Crear datos de ejemplo (una serie temporal con tendencia ascendente)
np.random.seed(42)
fechas = pd.date_range(start='2025-01-01', periods=100, freq='D')
valores = np.linspace(10, 50, 100) + np.random.normal(0, 2, 100)

datos = pd.Series(valores, index=fechas)

# 2. Definir y entrenar el modelo ARIMA
# Nota: Los parámetros (p, d, q) deben ajustarse a tus datos. 
# Usaremos (1, 1, 1) como ejemplo introductorio.
p, d, q = 1, 1, 1
modelo = ARIMA(datos, order=(p, d, q))
resultado_modelo = modelo.fit()

# 3. Mostrar el resumen estadístico del modelo
print(resultado_modelo.summary())

# 4. Hacer predicciones para los próximos 10 días
predicciones = resultado_modelo.forecast(steps=10)
print("\nPredicciones para los próximos 10 días:")
print(predicciones)

# 5. Visualizar los datos históricos y las predicciones
plt.figure(figsize=(10, 6))
plt.plot(datos.index, datos.values, label='Datos Históricos')
plt.plot(pd.date_range(start='2025-04-11', periods=10, freq='D'), predicciones, label='Predicciones', color='red')
plt.title('Pronóstico con Modelo ARIMA')
plt.legend()
plt.show()
```

---

## Sistemas de Recomendación

**Qué resuelve:** Predecir qué ítems (productos, contenido, personas) le interesarían a un usuario específico, a partir de su comportamiento pasado y/o del comportamiento de usuarios similares.

**Enfoques principales:**

- **Filtrado colaborativo (Collaborative Filtering):** se basa en el patrón "usuarios parecidos a ti también les gustó X" — usa [[01-Fundamentos-y-Panorama-General#¿Qué significa embedding?|embeddings]] de usuarios e ítems aprendidos de las interacciones (compras, clics, calificaciones).
- **Filtrado basado en contenido (Content-Based):** recomienda ítems parecidos, en sus atributos, a los que al usuario ya le gustaron (ej. "te gustó esta película de acción, aquí hay otra de acción").
- **Enfoques híbridos:** combinan ambos, y son el estándar en sistemas de recomendación de producción modernos (Netflix, Spotify, Amazon).

**Cuándo usarlo:** Cuando el problema central es "de todos los ítems posibles, ¿cuáles mostrarle a este usuario específico?" — e-commerce, streaming, contenido personalizado. El reto principal en producción no es solo la precisión, sino el **problema del arranque en frío** (_cold start_: ¿qué recomiendas a un usuario o ítem nuevo del que no tienes historial?).

```python
import pandas as pd
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.metrics.pairwise import cosine_similarity

# 1. Crear el dataset de ejemplo
datos = {
    'pelicula': ['Toy Story', 'Buscando a Nemo', 'El Padrino', 'Batman', 'El Conjuro'],
    'genero': ['Animacion', 'Animacion', 'Crimen', 'Accion', 'Terror'],
    'director': ['John Lasseter', 'Andrew Stanton', 'Francis Coppola', 'Christopher Nolan', 'James Wan']
}
df = pd.DataFrame(datos)

# 2. Combinar características para crear una "firma" de cada película
df['caracteristicas'] = df['genero'] + ' ' + df['director']

# 3. Convertir las características en vectores numéricos
vectorizer = CountVectorizer()
matriz_caracteristicas = vectorizer.fit_transform(df['caracteristicas'])

# 4. Calcular la similitud del coseno entre todas las películas
similitud = cosine_similarity(matriz_caracteristicas)

# 5. Función para recomendar
def recomendar_pelicula(titulo_pelicula, df, matriz_similitud):
    # Obtener el índice de la película
    indice = df[df['pelicula'] == titulo_pelicula].index[0]
    
    # Obtener las puntuaciones de similitud para esa película
    puntuaciones = list(enumerate(matriz_similitud[indice]))
    
    # Ordenar las películas por similitud (de mayor a menor)
    puntuaciones = sorted(puntuaciones, key=lambda x: x[1], reverse=True)
    
    # Excluir la primera (que es la misma película ingresada)
    indices_recomendados = [i[0] for i in puntuaciones[1:3]]
    
    # Retornar las películas recomendadas
    return df['pelicula'].iloc[indices_recomendados]

# 6. Probar el sistema
print("Recomendaciones para 'Toy Story':")
print(recomendar_pelicula('Toy Story', df, similitud))
```

---

## NLP (Natural Language Processing / Procesamiento de Lenguaje Natural)

**Qué resuelve:** Cualquier tarea sobre **texto**: clasificación de texto (spam, sentimiento), extracción de entidades, traducción, resumen, respuesta a preguntas, generación de texto.

**Evolución del enfoque (importante entender el "por qué" histórico):**

1. **Enfoques clásicos (pre-2018):** bag-of-words, TF-IDF (convertir texto en vectores contando/ponderando palabras) + algoritmos de ML clásico (regresión logística, SVM) — funcional pero ciego al orden y contexto de las palabras.
2. **Embeddings de palabras (Word2Vec, GloVe):** representar cada palabra como un vector que captura significado (ver [[01-Fundamentos-y-Panorama-General#¿Qué significa embedding?]]), pero cada palabra tiene un solo vector fijo, sin importar el contexto ("banco" de sentarse vs. "banco" financiero tenían el mismo vector).
3. **Transformers y modelos contextuales (BERT, GPT, 2018 en adelante):** cada palabra obtiene una representación que **depende del contexto de la oración completa** (mecanismo de _atención_), lo que resolvió la ambigüedad del punto anterior y disparó la ola actual de LLMs (ver [[10-IA-Generativa-LLMs-y-Agentes]]).

**Cuándo usar cada enfoque hoy:** Para tareas simples de clasificación de texto con datasets pequeños, TF-IDF + ML clásico sigue siendo válido, rápido y barato. Para casi cualquier tarea moderna con volumen de texto razonable, los modelos basados en Transformers (fine-tuneados o vía LLMs con prompt engineering/RAG) son el estándar actual.

---

## Computer Vision (Visión por Computadora)

**Qué resuelve:** Tareas sobre **imágenes o video**: clasificación de imágenes, detección de objetos (dónde está y qué es cada objeto en la imagen), segmentación (delinear el contorno exacto de cada objeto), reconocimiento facial, OCR.

**Arquitectura dominante:** **CNNs (Convolutional Neural Networks / Redes Neuronales Convolucionales)** — diseñadas específicamente para explotar la estructura espacial de las imágenes (patrones locales como bordes y texturas se combinan progresivamente en patrones más complejos capa a capa). Modelos más recientes también usan Vision Transformers (ViT), aplicando la arquitectura Transformer (originalmente de NLP) a imágenes.

**Por qué CNNs y no una red neuronal "normal" (fully connected):** Una imagen de tamaño moderado tiene cientos de miles de píxeles; conectar cada píxel a cada neurona de la siguiente capa (como en una red densa tradicional) sería computacionalmente inviable y, más importante, ignoraría que la información relevante en una imagen es **local y espacial** (un borde, una textura) — las CNNs comparten parámetros a través de la imagen (filtros convolucionales) para capturar esto de forma eficiente.

**Cuándo usarlo:** Control de calidad visual en manufactura, diagnóstico médico por imagen, conteo/detección de objetos, OCR de documentos, reconocimiento facial/biométrico.

---

## LLMs (Large Language Models)

**Qué resuelve:** Generación y comprensión de texto de propósito general, a un nivel de fluidez y flexibilidad que antes requería modelos entrenados específicamente para cada tarea (traducción, resumen, respuesta a preguntas, generación de código, etc., todo con un mismo modelo).

**Por qué merecen su propia categoría, distinta de "NLP clásico":** Su tamaño (miles de millones a billones de parámetros) y la escala de datos con la que se entrenan les dan capacidades **emergentes** — pueden resolver tareas para las que nunca fueron explícitamente entrenados, solo a partir de instrucciones en lenguaje natural (_prompting_), sin necesitar reentrenamiento específico por tarea. Ver el detalle completo en [[10-IA-Generativa-LLMs-y-Agentes]].

**Cuándo usarlo:** Cuando la tarea es de lenguaje general y flexible, y no vale la pena (en tiempo/costo) entrenar un modelo especializado desde cero. Ojo: para tareas de clasificación simple y bien definida con mucho volumen (millones de inferencias diarias), un modelo clásico de ML sigue siendo mucho más barato y rápido en inferencia que llamar a un LLM — no todo problema de texto necesita un LLM.

---

## RL (Reinforcement Learning / Aprendizaje por Refuerzo)

**Qué resuelve:** Problemas donde un **agente aprende a tomar una secuencia de decisiones** interactuando con un entorno, recibiendo una **recompensa** (positiva o negativa) según el resultado de sus acciones, con el objetivo de maximizar la recompensa acumulada a largo plazo — no aprende de un dataset fijo de "respuestas correctas" como el aprendizaje supervisado, sino por prueba y error.

**Por qué es fundamentalmente distinto a todo lo anterior:** Los algoritmos anteriores (regresión, árboles, clustering...) aprenden de datos **estáticos y ya recolectados**. RL aprende de la **interacción activa** con un entorno (real o simulado), y su "dataset" se genera sobre la marcha a medida que el agente actúa y observa las consecuencias.

**Cuándo usarlo:** Sistemas de control robótico, optimización de decisiones secuenciales (ej. optimización de precios dinámicos, ruteo de flotas), juegos, y de forma muy relevante hoy: **RLHF (Reinforcement Learning from Human Feedback)**, la técnica usada para alinear el comportamiento de LLMs modernos con las preferencias humanas después del preentrenamiento inicial. Es, con diferencia, la familia más compleja de implementar y estabilizar en producción de toda esta lista — se usa solo cuando el problema es genuinamente secuencial/interactivo, no como primera opción.

---

## Tabla Resumen: Problema → Familia de Algoritmos

|Tengo este problema...|Empieza por...|
|---|---|
|Predecir un número, datos tabulares|Regresión → Boosting (XGBoost/LightGBM)|
|Clasificar en categorías, datos tabulares|Regresión logística → Boosting|
|Necesito que un gerente entienda el "por qué"|Árbol de decisión simple, regresión lineal|
|Agrupar clientes/ítems sin etiquetas|Clustering (K-Means, DBSCAN)|
|Detectar fraude/fallas raras, pocas etiquetas|Anomaly Detection (Isolation Forest)|
|Predecir demanda/ventas futuras|ARIMA/Prophet → Boosting con lags → DL si hay volumen|
|Recomendar productos/contenido|Filtrado colaborativo/híbrido|
|Clasificar o analizar texto|TF-IDF+ML clásico (simple) → Transformers/LLM (complejo)|
|Analizar imágenes/video|CNN / Vision Transformer|
|Tarea de lenguaje general y flexible|LLM (con prompt engineering o RAG)|
|Decisiones secuenciales con recompensa|Reinforcement Learning|

---

## Ver también

- [[05-Ciencia-de-Datos-Estadistica-y-Fundamentos-ML]]
- [[07-Librerias-de-Data-Science-y-ML]]
- [[10-IA-Generativa-LLMs-y-Agentes]]
