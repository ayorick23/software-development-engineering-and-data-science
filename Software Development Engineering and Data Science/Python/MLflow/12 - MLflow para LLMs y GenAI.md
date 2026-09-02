---
tags: [mlflow, mlops, llm, genai, tracing, cheat-sheet]
---

# 12 — MLflow para LLMs y GenAI

> Se apoya en [[02 - Tracking - Fundamentos y API de Logging]] y [[06 - Model Format y Flavors]].

Desde la versión 2.x, MLflow extendió Tracking y Models para cubrir el ciclo de vida de aplicaciones basadas en LLMs: prompts, cadenas de llamadas (chains/agents), tracing y evaluación de calidad de texto.

## `mlflow.log_param` para prompts — versionado simple

El caso más básico: tratar un prompt como cualquier otro parámetro versionado.

```python
import mlflow

with mlflow.start_run(run_name="prompt-v3-resumen-tickets"):
    mlflow.log_param("prompt_template", prompt_template)
    mlflow.log_param("model_name", "claude-sonnet-5")
    mlflow.log_param("temperature", 0.2)

    respuesta = llamar_llm(prompt_template.format(ticket=ticket_texto))

    mlflow.log_text(respuesta, "output_ejemplo.txt")
    mlflow.log_metric("longitud_respuesta", len(respuesta))
```

## MLflow Tracing — observabilidad de llamadas a LLMs

**Tracing** captura automáticamente cada paso de una cadena de llamadas (LLM, retriever, herramientas/tools) con latencia, tokens usados, inputs/outputs de cada etapa — indispensable para depurar aplicaciones RAG o agentes.

### Autologging de tracing para SDKs populares

```python
import mlflow

mlflow.openai.autolog()      # traza automáticamente llamadas a la API de OpenAI
mlflow.anthropic.autolog()   # traza automáticamente llamadas a la API de Anthropic
mlflow.langchain.autolog()   # traza cadenas completas de LangChain
mlflow.llama_index.autolog() # traza pipelines de LlamaIndex
```

Con autolog activo, cada llamada queda registrada como un **trace** navegable en la UI (`mlflow ui`, pestaña "Traces"), mostrando el árbol completo de spans sin escribir código de logging manual.

### Tracing manual con decoradores

```python
import mlflow

@mlflow.trace(name="recuperar_contexto", span_type="RETRIEVER")
def recuperar_documentos(query):
    return vector_store.similarity_search(query, k=5)

@mlflow.trace(name="generar_respuesta", span_type="LLM")
def generar_respuesta(query, contexto):
    prompt = construir_prompt(query, contexto)
    return llamar_llm(prompt)

@mlflow.trace(name="pipeline_rag")
def responder(query):
    contexto = recuperar_documentos(query)
    return generar_respuesta(query, contexto)
```

### Tracing manual con context manager (control más fino)

```python
with mlflow.start_span(name="paso_custom", span_type="TOOL") as span:
    span.set_inputs({"query": query})
    resultado = ejecutar_tool(query)
    span.set_outputs({"resultado": resultado})
    span.set_attribute("tokens_usados", 340)
```

## Empaquetar una aplicación LLM como modelo MLflow

El flavor `pyfunc` (ver [[06 - Model Format y Flavors]]) sirve igual para envolver una cadena RAG completa, no solo modelos tradicionales:

```python
import mlflow.pyfunc

class AsistenteRAG(mlflow.pyfunc.PythonModel):
    def load_context(self, context):
        self.vector_store = cargar_vector_store(context.artifacts["index"])
        self.llm_client = crear_cliente_llm()

    def predict(self, context, model_input, params=None):
        preguntas = model_input["pregunta"].tolist()
        return [self._responder(p) for p in preguntas]

    def _responder(self, pregunta):
        contexto = self.vector_store.similarity_search(pregunta, k=3)
        return self.llm_client.generar(pregunta, contexto)

with mlflow.start_run():
    mlflow.pyfunc.log_model(
        artifact_path="asistente_rag",
        python_model=AsistenteRAG(),
        artifacts={"index": "ruta/al/indice_vectorial"},
        pip_requirements=["langchain", "chromadb", "anthropic"],
    )
```

Esto le da a una aplicación de GenAI el mismo tratamiento que a un modelo clásico: versionado en el Registry, signature, y la posibilidad de servirla con `mlflow models serve`.

## Evaluación de calidad de texto — LLM-as-judge

`mlflow.evaluate` (ver [[11 - Evaluación de Modelos (mlflow.evaluate)]]) soporta `model_type="question-answering"` y métricas basadas en otro LLM como evaluador:

```python
from mlflow.metrics.genai import answer_correctness, answer_relevance

result = mlflow.evaluate(
    model=modelo_uri,
    data=df_preguntas_respuestas,   # columnas: inputs, ground_truth, predictions
    targets="ground_truth",
    predictions="predictions",
    model_type="question-answering",
    extra_metrics=[
        answer_correctness(model="openai:/gpt-4"),   # usa un LLM como "juez" de calidad
        answer_relevance(model="openai:/gpt-4"),
    ],
)
```

## MLflow AI Gateway / Deployments Server

Un proxy unificado para múltiples proveedores de LLM (OpenAI, Anthropic, Azure OpenAI, Cohere), útil para centralizar credenciales y aplicar rate limiting/caching en un solo punto:

```yaml
# config.yaml
endpoints:
  - name: chat-claude
    endpoint_type: llm/v1/chat
    model:
      provider: anthropic
      name: claude-sonnet-5
      config:
        anthropic_api_key: $ANTHROPIC_API_KEY
```

```bash
mlflow gateway start --config-path config.yaml --port 7000
```

```python
import mlflow.deployments

client = mlflow.deployments.get_deploy_client("http://localhost:7000")
response = client.predict(
    endpoint="chat-claude",
    inputs={"messages": [{"role": "user", "content": "Resume este ticket: ..."}]},
)
```

Esto permite que el código de la aplicación llame a un único endpoint interno, sin acoplarse directamente al SDK de cada proveedor — cambiar de proveedor es un cambio de configuración, no de código.

## Ver también

- [[02 - Tracking - Fundamentos y API de Logging]]
- [[06 - Model Format y Flavors]]
- [[11 - Evaluación de Modelos (mlflow.evaluate)]]
- `Machine Learning/10-IA-Generativa-LLMs-y-Agentes.md`
