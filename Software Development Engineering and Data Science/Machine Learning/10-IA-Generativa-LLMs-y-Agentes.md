---
tags: [ia-generativa, llm, rag, agentes, transformers, langchain]
aliases: [IA Generativa, LLMs y Agentes, GenAI]
---
# Parte 10 — IA Generativa: LLMs, RAG y Agentes

> Cierra (por ahora) la serie que empezó en [[01-Fundamentos-y-Panorama-General]]. Retoma varios conceptos ya sembrados ahí (embedding, vector, fine-tuning) y en [[06-Familia-de-Algoritmos-ML#LLMs (Large Language Models)]], y los profundiza. El orden de esta nota sigue, a propósito, la secuencia lógica: primero la arquitectura que lo hizo posible (Transformers), luego el modelo (LLM), luego cómo se especializa (fine-tuning/LoRA/PEFT), luego cómo se le da conocimiento externo (embeddings/vector DB/RAG), luego cómo se le da instrucciones (prompt engineering), y finalmente cómo se le da autonomía (agentes/MCP) y con qué herramientas se construye todo esto.

---

## Transformers (la arquitectura base)

**Qué es:** La arquitectura de red neuronal, presentada en el paper "Attention Is All You Need" (Google, 2017), que se convirtió en la base de prácticamente todos los LLMs modernos. Su innovación central es el **mecanismo de atención (self-attention)**: en vez de procesar una secuencia de texto palabra por palabra en orden estricto (como hacían las RNNs anteriores), el Transformer puede **relacionar cada palabra con todas las demás palabras de la secuencia simultáneamente**, pesando cuáles son más relevantes para entender el significado de cada una en ese contexto específico.

**Por qué reemplazó a las arquitecturas anteriores (RNNs/LSTMs):** Las RNNs procesaban texto de forma secuencial, palabra por palabra, lo que las hacía lentas de entrenar (no se pueden paralelizar bien) y con dificultad para "recordar" relaciones entre palabras muy alejadas en un texto largo. Los Transformers procesan la secuencia completa en paralelo y manejan relaciones de largo alcance de forma mucho más efectiva — esto, combinado con que escalan muy bien con más datos y más cómputo, es lo que hizo viable entrenar modelos del tamaño de los LLMs actuales.

---

## LLMs (Large Language Models)

Ya introducidos conceptualmente en [[06-Familia-de-Algoritmos-ML#11. LLMs (Large Language Models)]]. Aquí el detalle de **cómo se entrenan**, en dos grandes etapas:

1. **Preentrenamiento (pretraining):** el modelo se entrena sobre cantidades masivas de texto (una fracción enorme de internet, libros, código) con una tarea simple — predecir la siguiente palabra (token) dado el texto anterior. De ahí surge, de forma emergente, la capacidad del modelo de "entender" gramática, hechos, razonamiento básico y patrones del lenguaje, sin que nadie le haya enseñado esas reglas explícitamente.
2. **Alineación (alignment):** después del preentrenamiento, el modelo sabe "completar texto de forma plausible", pero no necesariamente sabe **seguir instrucciones de forma útil y segura**. Esta etapa (que incluye _instruction tuning_ y **RLHF** — Reinforcement Learning from Human Feedback, mencionado en [[06-Familia-de-Algoritmos-ML#12. RL (Reinforcement Learning / Aprendizaje por Refuerzo)]]) ajusta el modelo usando ejemplos de buenas respuestas y retroalimentación humana, para que responda de forma útil, siga instrucciones, y evite comportamientos indeseados.

---

## Embeddings (revisitado en el contexto de IA Generativa)

Ya definidos en [[01-Fundamentos-y-Panorama-General#10. ¿Qué significa embedding?]]. En el contexto específico de LLMs/RAG, un embedding es la representación vectorial de un texto (una palabra, una oración, un documento completo) generada por un modelo especializado (_embedding model_), diseñada específicamente para que **textos con significado similar tengan vectores cercanos entre sí** — esta propiedad es exactamente lo que hace posible la búsqueda semántica descrita a continuación.

---

## Vector DB (Base de Datos Vectorial)

Ya introducida en [[01-Fundamentos-y-Panorama-General#¿Qué es un Vector?]]. Aquí el detalle de su rol específico en aplicaciones de IA Generativa: cuando conviertes miles o millones de documentos (o fragmentos de documentos) en embeddings y los guardas en una Vector DB (Pinecone, Weaviate, Milvus, Chroma, o extensiones vectoriales de bases tradicionales como pgvector en PostgreSQL), puedes luego tomar la pregunta de un usuario, convertirla también en un embedding, y **buscar en la base de datos los documentos cuyo vector esté más cerca** (_similarity search_) — encontrando así los documentos más relevantes semánticamente, no solo por coincidencia exacta de palabras (como haría una búsqueda tradicional por palabras clave).

---

## RAG (Retrieval-Augmented Generation)

**Qué es:** Una técnica que combina **recuperación de información** (retrieval, usando una Vector DB) con **generación de texto** (un LLM): en vez de que el LLM responda solo con lo que "recuerda" de su entrenamiento, primero se buscan documentos relevantes a la pregunta (retrieval), y luego se le da esos documentos al LLM **como contexto dentro del prompt**, para que genere su respuesta basándose en esa información específica y actualizada.

**Por qué nació — el problema que resuelve:** Un LLM tiene dos limitaciones estructurales importantes: (1) su conocimiento está "congelado" en la fecha de su entrenamiento — no sabe nada de eventos posteriores ni de información privada de tu empresa que nunca vio; y (2) tiende a "alucinar" (inventar información con total confianza) cuando no sabe algo. RAG ataca ambos problemas: le das al modelo información **actualizada y específica** (ej. los documentos internos de tu empresa) directamente en el contexto de cada pregunta, reduciendo drásticamente la necesidad de que el modelo "adivine" y dándole una base factual concreta sobre la cual responder.

**Cuándo usarlo (vs. Fine-tuning, ver siguiente sección):** RAG es la opción por defecto cuando necesitas que el modelo tenga acceso a **conocimiento específico y que cambia con frecuencia** (documentación interna, políticas de la empresa, catálogo de productos actualizado) — es mucho más barato, rápido de actualizar (solo hay que actualizar los documentos indexados, no reentrenar nada) y transparente (puedes mostrar de qué documento salió cada respuesta) que el fine-tuning.

---

## Fine-Tuning (revisitado en profundidad)

Ya definido conceptualmente en [[01-Fundamentos-y-Panorama-General#¿Qué significa Fine-tuning?]]. La distinción clave que hay que tener clarísima:

**RAG le da al modelo información nueva sin cambiar sus parámetros** (el modelo sigue siendo el mismo, solo lee documentos adicionales en cada consulta). **Fine-tuning literalmente ajusta los parámetros internos del modelo** entrenándolo más, con un dataset específico — cambia _cómo_ el modelo responde (tono, formato, estilo, comportamiento en tareas muy específicas), no solo _qué información_ tiene disponible.

**Cuándo usar fine-tuning en vez de (o junto con) RAG:** Cuando necesitas cambiar el **comportamiento** del modelo de forma consistente y profunda — un tono de marca muy específico, un formato de salida particular que debe seguir siempre, o especialización en una tarea muy técnica y repetitiva (ej. clasificar tickets de soporte con una taxonomía interna muy específica) donde el prompt engineering solo no basta. En la práctica moderna, muchas aplicaciones serias combinan **ambos**: fine-tuning para el comportamiento/formato base, RAG para la información actualizada.

---

## LoRA (Low-Rank Adaptation)

**Qué es:** Una técnica de fine-tuning **eficiente** — en vez de reentrenar (y por tanto tener que almacenar) todos los miles de millones de parámetros del modelo original, LoRA congela el modelo original y solo entrena un número mucho menor de parámetros adicionales ("matrices de bajo rango") que se insertan en el modelo, capturando el ajuste necesario con una fracción del costo computacional y de memoria.

**Por qué nació:** El fine-tuning completo de un LLM grande requiere GPUs muy costosas y mucho tiempo — inviable para la mayoría de equipos y empresas fuera de los grandes labs de IA. LoRA hizo el fine-tuning **accesible**: es posible especializar un modelo grande en una sola GPU de consumo o en pocas horas, en vez de en un cluster durante días.

---

## PEFT (Parameter-Efficient Fine-Tuning)

**Qué es:** El término **paraguas** para toda la familia de técnicas de fine-tuning eficiente, de las cuales LoRA es la más popular y usada en la práctica actual (otras incluyen _prefix tuning_, _adapter layers_). PEFT también es el nombre de la librería de Hugging Face que implementa estas técnicas de forma lista para usar.

**Relación con LoRA:** LoRA es **una técnica específica dentro de** la categoría PEFT — no son alternativas, es una relación de "tipo de" (PEFT es el género, LoRA es la especie más popular).

---

## Prompt Engineering

**Qué es:** La práctica de diseñar cuidadosamente las instrucciones (el _prompt_) que se le dan a un LLM para obtener la respuesta deseada — incluye técnicas como dar ejemplos dentro del propio prompt (_few-shot prompting_), pedir al modelo que razone paso a paso (_chain-of-thought_), estructurar el prompt con roles y formato claro, o especificar restricciones explícitas de formato de salida.

**Por qué es una disciplina real y no "solo escribir bien":** El mismo modelo, con la misma pregunta de fondo, puede dar respuestas de calidad muy distinta según cómo se formule el prompt — es, en la práctica, la forma más rápida y barata de mejorar el comportamiento de un LLM, **antes** de considerar RAG o fine-tuning (que cuestan más tiempo/dinero de implementar). La jerarquía práctica de esfuerzo suele ser: prompt engineering primero → RAG si se necesita conocimiento externo → fine-tuning/LoRA si el prompt engineering y RAG no son suficientes para el comportamiento deseado.

---

## Agentes (AI Agents)

**Qué es:** Un sistema donde un LLM no solo responde texto, sino que puede **decidir y ejecutar acciones** — llamar herramientas externas (buscar en internet, consultar una base de datos, ejecutar código, llamar una API), observar el resultado de esa acción, y decidir el siguiente paso, de forma iterativa hasta completar una tarea compleja de varios pasos.

**Por qué nació este patrón:** Un LLM por sí solo, respondiendo un solo prompt, está limitado a lo que puede razonar internamente en una sola pasada — no puede, por ejemplo, revisar el calendario real de alguien, ejecutar una consulta SQL real, o navegar un sitio web actual. Los agentes le dan al LLM la capacidad de **actuar sobre el mundo real** (o sobre sistemas reales) de forma autónoma, encadenando múltiples pasos de razonamiento y acción — el patrón de diseño se conoce comúnmente como **ReAct** (Reasoning + Acting): el modelo alterna entre "pensar qué hacer" y "ejecutar una acción", usando el resultado de cada acción para decidir la siguiente.

---

## MCP (Model Context Protocol)

**Qué es:** Un protocolo abierto (creado por Anthropic) que estandariza **cómo los LLMs se conectan con herramientas y fuentes de datos externas** — en vez de que cada aplicación tenga que construir una integración a medida y distinta para conectar un LLM con, digamos, Google Drive, Slack o una base de datos interna, MCP define una interfaz común que cualquier modelo compatible y cualquier fuente de datos/herramienta compatible pueden usar para hablar entre sí.

**Por qué nació — el problema que resuelve:** Antes de protocolos como MCP, conectar un LLM a _N_ herramientas distintas requería construir _N_ integraciones a medida, cada una con su propia lógica — un problema de "M modelos × N herramientas" que crece de forma combinatoria. MCP estandariza esa conexión (similar en espíritu a cómo USB estandarizó la conexión de periféricos a computadoras, en vez de que cada dispositivo necesitara su propio puerto) — cualquier modelo que hable MCP puede usar cualquier herramienta que exponga un servidor MCP, sin integración a medida por cada par.

---

## Herramientas y plataformas del ecosistema

### Hugging Face

Ya cubierto en [[07-Librerias-de-Data-Science-y-ML#Hugging Face]]. En el contexto específico de IA Generativa, es el repositorio central donde se publican y descargan miles de modelos open-source (LLMs, modelos de embeddings, modelos de imagen), y su librería `transformers` permite cargarlos y usarlos (o fine-tunearlos, junto con PEFT) directamente.

### Ollama

**Qué hace:** Una herramienta que simplifica enormemente **correr LLMs open-source localmente** en tu propia máquina (laptop, servidor propio) — sin necesidad de configurar manualmente el entorno de inferencia, con un comando simple para descargar y ejecutar modelos como Llama, Mistral, entre otros.

**Por qué existe:** Correr un LLM localmente antes de Ollama requería bastante trabajo de configuración técnica (drivers, formatos de modelo, optimización de inferencia). Ollama empaqueta todo eso para que sea tan simple como `ollama run <modelo>`. Su caso de uso principal: privacidad total de datos (nada sale de tu máquina/red), experimentación sin costo de API, o despliegue en entornos sin conexión a internet.

### OpenAI

**Qué hace:** Empresa y proveedor de modelos LLM propietarios (familia GPT) accesibles vía API, junto con herramientas complementarias (generación de imágenes con DALL-E, agentes, herramientas de voz).

**Rol en el ecosistema:** Uno de los proveedores líderes de modelos de frontera accesibles vía API de pago — la alternativa a correr modelos propios (Hugging Face/Ollama) cuando se prioriza la máxima capacidad del modelo sin gestionar infraestructura de inferencia propia.

### Anthropic

**Qué hace:** Empresa creadora de Claude y del protocolo MCP mencionado arriba, con un enfoque explícito en la seguridad y la investigación de la alineación de modelos de IA, además de ofrecer sus modelos vía API.

### LangChain

Ya cubierto en [[07-Librerias-de-Data-Science-y-ML#LangChain]]. En este contexto: es uno de los frameworks más usados para **construir aplicaciones de RAG y agentes** — orquesta el flujo completo (recibir pregunta → buscar en Vector DB → construir el prompt con el contexto recuperado → llamar al LLM → devolver respuesta), y ofrece integraciones prearmadas con decenas de Vector DBs, LLMs y herramientas externas.

### LlamaIndex

**Qué hace:** Un framework especializado específicamente en la parte de **indexación y recuperación de datos** para RAG — mientras LangChain es más general (orquestación amplia de flujos con LLMs), LlamaIndex se enfoca con más profundidad en cómo estructurar, indexar y recuperar de forma óptima grandes colecciones de documentos para alimentar a un LLM.

**Cuándo elegir uno u otro:** LlamaIndex cuando el caso de uso central es específicamente RAG sobre documentos complejos (y quieres las estrategias de indexación más refinadas); LangChain cuando necesitas orquestar flujos más amplios que incluyen agentes, múltiples herramientas y lógica de aplicación más allá de solo la recuperación de documentos. En la práctica, muchos proyectos usan ambos juntos.

### CrewAI

**Qué hace:** Un framework especializado en **orquestar equipos de múltiples agentes de IA colaborando** entre sí, cada uno con un rol específico (ej. un agente "investigador", un agente "redactor", un agente "revisor") que se coordinan para completar una tarea compleja de forma conjunta, de forma similar a cómo un equipo humano dividiría el trabajo.

**Por qué existe frente a construir un solo agente:** Para tareas complejas, un solo agente generalista con muchas herramientas e instrucciones tiende a confundirse o perder foco. Dividir el trabajo en agentes especializados, cada uno con un objetivo y contexto más acotado, suele producir resultados más confiables — el mismo principio de "separación de responsabilidades" de la ingeniería de software, aplicado a agentes de IA.

### n8n

**Qué hace:** Una plataforma de **automatización de flujos de trabajo** (workflow automation), similar en espíritu a Zapier pero open-source y auto-hospedable, con interfaz visual de nodos conectados — y con soporte creciente para incorporar nodos de LLMs/agentes dentro de esos flujos de automatización más amplios (ej. "cuando llega un email nuevo → extraer información con un LLM → crear una tarea en el sistema interno → notificar por Slack").

**Rol en el ecosistema de IA Generativa:** Mientras LangChain/LlamaIndex/CrewAI están pensados para desarrolladores construyendo aplicaciones de IA con código, n8n permite construir automatizaciones que **incorporan** pasos de IA generativa dentro de flujos de negocio más amplios, con una barrera de entrada técnica mucho más baja (interfaz visual) — muy relevante para automatizar procesos operativos reales (como los que tocas en consultoría) sin construir una aplicación completa desde cero.

---

## El flujo completo de una aplicación RAG con agentes (síntesis)

```
1. Documentos de la empresa (PDFs, wikis, tickets...)
        │
2. Se dividen en fragmentos y se convierten en EMBEDDINGS
        │
3. Se almacenan en una VECTOR DB
        │
4. Llega la pregunta de un usuario
        │
5. La pregunta se convierte también en un embedding
        │
6. Se buscan los fragmentos más similares en la Vector DB → RETRIEVAL
        │
7. Esos fragmentos se insertan en el PROMPT junto con la pregunta → RAG
        │
8. El LLM genera una respuesta basada en ese contexto
        │
9. (Opcional) Si la tarea requiere acción, un AGENTE decide llamar
   herramientas externas (vía MCP u otras integraciones) y repite
   el ciclo de razonar → actuar → observar hasta completar la tarea
        │
10. Todo esto se orquesta con LangChain/LlamaIndex/CrewAI/n8n,
    usando modelos de OpenAI/Anthropic/Hugging Face/Ollama por debajo
```

---

## Ver también

- [[01-Fundamentos-y-Panorama-General]]
- [[06-Familia-de-Algoritmos-ML]]
- [[07-Librerias-de-Data-Science-y-ML]]
- [[09-MLOps-en-Profundidad]]