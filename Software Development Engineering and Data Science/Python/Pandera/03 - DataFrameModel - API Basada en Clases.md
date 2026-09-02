---
tags: [pandera, testing, validacion-de-datos, cheat-sheet]
---

# 03 — DataFrameModel: API Basada en Clases

> Continúa de [[02 - DataFrameSchema - API Funcional]]. Estilo directamente inspirado en Pydantic.

## `DataFrameModel` — la sintaxis básica

```python
import pandera as pa
from pandera.typing import Series

class DemandSchema(pa.DataFrameModel):
    office_id: Series[int] = pa.Field(gt=0)
    total_demand: Series[float] = pa.Field(in_range={"min_value": 0, "max_value": 100_000})
    region: Series[str] = pa.Field(isin=["RD", "PR", "MX"])
    interval_start: Series["datetime64[ns]"]

    class Config:
        strict = True
        coerce = True
```

```python
df_validado = DemandSchema.validate(df)
```

Es funcionalmente equivalente a `DataFrameSchema` (ver [[02 - DataFrameSchema - API Funcional]]) — la diferencia es puramente de estilo: columnas como atributos de clase con anotación de tipo, en vez de entradas de un diccionario. Esto habilita autocompletado de IDE sobre los nombres de columna y es más natural para equipos que ya usan Pydantic extensivamente en el resto del proyecto.

## `Field` — el equivalente de `Column`

```python
class Schema(pa.DataFrameModel):
    edad: Series[int] = pa.Field(ge=0, le=120)                    # greater/less than or equal
    nombre: Series[str] = pa.Field(str_length={"min_value": 1})
    email: Series[str] = pa.Field(str_matches=r"^[^@]+@[^@]+\.[^@]+$")
    categoria: Series[str] = pa.Field(isin=["A", "B", "C"])
    nulos_permitidos: Series[float] = pa.Field(nullable=True)
    unico: Series[int] = pa.Field(unique=True)
```

| Argumento de `Field` | Equivalente en `Check` de la API funcional |
|---|---|
| `gt`, `ge`, `lt`, `le` | `Check.greater_than`, `Check.greater_than_or_equal_to`, etc. |
| `in_range` | `Check.in_range` |
| `isin` | `Check.isin` |
| `str_length` | `Check.str_length` |
| `str_matches` | `Check.str_matches` |
| `nullable`, `unique`, `coerce` | Argumentos directos de `Column` |

## `class Config` — configuración a nivel de esquema completo

```python
class DemandSchema(pa.DataFrameModel):
    office_id: Series[int]
    total_demand: Series[float]

    class Config:
        strict = True                 # equivalente a strict= en DataFrameSchema
        coerce = True
        ordered = False
        unique = ["office_id"]
        name = "DemandSchema"
```

## Herencia de esquemas — reutilización entre esquemas relacionados

```python
class BaseSchema(pa.DataFrameModel):
    office_id: Series[int] = pa.Field(gt=0)
    fecha: Series["datetime64[ns]"]

class DemandSchema(BaseSchema):
    total_demand: Series[float] = pa.Field(ge=0)   # hereda office_id y fecha, agrega total_demand

class ForecastSchema(BaseSchema):
    prediccion: Series[float] = pa.Field(ge=0)
    intervalo_confianza: Series[float] = pa.Field(ge=0, le=1)
```

Este patrón resuelve exactamente el mismo problema que `add_columns`/`remove_columns` en la API funcional (ver [[02 - DataFrameSchema - API Funcional]]), pero con sintaxis de herencia de clases estándar de Python — natural cuando varios esquemas comparten una base común (ej. "todo DataFrame en este pipeline tiene `office_id` y `fecha`").

## Checks a nivel de esquema completo — método decorado

```python
class DemandSchema(pa.DataFrameModel):
    demanda_predicha: Series[float]
    demanda_real: Series[float]

    @pa.dataframe_check
    def error_dentro_de_rango(cls, df: pd.DataFrame) -> Series[bool]:
        error_relativo = abs(df["demanda_predicha"] - df["demanda_real"]) / df["demanda_real"]
        return error_relativo < 0.5   # el error no debería superar el 50%
```

Equivalente al concepto de "wide checks" de la API funcional (ver [[04 - Checks en Profundidad]]) — checks que involucran la relación entre **múltiples columnas**, no una columna aislada. El decorador `@pa.dataframe_check` marca el método como un check que recibe el DataFrame completo.

## Checks a nivel de columna, definidos como método

```python
class Schema(pa.DataFrameModel):
    precio: Series[float]

    @pa.check("precio")
    def precio_razonable(cls, serie: Series[float]) -> Series[bool]:
        return serie.between(0, 1_000_000)
```

Alternativa a pasar el check directamente en `Field(...)`, útil cuando la lógica de validación es más compleja que lo que cabe naturalmente en un argumento de función.

## Convertir entre `DataFrameModel` y `DataFrameSchema`

```python
schema_funcional = DemandSchema.to_schema()   # obtiene el DataFrameSchema equivalente

# También sirve para inspeccionar el esquema generado, o para usarlo donde se espera un DataFrameSchema explícito
```

Ambas APIs son intercambiables en tiempo de ejecución — `DataFrameModel` es, internamente, syntactic sugar que se compila a un `DataFrameSchema` — por lo que cualquier funcionalidad disponible en uno está disponible en el otro.

## Usar `DataFrameModel` como type hint de función — integración con `pandera.typing`

```python
from pandera.typing import DataFrame

def calcular_totales(df: DataFrame[DemandSchema]) -> DataFrame[DemandSchema]:
    df["total_demand"] = df["total_demand"].round(2)
    return df
```

`DataFrame[DemandSchema]` es un type hint genérico que documenta (y, combinado con los decoradores de [[05 - Validación de Funciones - Decoradores]], **valida en runtime**) qué esquema se espera como entrada/salida de una función — el mismo patrón que Pydantic aplica a modelos de datos, aplicado aquí a DataFrames completos.

## Cuándo preferir `DataFrameModel` sobre `DataFrameSchema`

| Situación | API recomendada |
|---|---|
| El proyecto ya usa Pydantic extensivamente | `DataFrameModel` |
| Se necesita autocompletado de IDE sobre nombres de columna | `DataFrameModel` |
| El esquema se genera/modifica dinámicamente en tiempo de ejecución | `DataFrameSchema` |
| Se necesita composición ad-hoc (`add_columns`/`remove_columns`) frecuente | `DataFrameSchema` |
| Se integra con FastAPI (ver [[09 - Integración con el Ecosistema]]) | `DataFrameModel` (más natural con Pydantic) |

## Ver también

- [[02 - DataFrameSchema - API Funcional]]
- [[04 - Checks en Profundidad]]
- [[05 - Validación de Funciones - Decoradores]]
- [[09 - Integración con el Ecosistema]]
