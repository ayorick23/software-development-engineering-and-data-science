---
tags: [streamlit, dashboards, integraciones, conexiones, cheat-sheet]
---

# 13 — Integración con el Ecosistema

> Consolida referencias de todo el cheat-sheet.

## `st.connection` — conexiones a fuentes de datos, con caching integrado

```python
conexion = st.connection("mi_bd", type="sql", url="postgresql://usuario:pass@host:5432/db")

df = conexion.query("SELECT * FROM demanda WHERE region = :region", params={"region": "RD"}, ttl=600)
```

`st.connection` es un wrapper que combina la conexión a la fuente de datos con `st.cache_data` internamente (el parámetro `ttl` controla cuánto tiempo se reutiliza el resultado antes de volver a consultar) — evita escribir manualmente la combinación de `@st.cache_resource` (para la conexión) + `@st.cache_data` (para los resultados de cada query) por separado.

### Configuración vía `secrets.toml`

```toml
# .streamlit/secrets.toml
[connections.mi_bd]
type = "sql"
url = "postgresql://usuario:pass@host:5432/db"
```

```python
conexion = st.connection("mi_bd")   # lee la configuración automáticamente desde secrets.toml
```

### Otros tipos de conexión soportados

```python
conexion_snowflake = st.connection("snowflake")
conexion_s3 = st.connection("s3", type="s3")   # requiere st-files-connection
```

Ver `SQL/` para fundamentos de SQL en general — `st.connection` cubre la parte de "cómo conectar desde Streamlit específicamente", no reemplaza el conocimiento de SQL necesario para escribir las queries.

## pandas — el flujo de datos central

```python
df = pd.read_csv(archivo_subido)
df_filtrado = df[df["region"] == region_seleccionada]
st.dataframe(df_filtrado)
```

Prácticamente toda manipulación de datos dentro de una app de Streamlit pasa por pandas — Streamlit no reemplaza pandas, solo provee la capa de visualización/interactividad encima. Ver `Scikit-learn/15 - Integraciones con el Ecosistema.md` para cómo pandas se conecta también con el resto del stack de ML.

## MLflow — tracking y model registry desde la app

Ya cubierto en profundidad en [[11 - Integración con Modelos de ML]] — resumen de los puntos de integración: cargar modelos del Registry (`mlflow.pyfunc.load_model`), consultar experimentos (`mlflow.search_runs`), y mostrar esos resultados con los elementos de datos cubiertos en [[05 - Mostrar Datos - DataFrames, Tablas y Métricas]].

## DVC — mostrar el estado de datos versionados

```python
import subprocess
import json

@st.cache_data(ttl=300)
def obtener_estado_dvc():
    resultado = subprocess.run(["dvc", "status", "--json"], capture_output=True, text=True)
    return json.loads(resultado.stdout) if resultado.stdout else {}

estado = obtener_estado_dvc()
if estado:
    st.warning(f"Hay {len(estado)} archivos de datos desincronizados con el remote de DVC")
else:
    st.success("Todos los datos están sincronizados con DVC")
```

Un dashboard interno de "salud del pipeline de datos" puede combinar el estado de DVC (ver `DVC/02 - Versionado de Datos - Comandos Fundamentales.md`) con métricas de MLflow para dar una vista unificada de todo el pipeline de ML a un equipo, sin que cada persona tenga que ejecutar comandos de CLI por separado.

## Optuna — visualizar una búsqueda de hiperparámetros en vivo

```python
import optuna
import optuna.visualization as vis

@st.cache_data(ttl=60)
def cargar_estudio():
    study = optuna.load_study(study_name="tuning", storage="postgresql://...")
    return study.trials_dataframe()

df_trials = cargar_estudio()
st.dataframe(df_trials.sort_values("value").head(10))

study = optuna.load_study(study_name="tuning", storage="postgresql://...")
fig = vis.plot_optimization_history(study)
st.plotly_chart(fig, use_container_width=True)
```

Ver `Optuna/07 - Visualización y Análisis de Resultados.md` — las gráficas de `optuna.visualization` están construidas sobre Plotly, por lo que se integran directamente con `st.plotly_chart` sin conversión adicional.

## Pandera — mostrar resultados de validación de datos

Ya cubierto en [[11 - Integración con Modelos de ML]] con el patrón de mostrar `e.failure_cases` como una tabla cuando la validación falla — el mismo patrón aplica a cualquier etapa del pipeline donde se quiera exponer visualmente el resultado de una validación (ver `Pandera/06 - Manejo de Errores y Validación Perezosa (Lazy).md`).

## APIs externas — llamadas HTTP dentro de la app

```python
import requests

@st.cache_data(ttl=300)
def consultar_api_clima(ciudad):
    respuesta = requests.get(f"https://api.clima.com/v1/{ciudad}", timeout=5)
    respuesta.raise_for_status()
    return respuesta.json()

datos_clima = consultar_api_clima("Santo Domingo")
st.json(datos_clima)
```

Cachear llamadas a APIs externas con `st.cache_data(ttl=...)` es especialmente importante — sin cache, cada rerun del script repetiría la llamada HTTP, multiplicando innecesariamente la carga sobre el servicio externo y ralentizando la app.

## Streamlit dentro de un notebook — `streamlit run` vs. exploración interactiva

Streamlit está diseñado para ejecutarse como script (`streamlit run app.py`), no dentro de un notebook Jupyter de forma nativa — para exploración de datos interactiva dentro de notebooks, herramientas como `ipywidgets` o Plotly directamente son más apropiadas; Streamlit entra en juego cuando el objetivo es una **aplicación** compartible, no una celda interactiva de exploración personal.

## Ver también

- [[11 - Integración con Modelos de ML]]
- `MLflow/04 - Tracking - Búsqueda, Comparación y Organización.md`
- `DVC/02 - Versionado de Datos - Comandos Fundamentales.md`
- `Optuna/07 - Visualización y Análisis de Resultados.md`
