---
tags: [data-science, machine-learning, fundamentos, panorama-general]
aliases: [Panorama DS/ML/DL, Fundamentos de Ciencia de Datos]
---
# Parte 1 — Fundamentos y Panorama General

> **Objetivo de esta nota:** construir el "_mapa mental_" que conecta todo lo demás. Antes de aprender herramientas o roles, necesitas entender **qué es cada cosa y cómo se relaciona con las demás**. Esta nota es la raíz de la que cuelgan [[02-Roles-y-Carreras-en-DS-ML]], [[03-Arquitectura-Empresarial-de-Datos-y-ML]], [[04-Ingenieria-de-Datos]] y [[05-Ciencia-de-Datos-Estadistica-y-Fundamentos-ML]].
---

## Tabla de Contenidos

1. [[#¿Qué es Data Science?|¿Qué es Data Science?]]
2. [[#¿Qué Diferencia Hay Entre IA, ML y Deep Learning?|¿Qué Diferencia Hay Entre IA, ML y Deep Learning?]]
3. [[#¿Qué es un Modelo?|¿Qué es un Modelo?]]
4. [[#¿Qué es un Algoritmo?|¿Qué es un Algoritmo?]]
5. [[#¿Qué es un Pipeline?|¿Qué es un Pipeline?]]
6. [[#¿Qué significa Producción?|¿Qué significa Producción?]]
7. [[#¿Qué significa Entrenamiento?|¿Qué significa Entrenamiento?]]
8. [[#¿Qué significa Inferencia?|¿Qué significa Inferencia?]]
9. [[#¿Qué significa Feature?|¿Qué significa Feature?]]
10. [[#¿Qué significa Embedding?|¿Qué significa Embedding?]]
11. [[#¿Qué es un Vector?|¿Qué es un Vector?]]
12. [[#¿Qué significa Fine-tuning?|¿Qué significa Fine-tuning?]]
13. [[#¿Cómo se Conecta Absolutamente Todo?|¿Cómo se Conecta Absolutamente Todo?]]
14. [[#Ver también|Ver también]]

---
## ¿Qué es Data Science?

La **Ciencia de Datos (_Data Science_) no es una tecnología ni una herramienta**: es una disciplina que combina tres cosas para responder preguntas y tomar decisiones a partir de datos:

1. **Estadística y matemáticas** → para entender la incertidumbre y los patrones.
2. **Programación / ingeniería** → para procesar datos a escala y construir sistemas.
3. **Conocimiento de negocio (domain knowledge)** → para saber qué preguntas vale la pena hacer.

Esta idea se representa clásicamente con el **diagrama de Venn de Drew Conway**: donde se cruzan Hacking Skills, Math & Stats Knowledge y Substantive Expertise, ahí vive la Ciencia de Datos. Si te falta el conocimiento de negocio, terminas con modelos técnicamente correctos pero inútiles. Si te falta la estadística, terminas engañado por patrones que son solo ruido. Si te falta la programación, tus ideas nunca llegan a producción.

**En la práctica**, un proyecto de Data Science recorre un ciclo:

```
Pregunta de negocio → Datos → Exploración (EDA) → Modelado → Validación → Comunicación/Decisión → (a veces) Producción
```

La clave: **Data Science es el proceso completo de convertir datos crudos en decisiones**, y el Machine Learning es solo una de las herramientas que puede usar ese proceso (no siempre hace falta un modelo; a veces basta un buen análisis estadístico o un dashboard).

---

## ¿Qué Diferencia Hay Entre IA, ML y Deep Learning?

Son **círculos concéntricos**, no sinónimos:

```
┌──────────────────────────────────────────────┐
│  Inteligencia Artificial (IA)                │
│  "Sistemas que imitan capacidades humanas"   │
│  ┌─────────────────────────────────────────┐ │
│  │  Machine Learning (ML)                  │ │
│  │  "Sistemas que aprenden de datos        │ │
│  │   en vez de reglas escritas a mano"     │ │
│  │  ┌───────────────────────────────────┐  │ │
│  │  │  Deep Learning (DL)               │  │ │
│  │  │  "ML con redes neuronales de      │  │ │
│  │  │   muchas capas"                   │  │ │
│  │  └───────────────────────────────────┘  │ │
│  └─────────────────────────────────────────┘ │
└──────────────────────────────────────────────┘
```

- **IA**: el campo más amplio. Incluye desde un sistema experto basado en reglas `if/else` (IA "simbólica", años 60-80) hasta un LLM moderno. Lo único que exige es que el sistema realice tareas que asociamos con inteligencia (razonar, decidir, reconocer patrones).
- **ML**: un subconjunto de la IA donde **el sistema aprende reglas a partir de datos**, en vez de que un humano las programe explícitamente. En vez de escribir "si el email contiene 'gratis' y 'urgente', es spam", le muestras miles de ejemplos etiquetados y el algoritmo _deriva_ las reglas.
- **Deep Learning**: un subconjunto del ML que usa **redes neuronales artificiales con muchas capas** (_"profundas"_). Su ventaja es que puede aprender automáticamente qué características (_features_) importan, sin que un humano las diseñe a mano — algo crítico para datos no estructurados como imágenes, audio y texto.

**¿Por qué importa la distinción?** Porque determina qué herramienta usar. Un problema de predicción de churn con datos tabulares casi siempre se resuelve mejor y más barato con ML clásico (ej. XGBoost) que con Deep Learning. El DL brilla cuando los datos son masivos y no estructurados (imágenes, texto, audio, video).

---

## ¿Qué es un Modelo?

Un **modelo** es una representación matemática (una función) que mapea entradas a salidas, aprendida a partir de datos:

$$
f(x) ≈ y
$$

```
f(x) ≈ y
```

Donde $x$ son tus features (entradas) y $y$ es lo que quieres predecir. El "modelo" no es más que los **parámetros** de esa función ya ajustados. Ejemplos:

- **Una regresión lineal:** el modelo son los coeficientes $β₀, β₁, β₂...$.
- **Un árbol de decisión:** el modelo es la estructura del árbol (qué preguntas hace y en qué orden).
- **Una red neuronal:** el modelo son los millones/billones de pesos (_weights_) de sus conexiones.

Un modelo **entrenado** es literalmente un archivo (`.pkl`, `.h5`, `.onnx`, `.safetensors`) que contiene esos parámetros ya ajustados, listo para hacer [[#¿Qué significa inferencia?|inferencia]].

---

## ¿Qué es un Algoritmo?

Aquí hay una confusión muy común: **algoritmo ≠ modelo**.

- El **algoritmo** es el _procedimiento_ que usas para _encontrar_ los parámetros del modelo a partir de los datos (ej. "regresión logística", "random forest", "gradient boosting", "descenso de gradiente").
- El **modelo** es el _resultado_ de aplicar ese algoritmo a tus datos: los parámetros ya ajustados.

> [!IMPORTANT] Analogía:
> El algoritmo es la **receta**; el modelo es el **pastel ya horneado** con esos ingredientes específicos. Puedes usar la misma receta (algoritmo) con distintos ingredientes (datos) y obtener pasteles (modelos) distintos.

---

## ¿Qué es un Pipeline?

Un **pipeline** es una secuencia encadenada de pasos de procesamiento, donde la salida de uno es la entrada del siguiente, ejecutada de forma reproducible y (_idealmente_) automatizada. Existen dos sentidos que se solapan mucho en el trabajo diario:

- **Pipeline de datos**: extracción → limpieza → transformación → carga (ver [[04-Ingenieria-de-Datos]]).
- **Pipeline de ML**: ingesta de datos → feature engineering → entrenamiento → validación → despliegue → monitoreo (ver [[03-Arquitectura-Empresarial-de-Datos-y-ML]]).

La razón de que existan los pipelines: **reproducibilidad y automatización**. Sin un pipeline, cada vez que llega un dato nuevo alguien tiene que ejecutar manualmente 15 scripts en el orden correcto. Con un pipeline, ese proceso es un solo comando (o se dispara solo) y produce el mismo resultado cada vez, dado el mismo input.

---

## ¿Qué significa Producción?

"Producción" (_production / prod_) es el **entorno donde el sistema es usado por usuarios reales o toma decisiones reales**, en contraste con:

- **Desarrollo (dev)**: donde el código se escribe y se prueba de forma aislada.
- **Staging / QA**: un entorno "_espejo_" de producción donde se prueba todo junto antes del lanzamiento real.
- **Producción (prod)**: el entorno real, con datos reales, tráfico real y consecuencias reales si algo falla.

"Llevar un modelo a producción" significa que dejó de ser un experimento en un notebook y ahora **responde a peticiones reales de forma continua y confiable** — con monitoreo, control de versiones, manejo de errores, límites de latencia, etc. Este salto (de notebook a producción) es, en la práctica, el trabajo más difícil y menos glamoroso de la ciencia de datos, y es la razón de ser de disciplinas como [[#Rol MLOps Engineer|MLOps]] (ver [[02-Roles-y-Carreras-en-DS-ML]]).

---

## ¿Qué significa Entrenamiento?

**Entrenar** (_training_) es el proceso de **ajustar los parámetros de un modelo** para que minimice el error entre sus predicciones y los valores reales, usando un conjunto de datos de entrenamiento. Mecánicamente:

1. El modelo hace una predicción con sus parámetros actuales (al inicio, aleatorios).
2. Se mide el error contra el valor real (la "función de pérdida" o _loss function_).
3. Se ajustan los parámetros para reducir ese error (ej. mediante _gradient descent_).
4. Se repite miles/millones de veces (_epochs_, _iteraciones_) hasta que el error deja de bajar significativamente.

El resultado final de este proceso es el **modelo entrenado** mencionado en la sección 3.

---

## ¿Qué significa Inferencia?

**Inferencia** (_inference_) es usar un modelo _ya entrenado_ para generar una predicción sobre datos _nuevos_ que nunca vio durante el entrenamiento.

```
Entrenamiento: datos históricos + respuestas conocidas → ajustar el modelo
Inferencia:    dato nuevo (sin respuesta conocida) → modelo entrenado → predicción
```

> [!NOTE] Es la diferencia entre "aprender a manejar" (entrenamiento, con instructor y correcciones) y "manejar de verdad, solo, en una calle nueva" (inferencia).
> En producción, el 99% de las veces que interactúas con un modelo es haciendo inferencia, no entrenamiento — por eso existe la distinción entre infraestructura de entrenamiento (GPUs potentes, corridas largas) e infraestructura de inferencia (baja latencia, alta disponibilidad, escalado horizontal).

---

## ¿Qué significa Feature?

Una **feature** (característica o variable predictora) es **una columna de tus datos que el modelo usa como entrada** para hacer una predicción. Si quieres predecir si un cliente hará _churn_ (se irá), tus features podrían ser: antigüedad del cliente, número de quejas, monto de la última factura, uso mensual promedio, etc.

No todas las columnas son buenas features "tal cual" — de ahí nace el **[[05-Ciencia-de-Datos-Estadistica-y-Fundamentos-ML#Feature Engineering|Feature Engineering]]**: transformar datos crudos en variables que realmente ayuden al modelo a aprender. Un **[[03-Arquitectura-Empresarial-de-Datos-y-ML#Feature Store|Feature Store]]** es la pieza de infraestructura que existe específicamente para almacenar, versionar y servir features de forma consistente entre entrenamiento e inferencia.

---

## ¿Qué significa Embedding?

Un **embedding** es una **representación numérica densa (un vector) de algo que originalmente no es numérico** — una palabra, una imagen, un usuario, un producto — de forma que ese vector captura su _significado_ o _similitud_ con otros elementos.

La idea clave: **cosas parecidas terminan con vectores parecidos** (cercanos en el espacio matemático). Por ejemplo, en un buen embedding de palabras, el vector de "rey" menos el vector de "hombre" más el vector de "mujer" da un vector muy cercano al de "reina" — el embedding capturó una relación semántica sin que nadie la programara explícitamente.

Los embeddings son la base de:

- Los sistemas de recomendación (usuarios y productos como vectores).
- La búsqueda semántica y RAG (_Retrieval-Augmented Generation_) con LLMs.
- El funcionamiento interno de cualquier red neuronal moderna (cada capa transforma su entrada en una nueva representación vectorial).

---

## ¿Qué es un Vector?

Matemáticamente, un **vector** es simplemente **una lista ordenada de números**: `[0.23, -1.4, 0.87, ...]`. En ciencia de datos casi todo termina siendo un vector:

- Una fila de tu dataset (cada feature es una posición del vector).
- Un embedding (sección anterior).
- Los pesos de una neurona.

Cuando tienes muchos vectores, tienes una **matriz** (una tabla de vectores); cuando tienes matrices apiladas en más dimensiones, tienes un **tensor** — la estructura de datos central en frameworks de Deep Learning como [[07-Librerias-de-Data-Science-y-ML#PyTorch|Pytorch]] y [[07-Librerias-de-Data-Science-y-ML#TensorFlow|TensorFlow]] (de ahí el nombre "_Tensor_Flow__").

Un **vector database / vector store** (ej. Pinecone, Weaviate, Milvus, pgvector) es una base de datos optimizada para almacenar millones de vectores y encontrar rápidamente "los vectores más parecidos a este otro vector" (_similarity search_) — la pieza de infraestructura que hace posible la búsqueda semántica y RAG.

---

## ¿Qué significa Fine-tuning?

**Fine-tuning** (ajuste fino) es tomar un modelo **ya preentrenado** en una tarea general (por ejemplo, un LLM entrenado con una fracción enorme de internet) y **continuar entrenándolo con un dataset más pequeño y específico** para especializarlo en una tarea o dominio concreto.

Por qué existe: entrenar un modelo grande desde cero cuesta millones de dólares y requiere datasets gigantescos. El fine-tuning aprovecha el conocimiento general que el modelo ya adquirió (_transfer learning_) y solo lo "afina" para tu caso — mucho más barato, rápido y con muchísimos menos datos.

>[!IMPORTANT] **Ejemplo:**
>Tomar un LLM genérico y hacerle fine-tuning con miles de tickets de soporte de tu empresa para que responda con el tono y el conocimiento específico de tu producto. Alternativas más ligeras al fine-tuning completo: _prompt engineering_, _RAG_ (darle contexto externo sin reentrenar) y técnicas eficientes como _LoRA_ (Low-Rank Adaptation), que ajustan solo una fracción pequeña de los parámetros.

---

## ¿Cómo se Conecta Absolutamente Todo?

Aquí está el mapa completo, de punta a punta, para que veas cómo cada concepto anterior encaja en un flujo real:

```
1. Una empresa genera DATOS (transacciones, clics, sensores, formularios)
        │
2. Esos datos se mueven y transforman → INGENIERÍA DE DATOS
   (ETL/ELT, pipelines, data warehouse/lake)  → [[04-Ingenieria-de-Datos]]
        │
3. Un Data Analyst / Data Scientist explora los datos → EDA
   (estadística descriptiva, visualización)
        │
4. Se construyen FEATURES a partir de los datos crudos
   (feature engineering) → [[05-Ciencia-de-Datos-Estadistica-y-Fundamentos-ML]]
        │
5. Se elige un ALGORITMO y se ENTRENA un MODELO con esas features
   (ML clásico o Deep Learning, según el problema)
        │
6. Se VALIDA el modelo (cross-validation, métricas: precision, recall, RMSE...)
   para asegurarse de que generaliza y no solo "memorizó" (overfitting)
        │
7. El modelo se registra en un MODEL REGISTRY (versión, metadata, métricas)
        │
8. Se DESPLIEGA a producción (deployment) — MLOps
   El modelo ahora hace INFERENCIA sobre datos nuevos en tiempo real o en lote
        │
9. Se MONITOREA en producción (¿el rendimiento se degrada? ¿hay data drift?)
        │
10. Si se degrada, se REENTRENA con datos nuevos → vuelve al paso 5
```

**La idea central que debes internalizar**: Data Science / ML **no es un evento, es un ciclo continuo**. Cada rol de los que verás en [[02-Roles-y-Carreras-en-DS-ML]] es responsable de una o varias etapas de este ciclo. Cada herramienta que verás en [[04-Ingenieria-de-Datos]] existe para resolver un problema específico en alguna de estas etapas. Y cada concepto estadístico de [[05-Ciencia-de-Datos-Estadistica-y-Fundamentos-ML]] existe para que la etapa de validación (paso 6) sea rigurosa y no un engaño.

Cuando en el futuro aprendas una herramienta nueva (digamos, Kafka o dbt), la primera pregunta que debes hacerte es: **¿en qué paso de este ciclo encaja, y qué problema de ese paso resuelve?** Esa es la pregunta que evita que el conocimiento se sienta como una lista de nombres sueltos.

---

## Ver también

- [[02-Roles-y-Carreras-en-DS-ML]]
- [[03-Arquitectura-Empresarial-de-Datos-y-ML]]
- [[04-Ingenieria-de-Datos]]
- [[05-Ciencia-de-Datos-Estadistica-y-Fundamentos-ML]]
