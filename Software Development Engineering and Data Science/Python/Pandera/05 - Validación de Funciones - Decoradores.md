---
tags: [pandera, testing, validacion-de-datos, decoradores, cheat-sheet]
---

# 05 — Validación de Funciones: Decoradores

> Continúa de [[04 - Checks en Profundidad]].

Más allá de validar un DataFrame de forma aislada con `schema.validate(df)`, Pandera puede validar **directamente los inputs/outputs de una función** mediante decoradores — útil para garantizar que cada paso de un pipeline de transformación reciba y produzca exactamente lo esperado, sin tener que llamar `.validate()` manualmente al inicio y final de cada función.

## `@check_input` — validar un argumento de la función

```python
import pandera as pa

schema_entrada = pa.DataFrameSchema({
    "office_id": pa.Column(int, pa.Check.greater_than(0)),
    "total_demand": pa.Column(float, pa.Check.greater_than_or_equal_to(0)),
})

@pa.check_input(schema_entrada, "df")   # "df" es el nombre del parámetro a validar
def calcular_demanda_por_hora(df: pd.DataFrame) -> pd.DataFrame:
    df["demanda_por_hora"] = df["total_demand"] / 24
    return df
```

Si `df` no cumple `schema_entrada` al llamar la función, `SchemaError` se lanza **antes** de que el cuerpo de la función se ejecute — evita que código con datos inválidos llegue a ejecutarse y produzca resultados corruptos silenciosamente.

```python
# Validar por posición en vez de por nombre:
@pa.check_input(schema_entrada, 0)   # valida el primer argumento posicional
def procesar(df):
    ...
```

## `@check_output` — validar el valor de retorno

```python
schema_salida = pa.DataFrameSchema({
    "office_id": pa.Column(int),
    "total_demand": pa.Column(float),
    "demanda_por_hora": pa.Column(float, pa.Check.greater_than_or_equal_to(0)),
})

@pa.check_output(schema_salida)
def calcular_demanda_por_hora(df: pd.DataFrame) -> pd.DataFrame:
    df["demanda_por_hora"] = df["total_demand"] / 24
    return df
```

Garantiza que la función realmente produce lo que promete — útil para detectar bugs en la lógica de transformación (ej. una división que introduce `NaN` inesperados) inmediatamente donde ocurren, no varios pasos después en el pipeline.

### Validar un elemento específico de una tupla retornada

```python
@pa.check_output(schema_salida, obj_getter=0)   # si la función retorna (df, metadata), valida solo df
def procesar_y_reportar(df):
    resultado = transformar(df)
    return resultado, {"filas_procesadas": len(resultado)}
```

## `@check_io` — validar entrada y salida en un solo decorador

```python
@pa.check_io(df=schema_entrada, out=schema_salida)
def calcular_demanda_por_hora(df: pd.DataFrame) -> pd.DataFrame:
    df["demanda_por_hora"] = df["total_demand"] / 24
    return df
```

Combina `check_input` + `check_output` — la forma más compacta de blindar completamente una función de transformación, garantizando el "contrato de datos" completo (qué entra, qué sale) en una sola declaración.

## El patrón recomendado: cada etapa del pipeline con su propio contrato

```python
@pa.check_io(df=schema_datos_crudos, out=schema_datos_limpios)
def limpiar_datos(df: pd.DataFrame) -> pd.DataFrame:
    df = df.dropna(subset=["office_id"])
    df = df[df["total_demand"] >= 0]
    return df

@pa.check_io(df=schema_datos_limpios, out=schema_features)
def generar_features(df: pd.DataFrame) -> pd.DataFrame:
    df["demanda_por_hora"] = df["total_demand"] / 24
    return df

@pa.check_io(df=schema_features, out=schema_predicciones)
def predecir(df: pd.DataFrame) -> pd.DataFrame:
    df["prediccion"] = modelo.predict(df[columnas_features])
    return df
```

Encadenar funciones así hace que **cada transición entre etapas del pipeline sea un punto de verificación** — si algo corrompe los datos en cualquier etapa intermedia, el error se detecta inmediatamente en esa etapa específica, con un mensaje claro de qué esquema falló, en vez de propagarse silenciosamente hasta la predicción final.

## Usar `DataFrame[Schema]` como type hint — validación implícita con `DataFrameModel`

```python
from pandera.typing import DataFrame

class DatosCrudos(pa.DataFrameModel):
    office_id: pa.typing.Series[int]
    total_demand: pa.typing.Series[float]

class DatosLimpios(DatosCrudos):
    demanda_por_hora: pa.typing.Series[float]

@pa.check_types   # habilita la validación automática basada en los type hints DataFrame[...]
def generar_features(df: DataFrame[DatosCrudos]) -> DataFrame[DatosLimpios]:
    df["demanda_por_hora"] = df["total_demand"] / 24
    return df
```

Con `DataFrameModel` (ver [[03 - DataFrameModel - API Basada en Clases]]) y el decorador `@pa.check_types`, los propios type hints de la función (`DataFrame[DatosCrudos]` → `DataFrame[DatosLimpios]`) **son** la declaración de validación — el código queda más legible porque el contrato de datos es visualmente parte de la firma de la función, no una declaración separada como en `@check_io`.

## Desactivar validación en producción por rendimiento (si es necesario)

```python
import pandera as pa

pa.reset_config()   # restaura la config por defecto

# Desactivar validación globalmente (ej. tras probar exhaustivamente en staging):
import os
os.environ["PANDERA_VALIDATION_ENABLED"] = "False"
```

Ver [[10 - Buenas Prácticas y Pandera en Producción]] para la discusión completa de cuándo (y si) vale la pena desactivar validaciones en producción por consideraciones de rendimiento.

## Ver también

- [[03 - DataFrameModel - API Basada en Clases]]
- [[06 - Manejo de Errores y Validación Perezosa (Lazy)]]
- [[10 - Buenas Prácticas y Pandera en Producción]]
