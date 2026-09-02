---
tags: [pandera, testing, validacion-de-datos, cheat-sheet]
---

# 02 — DataFrameSchema: API Funcional

> Continúa de [[Python/Pandera/01 - Introducción y Conceptos Fundamentales]].

## `DataFrameSchema` — estructura general

```python
import pandera as pa

schema = pa.DataFrameSchema(
    columns={
        "office_id": pa.Column(int, pa.Check.greater_than(0)),
        "total_demand": pa.Column(float, pa.Check.in_range(0, 100_000), nullable=False),
        "region": pa.Column(str, pa.Check.isin(["RD", "PR", "MX"])),
    },
    index=pa.Index(int, pa.Check.greater_than_or_equal_to(0)),
    strict=True,       # rechaza columnas NO declaradas en el esquema
    coerce=True,        # fuerza la conversión de tipos en vez de solo validarlos (ver 07)
    ordered=False,       # si True, exige que las columnas aparezcan en el orden declarado
    unique=["office_id"],  # combinaciones de columnas que deben ser únicas en conjunto
)
```

## `Column` — anatomía completa

```python
pa.Column(
    dtype=float,                          # tipo esperado — puede ser tipo Python, string, o dtype de pandas/numpy
    checks=[pa.Check.greater_than(0)],     # uno o más checks, ver 04
    nullable=False,                         # si la columna puede contener NaN/None
    unique=False,                            # si los valores de ESTA columna deben ser únicos
    coerce=False,                             # forzar conversión a `dtype` si el tipo original no coincide
    required=True,                             # si la columna DEBE existir (False = opcional)
    name="total_demand",                        # normalmente se infiere de la clave del diccionario
    regex=False,                                  # si el nombre de columna es un patrón regex (ver más abajo)
    description="Demanda total del intervalo",     # documentación, aparece en schema.to_yaml()
    title="Demanda Total",
)
```

## `Index` y `MultiIndex`

```python
# Index simple
schema = pa.DataFrameSchema(
    columns={...},
    index=pa.Index(int, pa.Check.greater_than_or_equal_to(0), name="id"),
)

# MultiIndex — para DataFrames con índice jerárquico
schema = pa.DataFrameSchema(
    columns={...},
    index=pa.MultiIndex([
        pa.Index(str, name="region"),
        pa.Index("datetime64[ns]", name="fecha"),
    ]),
)
```

## Columnas con nombres dinámicos — `regex=True`

```python
schema = pa.DataFrameSchema({
    "producto_.*": pa.Column(float, pa.Check.greater_than_or_equal_to(0), regex=True),
})

df = pd.DataFrame({
    "producto_a": [10.0, 20.0],
    "producto_b": [5.0, 15.0],
    "producto_c": [8.0, 12.0],
})
schema.validate(df)   # válida las TRES columnas, todas coinciden con el patrón "producto_.*"
```

Útil para datasets con estructura wide donde el número exacto de columnas varía (ej. una columna por producto, cuyo catálogo cambia con el tiempo) — sin `regex=True`, habría que declarar cada columna explícitamente y actualizar el esquema cada vez que se agrega un producto nuevo.

## `strict` — controlar columnas no declaradas

```python
schema_estricto = pa.DataFrameSchema({"a": pa.Column(int)}, strict=True)
schema_estricto.validate(pd.DataFrame({"a": [1], "b": [2]}))
# SchemaError: column 'b' not in DataFrameSchema

schema_filtrante = pa.DataFrameSchema({"a": pa.Column(int)}, strict="filter")
resultado = schema_filtrante.validate(pd.DataFrame({"a": [1], "b": [2]}))
# NO lanza error — simplemente ELIMINA la columna "b" del resultado, se queda solo con "a"
```

`strict="filter"` es especialmente útil como primer paso de un pipeline que recibe datos de una fuente externa con columnas adicionales impredecibles — normaliza el DataFrame a exactamente el esquema esperado, descartando el resto sin fallar.

## `ordered` — exigir un orden específico de columnas

```python
schema = pa.DataFrameSchema(
    {"a": pa.Column(int), "b": pa.Column(int)},
    ordered=True,
)
schema.validate(pd.DataFrame({"b": [1], "a": [2]}))
# SchemaError: column order mismatch
```

Relevante cuando el orden de columnas importa para un consumidor posterior (ej. un archivo CSV exportado con un formato fijo esperado por otro sistema).

## Unicidad — a nivel de columna vs. combinación de columnas

```python
# Una sola columna debe tener valores únicos:
schema = pa.DataFrameSchema({"id": pa.Column(int, unique=True)})

# La COMBINACIÓN de varias columnas debe ser única (aunque cada una individualmente pueda repetirse):
schema = pa.DataFrameSchema(
    {"office_id": pa.Column(int), "fecha": pa.Column("datetime64[ns]")},
    unique=["office_id", "fecha"],   # no puede haber dos filas con la MISMA oficina Y MISMA fecha
)
```

## Modificar un esquema existente — composición

```python
schema_base = pa.DataFrameSchema({
    "office_id": pa.Column(int),
    "total_demand": pa.Column(float),
})

# Agregar columnas sin reescribir el esquema completo:
schema_extendido = schema_base.add_columns({
    "region": pa.Column(str, pa.Check.isin(["RD", "PR"])),
})

# Quitar columnas:
schema_reducido = schema_base.remove_columns(["total_demand"])

# Actualizar una columna existente:
schema_actualizado = schema_base.update_column("total_demand", checks=pa.Check.greater_than(0))
```

Este patrón de composición evita duplicar esquemas casi idénticos — por ejemplo, un esquema de "datos crudos" y otro de "datos procesados" que comparten la mayoría de columnas pero difieren en un par.

## Validar solo columnas, no todo el DataFrame

```python
schema_columna = pa.Column(float, pa.Check.greater_than(0), name="precio")
schema_columna.validate(df["precio"])   # valida una Serie individual, sin necesitar un DataFrameSchema completo
```

Útil para validar una columna aislada fuera del contexto de un DataFrame completo, por ejemplo, dentro de una función que recibe una `Series` como parámetro.

## Serializar un esquema — `to_yaml` / `to_script`

```python
print(schema.to_yaml())   # exporta el esquema como YAML legible — útil para documentación o versionado

codigo = schema.to_script()   # genera el código Python que reconstruye el esquema
```

Ver [[10 - Buenas Prácticas y Pandera en Producción]] para el patrón de versionar esquemas junto al código del pipeline.

## Ver también

- [[03 - DataFrameModel - API Basada en Clases]]
- [[04 - Checks en Profundidad]]
- [[07 - Coerción, Tipos de Datos y Filtrado de Filas Inválidas]]
