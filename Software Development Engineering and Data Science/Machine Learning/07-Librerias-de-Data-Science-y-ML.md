---
tags: [librerias, python, pandas, numpy, pytorch, tensorflow, scikit-learn, mlops]
aliases: [Librerias de Python para Data Science, Librerias ML]
---
# Parte 7 — Librerías: Qué Hace Cada Una (y Hasta Dónde Llega)

> Continúa de [[06-Familia-de-Algoritmos-ML]]. El objetivo explícito de esta nota es evitar el error más común al aprender el ecosistema: **asumir que una librería hace más de lo que realmente hace**. Cada sección termina con un límite explícito — la frontera exacta donde esa librería termina y necesitas otra cosa.

---

## Tabla de Contenidos

1. [[#Manipulación de Datos|Manipulación de Datos]]
	1. [[#Manipulación de Datos#Pandas|Pandas]]
	2. [[#Manipulación de Datos#NumPy|NumPy]]
	3. [[#Manipulación de Datos#SciPy|SciPy]]
	4. [[#Manipulación de Datos#SimPy|SimPy]]
	5. [[#Manipulación de Datos#StatsModels|StatsModels]]
2. [[#Machine Learning Clásico|Machine Learning Clásico]]
	1. [[#Machine Learning Clásico#Scikit-learn|Scikit-learn]]
3. [[#Visualización de Datos|Visualización de Datos]]
	1. [[#Visualización de Datos#Matplotlib|Matplotlib]]
	2. [[#Visualización de Datos#Seaborn|Seaborn]]
	3. [[#Visualización de Datos#Plotly|Plotly]]
	4. [[#Visualización de Datos#Dash|Dash]]
	5. [[#Visualización de Datos#Altair|Altair]]
	6. [[#Visualización de Datos#Bokeh|Bokeh]]
	7. [[#Visualización de Datos#HoloViews|HoloViews]]
4. [[#Deep Learning|Deep Learning]]
	1. [[#Deep Learning#TensorFlow|TensorFlow]]
	2. [[#Deep Learning#PyTorch|PyTorch]]
5. [[#Boosting (Gradient Boosting Especializado)|Boosting (Gradient Boosting Especializado)]]
	1. [[#Boosting (Gradient Boosting Especializado)#XGBoost|XGBoost]]
	2. [[#Boosting (Gradient Boosting Especializado)#LightGBM|LightGBM]]
	3. [[#Boosting (Gradient Boosting Especializado)#CatBoost|CatBoost]]
6. [[#NLP e IA Generativa (Base)|NLP e IA Generativa (Base)]]
	1. [[#NLP e IA Generativa (Base)#Hugging Face|Hugging Face]]
	2. [[#NLP e IA Generativa (Base)#LangChain|LangChain]]
7. [[#MLOps y Ciclo de Vida del Modelo|MLOps y Ciclo de Vida del Modelo]]
	1. [[#MLOps y Ciclo de Vida del Modelo#MLflow|MLflow]]
	2. [[#MLOps y Ciclo de Vida del Modelo#Feast|Feast]]
	3. [[#MLOps y Ciclo de Vida del Modelo#Evidently|Evidently]]
	4. [[#MLOps y Ciclo de Vida del Modelo#Great Expectations|Great Expectations]]
	5. [[#MLOps y Ciclo de Vida del Modelo#ONNX (Open Neural Network Exchange)|ONNX (Open Neural Network Exchange)]]
8. [[#Cómputo Distribuido / Paralelo|Cómputo Distribuido / Paralelo]]
	1. [[#Cómputo Distribuido / Paralelo#Ray|Ray]]
	2. [[#Cómputo Distribuido / Paralelo#Dask|Dask]]
9. [[#Optimización y Experimentación|Optimización y Experimentación]]
	1. [[#Optimización y Experimentación#Optuna|Optuna]]
	2. [[#Optimización y Experimentación#Weights & Biases (W&B)|Weights & Biases (W&B)]]
10. [[#Tabla Resumen: "esto NO es lo que crees"|Tabla Resumen: "esto NO es lo que crees"]]
11. [[#Ver también|Ver también]]

---
## Manipulación de Datos

### Pandas

![[Pasted image 20260714142530.png|319]]

**Qué hace:** Manipulación y análisis de datos **tabulares** en memoria (dataframes) — filtrar, agrupar, unir, transformar, limpiar. Es la herramienta de trabajo diario más usada en ciencia de datos con Python.

**Hasta dónde llega:** Pandas **no entrena modelos** (aunque a veces se usa junto a scikit-learn tan seguido que la gente los confunde), y **no está diseñado para datasets que no caben en la memoria RAM de una sola máquina** (para eso: Spark, [[#Dask]], o [[04-Ingenieria-de-Datos#DuckDB|DuckDB]]).

**Ver también:** cheat-sheet práctico completo en [[Python/Pandas/01 - Introducción y Arquitectura Interna|Python/Pandas]] (arquitectura interna, selección, groupby, series de tiempo, rendimiento e integración con el resto del stack).

### NumPy

![[Pasted image 20260714142214.png|317]]

**Qué hace:** La librería base de **cómputo numérico** en Python — arrays multidimensionales eficientes y operaciones matemáticas/álgebra lineal vectorizadas (mucho más rápidas que loops nativos de Python). Es, literalmente, el cimiento sobre el que está construido Pandas, scikit-learn, y en buena medida TensorFlow/PyTorch.

**Ver también:** cheat-sheet práctico completo en [[Python/NumPy/01 - Introducción y Arquitectura Interna|Python/NumPy]] (arquitectura interna, broadcasting, álgebra lineal, rendimiento e integración con el resto del stack).

**Hasta dónde llega:** NumPy no sabe nada de "dataframes con nombres de columnas" (eso es Pandas encima de NumPy), no entrena modelos, y no maneja datos que no caben en memoria de una sola máquina.

### SciPy

![[Pasted image 20260714142305.png|315]]

**Qué hace:** Extiende NumPy con algoritmos científicos más avanzados: optimización matemática, estadística (distribuciones de probabilidad, tests estadísticos), procesamiento de señales, álgebra lineal avanzada, interpolación.

**Hasta dónde llega:** Es una **caja de herramientas matemáticas de propósito general**, no una librería de Machine Learning — no tiene la noción de "entrenar un modelo con datos etiquetados" como scikit-learn. Muchas librerías de ML (incluido scikit-learn) usan SciPy internamente.

### SimPy

![[simpy-logo-small.webp|302]]

**Qué hace:** Una librería de **simulación de eventos discretos** — modelar sistemas donde ocurren eventos en el tiempo (ej. clientes llegando a una fila, llamadas entrando a un call center, piezas moviéndose por una línea de producción) para estudiar su comportamiento sin tener que observarlo en la vida real.

**Hasta dónde llega:** SimPy **no es una librería de Machine Learning ni de análisis estadístico de datos históricos** — es para _simular_ sistemas hipotéticos, útil para responder preguntas tipo "¿qué pasaría si aumentamos la dotación de agentes en un 20%?" (muy relevante para _modelado de capacidad_ en contextos de WFM), no para predecir a partir de datos reales ya observados.

### StatsModels

![[statsmodels-logo-v2-horizontal.svg|413]]

**Qué hace:** Modelado estadístico riguroso en Python — regresión lineal/logística con salida estadística completa (p-valores, intervalos de confianza, diagnósticos), modelos de series de tiempo (ARIMA), tests de hipótesis.

**Hasta dónde llega:** A diferencia de scikit-learn (que prioriza el desempeño predictivo), StatsModels prioriza la **inferencia estadística** — te da mucho más detalle sobre significancia estadística y supuestos del modelo, pero tiene muchos menos algoritmos de ML moderno (no vas a encontrar Random Forest o Gradient Boosting aquí) y no está optimizado para producción a gran escala.

---

## Machine Learning Clásico

### Scikit-learn

![[Scikit_learn_logo_small.svg.webp|352]]

**Qué hace:** La librería estándar de **Machine Learning clásico** en Python — implementa la inmensa mayoría de los algoritmos de [[06-Familia-de-Algoritmos-ML]] que no son Deep Learning (regresión, árboles, Random Forest, clustering, SVM, reducción de dimensionalidad), junto con herramientas de preprocesamiento, [[05-Ciencia-de-Datos-Estadistica-y-Fundamentos-ML#Validación y Cross-Validation|validación cruzada]] y pipelines de transformación, todo con una API consistente y muy fácil de usar.

**Hasta dónde llega (el ejemplo que mencionaste):** **Scikit-learn NO está diseñado para Deep Learning.** No tiene soporte nativo para redes neuronales profundas, no usa GPU de forma nativa, y no maneja el tipo de arquitecturas (CNNs, Transformers) que requieren Deep Learning moderno. Tiene un módulo básico de redes neuronales (`MLPClassifier`/`MLPRegressor`) pensado solo para redes pequeñas y simples — no es lo que usarías para visión por computadora o NLP moderno. Para eso: TensorFlow o PyTorch.

---

## Visualización de Datos

### Matplotlib

![[Pasted image 20260715082641.png|381]]

**Qué hace:** Biblioteca base para crear **gráficos estáticos** en Python. Permite construir desde gráficos simples hasta visualizaciones altamente personalizadas. Es el fundamento sobre el que se apoyan muchas otras librerías de visualización.

**Hasta dónde llega:** Matplotlib **no está orientado a crear dashboards interactivos** ni ofrece una sintaxis especialmente amigable para análisis exploratorio rápido. Para visualizaciones estadísticas de alto nivel suele utilizarse [[#Seaborn]], y para gráficos interactivos [[#Plotly]].

**Ver también:** cheat-sheet práctico completo en [[Python/Matplotlib/01 - Introducción y Arquitectura|Python/Matplotlib]] (arquitectura Figure/Axes, tipos de gráficos, personalización, animaciones y rendimiento).

### Seaborn

![[logo-wide-lightbg.png.webp|375]]

**Qué hace:** Biblioteca de visualización estadística construida sobre **Matplotlib**. Simplifica la creación de gráficos elegantes para análisis exploratorio (EDA), distribuciones, correlaciones, relaciones entre variables y comparación de grupos.

**Hasta dónde llega:** Seaborn **no reemplaza completamente a Matplotlib**, ya que internamente depende de él. Cuando se requiere una personalización muy específica normalmente se modifica el gráfico utilizando funciones de Matplotlib.

**Ver también:** cheat-sheet práctico completo en [[Python/Seaborn/01 - Introducción y Arquitectura|Python/Seaborn]] (figure-level vs axes-level, gráficos de relación/distribución/categóricos, FacetGrid, estadística integrada e integración con Pandas/Matplotlib).

### Plotly

![[Pasted image 20260715082743.png|398]]

**Qué hace:** Biblioteca para crear **gráficos interactivos** con zoom, selección, filtros y herramientas de exploración. Es ampliamente utilizada para dashboards, aplicaciones web y análisis exploratorio donde el usuario necesita interactuar con la información.

**Hasta dónde llega:** Plotly **no está pensado para análisis estadístico**, sino para la visualización de resultados. Para construir aplicaciones completas suele combinarse con [[#Dash]] o Streamlit (cheat-sheet técnico completo en `Streamlit/01 - Introducción y Modelo de Ejecución.md`).

### Dash

![[Pasted image 20260715083424.png|340]]

**Qué hace:** Framework desarrollado por Plotly para construir **dashboards y aplicaciones web analíticas** utilizando únicamente Python. Permite conectar gráficos, filtros, tablas y componentes interactivos sin necesidad de escribir JavaScript.

**Hasta dónde llega:** Dash **no reemplaza frameworks web generales** como Django o Flask. Está especializado en aplicaciones de análisis de datos y visualización interactiva.

### Altair

![[Pasted image 20260715083947.png|120]]

**Qué hace:** Biblioteca declarativa de visualización basada en la gramática de gráficos (*Grammar of Graphics*). Permite crear gráficos complejos escribiendo muy poco código y favorece la reproducibilidad de las visualizaciones.

**Hasta dónde llega:** Altair **no ofrece el mismo nivel de personalización que Matplotlib** y puede presentar limitaciones con datasets muy grandes, aunque es excelente para análisis exploratorio y visualizaciones limpias.

### Bokeh

![[Pasted image 20260715084101.png|366]]

**Qué hace:** Biblioteca para crear **visualizaciones interactivas** orientadas a aplicaciones web y grandes volúmenes de datos. Permite construir gráficos dinámicos que pueden integrarse fácilmente en páginas web.

**Hasta dónde llega:** Aunque puede utilizarse para dashboards, actualmente muchas organizaciones prefieren Plotly + Dash o Streamlit por su ecosistema y facilidad de uso.

### HoloViews

![[Pasted image 20260715084315.png|381]]

**Qué hace:** Biblioteca de alto nivel que simplifica la creación de visualizaciones complejas a partir de estructuras de datos, permitiendo centrarse en el análisis en lugar del código de representación gráfica.

**Hasta dónde llega:** HoloViews **no genera gráficos por sí sola**; utiliza motores como Matplotlib, Bokeh o Plotly para renderizar las visualizaciones.

---

## Deep Learning

### TensorFlow

![[Pasted image 20260714142943.png|322]]

**Qué hace:** Framework de Deep Learning creado por Google — permite definir, entrenar y desplegar redes neuronales de cualquier arquitectura, con soporte nativo de GPU/TPU para entrenamiento distribuido a gran escala. Incluye Keras como su API de alto nivel (más simple de usar).

**Hasta dónde llega (el ejemplo que mencionaste):** **TensorFlow NO reemplaza a Pandas.** TensorFlow trabaja con tensores optimizados para el entrenamiento de redes neuronales, no está diseñado para la exploración, limpieza y manipulación flexible de datos tabulares del día a día — para eso, sigues usando Pandas (u otras herramientas de la sección anterior) y luego alimentas los datos ya procesados a TensorFlow.

### PyTorch

![[Pasted image 20260714143037.png|342]]

**Qué hace:** Framework de Deep Learning creado por Meta (Facebook AI Research) — funcionalmente similar a TensorFlow (define, entrena y despliega redes neuronales con soporte GPU), pero con una filosofía de diseño distinta: más "pythónico", con ejecución dinámica de grafos (_eager execution_ desde el inicio) que lo hace más intuitivo para depurar y experimentar. Se ha convertido en el estándar dominante en investigación académica y en la mayoría de los labs de IA generativa modernos.

**Hasta dónde llega (el ejemplo que mencionaste):** **PyTorch NO es "una evolución de scikit-learn".** Son librerías con propósitos y paradigmas distintos desde su diseño — scikit-learn es para algoritmos clásicos con una API de "fit/predict" simple sobre datos tabulares; PyTorch es para construir arquitecturas de redes neuronales, capa por capa, con control total sobre el proceso de entrenamiento (funciones de pérdida, optimizadores, arquitecturas personalizadas). No hay una relación de "sucesor" entre ambas — de hecho, siguen usándose juntas constantemente en el mismo proyecto (scikit-learn para preprocesamiento/baseline, PyTorch para el modelo de Deep Learning final).

---

## Boosting (Gradient Boosting Especializado)

### XGBoost

![[Pasted image 20260714143216.png|340]]

**Qué hace:** Una implementación de [[06-Familia-de-Algoritmos-ML#Boosting (Gradient Boosting: XGBoost, LightGBM, CatBoost)|Gradient Boosting]] extremadamente optimizada en velocidad y regularización, que se convirtió en el estándar de facto en competencias de datos tabulares desde mediados de la década de 2010.

**Hasta dónde llega:** Es una librería especializada exclusivamente en modelos basados en árboles con boosting — no reemplaza a scikit-learn como herramienta general (de hecho se integra con su API), y no sirve para datos no estructurados (imágenes, texto crudo, audio).

### LightGBM

![[LightGBM_logo_black_text.svg|333]]

**Qué hace:** Otra implementación de Gradient Boosting, creada por Microsoft, diseñada específicamente para ser **más rápida y usar menos memoria que XGBoost** en datasets muy grandes, mediante técnicas de muestreo inteligente de datos y features durante el entrenamiento.

**Cuándo elegirla sobre XGBoost:** Cuando el dataset es muy grande y la velocidad de entrenamiento/memoria es una restricción real. En desempeño predictivo puro, ambas suelen quedar muy cerca; la diferencia práctica está más en velocidad y en cómo manejan variables categóricas.

### CatBoost

![[Pasted image 20260714163633.png|347]]

**Qué hace:** Una tercera implementación de Gradient Boosting, creada por Yandex, con una ventaja distintiva: **maneja variables categóricas de forma nativa y automática**, sin que tengas que hacer _one-hot encoding_ o _target encoding_ manual de antemano — reduce mucho el trabajo de feature engineering para datasets con muchas columnas categóricas.

**Cuándo elegirla:** Cuando tu dataset tiene muchas variables categóricas (común en datos de negocio: región, categoría de producto, tipo de cliente) y quieres minimizar el preprocesamiento manual.

---

## NLP e IA Generativa (Base)

### Hugging Face

![[Hf-logo-with-title.svg|347]]

**Qué hace:** No es una sola librería, sino un **ecosistema**: la librería `transformers` (acceso fácil a miles de modelos preentrenados de NLP/visión/audio listos para usar o fine-tunear), el `Hub` (repositorio central de modelos y datasets compartidos por la comunidad), y librerías complementarias (`datasets`, `tokenizers`, `accelerate`) para todo el ciclo de trabajo con modelos de Deep Learning modernos.

**Hasta dónde llega:** Hugging Face **no entrena modelos desde cero por ti automáticamente** ni reemplaza a PyTorch/TensorFlow — de hecho, `transformers` está construido _encima_ de esos frameworks; lo que aporta es acceso simplificado a modelos ya preentrenados y herramientas para fine-tunearlos, no un motor de entrenamiento propio independiente.

### LangChain

![[Pasted image 20260714163653.png|363]]

**Qué hace:** Un framework para construir **aplicaciones** sobre LLMs — encadenar llamadas a modelos, conectar LLMs con fuentes de datos externas (documentos, bases de datos, APIs) para RAG, gestionar memoria de conversación, orquestar agentes. Ver más en [[10-IA-Generativa-LLMs-y-Agentes]].

**Hasta dónde llega:** LangChain **no entrena ni aloja modelos** — es una capa de orquestación que llama a modelos ya existentes (vía API de OpenAI, Anthropic, modelos locales, etc.). No es una alternativa a Hugging Face o PyTorch, sino una capa que vive por encima de ellos, enfocada en construir aplicaciones y flujos, no en el modelo en sí.

---

## MLOps y Ciclo de Vida del Modelo

### MLflow

![[Pasted image 20260714163818.png|335]]

**Qué hace:** Plataforma open-source para gestionar el ciclo de vida de modelos de ML: **experiment tracking** (registrar qué parámetros/métricas tuvo cada corrida de entrenamiento), **Model Registry** (versionar y catalogar modelos — ver [[03-Arquitectura-Empresarial-de-Datos-y-ML#Registro del Modelo (_Model Registry_)]]), y empaquetado/despliegue estandarizado de modelos.

**Hasta dónde llega:** MLflow no entrena modelos por sí mismo (es agnóstico del framework — funciona con scikit-learn, XGBoost, PyTorch, TensorFlow, etc.) ni orquesta pipelines complejos de datos (para eso, Airflow/Kubeflow) — su rol es específicamente el _tracking_ y _registro_, no la ejecución/orquestación del pipeline completo.

### Feast

![[Pasted image 20260714164043.png|324]]

**Qué hace:** Un Feature Store open-source (ver [[03-Arquitectura-Empresarial-de-Datos-y-ML#Feature Store]]) — almacena definiciones de features y las sirve de forma consistente tanto para entrenamiento (en lote) como para inferencia en producción (en tiempo real), evitando el problema de _training-serving skew_.

**Hasta dónde llega:** Feast no calcula features automáticamente a partir de datos crudos por ti (tú defines la lógica de transformación) — orquesta el _almacenamiento y servicio_ de features ya calculadas, no el feature engineering en sí.

### Evidently

![[Pasted image 20260714164319.png|345]]

**Qué hace:** Una librería especializada en **monitoreo de modelos en producción** — detecta _data drift_ (¿la distribución de los datos de entrada cambió?), _concept drift_, y calidad de datos, generando reportes y dashboards comparando datos de referencia (entrenamiento) contra datos actuales de producción.

**Hasta dónde llega:** Evidently monitorea y reporta — no reentrena el modelo automáticamente ni corrige el drift por sí sola; típicamente dispara alertas que un humano (o un pipeline automatizado, ver [[09-MLOps-en-Profundidad]]) usa para decidir si es momento de reentrenar.

### Great Expectations

![[gx-logo.svg|323]]

**Qué hace:** Una librería de **validación de calidad de datos** — defines "expectativas" explícitas sobre tus datos (ej. "esta columna nunca debe tener nulos", "esta columna debe estar entre 0 y 100") y la librería las valida automáticamente en cada corrida de tu pipeline, fallando o alertando si algo no cumple.

**Hasta dónde llega:** Great Expectations valida la **calidad de los datos que entran al pipeline**, no la calidad de las predicciones del modelo (eso es Evidently) — son complementarias: una vigila los datos de entrada, la otra vigila el comportamiento del modelo.

### ONNX (Open Neural Network Exchange)

![[Pasted image 20260714164452.png|329]]

**Qué hace:** Un **formato estándar abierto** para representar modelos de Deep Learning, de forma que un modelo entrenado en un framework (ej. PyTorch) pueda exportarse y ejecutarse en otro entorno de inferencia distinto (ej. un motor de inferencia en C++ optimizado, o un dispositivo móvil), sin depender del framework original.

**Hasta dónde llega:** ONNX no entrena modelos — es exclusivamente un formato de **intercambio e inferencia optimizada**, el puente entre "donde entrenaste el modelo" y "donde lo vas a correr en producción", especialmente útil cuando esos dos entornos usan tecnologías distintas.

---

## Cómputo Distribuido / Paralelo

### Ray

![[Pasted image 20260715080334.png|326]]

**Qué hace:** Un framework para **paralelizar y distribuir código Python** de forma general — desde entrenamiento distribuido de modelos de Deep Learning hasta ajuste de hiperparámetros distribuido (Ray Tune) y servir modelos en producción (Ray Serve), todo escalando de una laptop a un cluster con cambios mínimos de código.

**Hasta dónde llega:** Ray es más general que Spark (no está limitado a procesamiento de datos tipo dataframe) — pero si tu problema es específicamente ETL/transformación masiva de datos tabulares, Spark sigue siendo más maduro para eso; Ray brilla más para paralelizar cargas de trabajo de ML/Python arbitrarias (entrenamiento, tuning, simulación).

### Dask

![[Pasted image 20260715081529.png|346]]

**Qué hace:** Paraleliza y escala código de Python que usa Pandas/NumPy/scikit-learn a datasets que no caben en la memoria de una sola máquina, con una API deliberadamente muy parecida a Pandas/NumPy (curva de aprendizaje baja si ya sabes esas librerías).

**Hasta dónde llega:** Dask es más ligero y "pythónico" que Spark, pero el ecosistema y la madurez para procesamiento masivo distribuido (miles de nodos) siguen favoreciendo a Spark en instalaciones muy grandes de nivel empresarial — Dask es el punto intermedio entre "Pandas en una laptop" y "Spark en un cluster gigante".

---

## Optimización y Experimentación

### Optuna

![[Pasted image 20260715081706.png|348]]

**Qué hace:** Una librería de **optimización automática de hiperparámetros** — en vez de probar manualmente (o con fuerza bruta tipo grid search) combinaciones de hiperparámetros de un modelo, Optuna usa algoritmos de búsqueda inteligente (ej. optimización bayesiana) para converger más rápido hacia la mejor combinación.

**Hasta dónde llega:** Optuna optimiza hiperparámetros de un modelo ya definido (número de árboles, tasa de aprendizaje, profundidad...) — no elige el algoritmo por ti, ni hace feature engineering ni selección de variables; opera estrictamente sobre el espacio de hiperparámetros que tú le defines.

### Weights & Biases (W&B)

![[Pasted image 20260715081920.png|378]]

**Qué hace:** Una plataforma (con librería cliente en Python) de **experiment tracking** y colaboración para equipos de ML — muy similar en propósito a MLflow, pero con una interfaz visual más pulida, fuerte enfoque en Deep Learning (visualiza curvas de entrenamiento en tiempo real, compara corridas, gestiona artefactos y datasets), y mejor soporte para colaboración en equipos grandes/distribuidos.

**Hasta dónde llega:** Al igual que MLflow, W&B registra y visualiza experimentos — no entrena modelos ni los despliega a producción; es una capa de observabilidad sobre el proceso de experimentación, no un motor de entrenamiento ni de servicio de modelos.

---

## Tabla Resumen: "esto NO es lo que crees"

|Librería|Mito común|Realidad|
|---|---|---|
|Scikit-learn|"Sirve para Deep Learning"|Solo ML clásico; no tiene soporte real de redes profundas ni GPU|
|TensorFlow / PyTorch|"Reemplazan a Pandas"|Trabajan con tensores para entrenar redes; no manipulan datos tabulares del día a día|
|PyTorch|"Es una evolución de scikit-learn"|Paradigmas y propósitos distintos; se usan juntas, no una reemplaza a la otra|
|Hugging Face|"Entrena modelos desde cero por ti"|Da acceso a modelos preentrenados y herramientas de fine-tuning, construido sobre PyTorch/TF|
|LangChain|"Entrena o aloja modelos"|Orquesta llamadas a modelos ya existentes vía API|
|MLflow|"Orquesta pipelines de datos"|Solo tracking y registro de modelos, no ejecución de pipelines|
|Optuna|"Elige el algoritmo/features por ti"|Solo optimiza hiperparámetros de un modelo ya definido|
|ONNX|"Es un framework de entrenamiento"|Solo formato de intercambio para inferencia optimizada|

---

## Ver también

- Cheat-sheet técnico de XGBoost/LightGBM/CatBoost (sintaxis, API, comparativa): `Gradient Boosting/01 - Introducción y Panorama.md`
- Cheat-sheet técnico de scikit-learn (sintaxis y API): `Scikit-learn/01 - Introducción, Filosofía y la API Consistente.md`
- [[06-Familia-de-Algoritmos-ML]]
- [[08-Plataformas-de-Datos-y-ML]]
- [[09-MLOps-en-Profundidad]]
- [[10-IA-Generativa-LLMs-y-Agentes]]
