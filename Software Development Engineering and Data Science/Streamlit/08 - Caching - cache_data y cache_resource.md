---
tags: [streamlit, dashboards, caching, rendimiento, cheat-sheet]
---

# 08 — Caching: cache_data y cache_resource

> Continúa de [[07 - Session State - Estado entre Reruns]]. Fundamental por el modelo de re-ejecución completa cubierto en [[01 - Introducción y Modelo de Ejecución]].

## El problema: todo se recalcula en cada rerun

```python
def cargar_datos():
    return pd.read_csv("dataset_gigante.csv")   # sin cache, esto se re-lee en CADA interacción del usuario

df = cargar_datos()   # lento, y absurdamente repetitivo si nada relevante cambió
```

Dado el modelo de ejecución de Streamlit (ver [[01 - Introducción y Modelo de Ejecución]]), sin cache explícito, cargar un dataset de 500MB o entrenar un modelo se repetiría cada vez que el usuario mueve un slider — completamente impracticable para cualquier operación no trivial.

## `st.cache_data` — para datos (DataFrames, listas, dicts, resultados serializables)

```python
@st.cache_data
def cargar_datos():
    return pd.read_csv("dataset_gigante.csv")

df = cargar_datos()   # la PRIMERA vez, ejecuta la función y guarda el resultado
                        # en llamadas posteriores CON LOS MISMOS ARGUMENTOS, retorna el resultado guardado sin recalcular
```

`st.cache_data` está pensado para funciones que retornan **datos** — internamente, cada vez que se retorna un valor cacheado, Streamlit hace una **copia** del objeto guardado, para evitar que una parte del código modifique accidentalmente el objeto cacheado compartido (mutación accidental de un DataFrame cacheado sería un bug muy difícil de rastrear sin esta protección).

## `st.cache_resource` — para recursos (modelos, conexiones, objetos no serializables)

```python
@st.cache_resource
def cargar_modelo():
    return joblib.load("modelo_produccion.joblib")

modelo = cargar_modelo()   # el modelo se carga UNA sola vez, compartido entre todas las sesiones de usuarios
```

```python
@st.cache_resource
def conectar_base_de_datos():
    return psycopg2.connect(host="...", dbname="...", user="...", password="...")

conexion = conectar_base_de_datos()
```

`st.cache_resource` **no copia** el objeto retornado (a diferencia de `cache_data`) — retorna la **misma instancia** en cada llamada, lo cual es exactamente lo que se necesita para objetos como conexiones a bases de datos o modelos cargados en memoria, que no tendría sentido (ni sería posible, en el caso de una conexión) duplicar.

## La diferencia central — tabla de decisión

| | `st.cache_data` | `st.cache_resource` |
|---|---|---|
| Para qué | DataFrames, arrays, listas, resultados de cómputo | Modelos de ML, conexiones a BD, clientes de API |
| Comportamiento al retornar | Copia el objeto cacheado | Retorna la MISMA instancia (sin copiar) |
| Ejemplo típico | `pd.read_csv(...)`, resultado de una agregación | `joblib.load(...)`, `mlflow.pyfunc.load_model(...)`, conexión de base de datos |
| Serializable | Debe poder copiarse/serializarse razonablemente | No necesita ser serializable |

> **Error común**: usar `st.cache_data` sobre una función que retorna una conexión de base de datos o un modelo de PyTorch/TensorFlow — muchos de estos objetos no se pueden copiar de forma segura (o directamente lanzan una excepción al intentarlo), y es exactamente el caso para el que existe `st.cache_resource`.

## Invalidación por argumentos — el cache es sensible a los parámetros

```python
@st.cache_data
def cargar_datos_filtrados(region: str, dias: int):
    df = pd.read_csv("dataset.csv")
    return df[(df["region"] == region)].tail(dias)

df_rd_90 = cargar_datos_filtrados("RD", 90)   # ejecuta y cachea
df_rd_90_otra_vez = cargar_datos_filtrados("RD", 90)   # usa el cache — MISMOS argumentos
df_pr_90 = cargar_datos_filtrados("PR", 90)   # ejecuta de NUEVO — argumentos DISTINTOS, cache distinto
```

Streamlit usa un hash de los argumentos de la función como clave de cache — cada combinación distinta de argumentos tiene su propia entrada cacheada independiente.

## `ttl` — expiración por tiempo

```python
@st.cache_data(ttl=3600)   # el cache expira después de 1 hora, forzando una re-ejecución
def obtener_datos_actualizados():
    return consultar_api_externa()
```

Útil para datos que cambian con el tiempo (precios, métricas en vivo) donde se quiere balancear rendimiento (no consultar la fuente en cada rerun) con frescura (no servir datos de hace una semana indefinidamente).

## `max_entries` — límite de entradas cacheadas

```python
@st.cache_data(max_entries=100)   # solo conserva las 100 combinaciones de argumentos más recientes
def procesar(parametro):
    ...
```

Evita que el cache crezca indefinidamente en memoria cuando una función se llama con muchísimas combinaciones distintas de argumentos a lo largo del tiempo (por ejemplo, si un argumento incluye un timestamp único por sesión).

## Parámetros que NO deben afectar el hash — `_` como prefijo

```python
@st.cache_data
def procesar_datos(df: pd.DataFrame, _conexion):
    # el guión bajo antes de "_conexion" le dice a Streamlit: NO intentes hashear este argumento
    return df.merge(_conexion.query("SELECT * FROM tabla"))
```

Argumentos que empiezan con `_` se excluyen del cálculo de hash que determina si dos llamadas son "la misma" — necesario para objetos que no se pueden (o no tiene sentido) hashear, como conexiones de base de datos pasadas como parámetro.

## Invalidar el cache manualmente

```python
cargar_datos.clear()   # limpia el cache de ESA función específica

st.cache_data.clear()      # limpia TODO el cache de cache_data de la app
st.cache_resource.clear()   # limpia TODO el cache de cache_resource
```

```python
if st.button("Recargar datos"):
    cargar_datos.clear()
    st.rerun()   # fuerza un nuevo rerun, que ahora sí re-ejecutará la función sin cache
```

## Cachear la carga de un modelo de MLflow — patrón real

```python
import mlflow

@st.cache_resource
def cargar_modelo_produccion():
    mlflow.set_tracking_uri("http://mlflow-server:5000")
    return mlflow.pyfunc.load_model("models:/claro-rd-demand-model@champion")

modelo = cargar_modelo_produccion()   # se carga UNA vez, se reutiliza en todos los reruns/usuarios
predicciones = modelo.predict(df_nuevo)
```

Ver [[11 - Integración con Modelos de ML]] para el patrón completo de servir un modelo de MLflow desde una app de Streamlit.

## Ver también

- [[01 - Introducción y Modelo de Ejecución]]
- [[07 - Session State - Estado entre Reruns]]
- [[11 - Integración con Modelos de ML]]
- [[14 - Buenas Prácticas, Rendimiento y Testing]]
