---
tags: [pandera, testing, validacion-de-datos, coercion, cheat-sheet]
---

# 07 — Coerción, Tipos de Datos y Filtrado de Filas Inválidas

> Continúa de [[06 - Manejo de Errores y Validación Perezosa (Lazy)]].

## `coerce` — forzar la conversión de tipos en vez de solo validarlos

Por defecto, Pandera **verifica** que una columna ya tenga el tipo declarado — si no coincide exactamente, falla. `coerce=True` cambia ese comportamiento: intenta **convertir** la columna al tipo declarado antes de validar.

```python
schema = pa.DataFrameSchema({
    "office_id": pa.Column(int, coerce=True),       # convierte a int si llega como string/float
    "total_demand": pa.Column(float, coerce=True),
})

df = pd.DataFrame({"office_id": ["1", "2", "3"], "total_demand": [10, 20, 30]})   # tipos "incorrectos" al inicio
df_validado = schema.validate(df)
print(df_validado.dtypes)   # office_id: int64, total_demand: float64 — ya convertidos
```

Sin `coerce=True`, este mismo DataFrame lanzaría `SchemaError` porque `office_id` llega como `object` (strings), no como `int`. Con `coerce=True`, Pandera intenta la conversión (`.astype()`) automáticamente, y solo falla si la conversión en sí es imposible (por ejemplo, un valor `"abc"` no puede convertirse a `int`).

```python
schema = pa.DataFrameSchema({...}, coerce=True)   # aplica coerce a NIVEL DE ESQUEMA COMPLETO, todas las columnas
```

> **Cuándo usar `coerce` y cuándo no**: `coerce=True` es conveniente cuando los datos llegan de una fuente que no garantiza tipos estrictos (CSV, APIs externas, Excel) — pero puede **enmascarar problemas reales**: si `office_id` llega como `"3.5"`, forzar la conversión a `int` puede truncar silenciosamente el decimal en vez de alertar que ese valor nunca debió existir. Usar `coerce` deliberadamente, no como reflejo automático para "hacer que la validación pase".

## Tipos de datos soportados

```python
pa.Column(int)                    # tipo Python nativo
pa.Column("int64")                 # string de dtype de pandas/numpy
pa.Column(pd.Int64Dtype())          # dtype específico (nullable integer de pandas)
pa.Column("datetime64[ns]")          # fechas
pa.Column("category")                 # categóricas de pandas
pa.Column(str)                          # texto
pa.Column(bool)
```

### Tipos nullable de pandas — distinción importante

```python
pa.Column(int, nullable=False)              # int estándar de NumPy — NO puede tener NaN de por sí
pa.Column(pd.Int64Dtype(), nullable=True)   # Int64 (con mayúscula) de pandas — SÍ soporta NaN, a diferencia de int64 de NumPy
```

Un detalle técnico que confunde con frecuencia: el tipo `int64` nativo de NumPy no puede representar `NaN` (NumPy fuerza la columna completa a `float64` si hay algún nulo) — para tener una columna verdaderamente entera que además permita nulos, se necesita el tipo nullable `Int64` de pandas (con mayúscula), no `int64` de NumPy. Pandera valida esta distinción explícitamente si se declara `nullable=True` sobre un tipo que no la soporta.

## `drop_invalid_rows` — filtrar en vez de fallar todo el DataFrame

```python
schema = pa.DataFrameSchema(
    {"total_demand": pa.Column(float, pa.Check.greater_than_or_equal_to(0))},
    drop_invalid_rows=True,
)

df_resultado = schema.validate(df, lazy=True)   # requiere lazy=True para funcionar
# las filas que NO cumplían el check simplemente se ELIMINAN del resultado, sin lanzar excepción
```

En vez de que **todo** el DataFrame falle por unas pocas filas problemáticas, `drop_invalid_rows=True` conserva las filas válidas y descarta silenciosamente las inválidas — apropiado cuando el negocio prefiere "procesar lo que se pueda, ignorar lo corrupto" en vez de bloquear el pipeline completo por un pequeño porcentaje de filas malas.

> **Precaución**: `drop_invalid_rows=True` descarta datos sin registro explícito por defecto — combinarlo siempre con logging de qué se descartó (ver el patrón de `failure_cases` en [[06 - Manejo de Errores y Validación Perezosa (Lazy)]]) para no perder visibilidad de cuántos datos se están silenciosamente ignorando, lo cual podría ocultar un problema sistemático más serio (ej. una fuente de datos que empezó a enviar 30% de filas corruptas).

## Filtrado manual usando `failure_cases` — más control que `drop_invalid_rows`

```python
def validar_y_filtrar(df, schema):
    try:
        return schema.validate(df, lazy=True), 0
    except pa.errors.SchemaErrors as e:
        indices_invalidos = e.failure_cases["index"].dropna().unique()
        df_limpio = df.drop(index=indices_invalidos)
        df_validado = schema.validate(df_limpio)   # revalida el subconjunto limpio, debería pasar ahora
        return df_validado, len(indices_invalidos)

df_final, n_descartadas = validar_y_filtrar(df, schema)
if n_descartadas > 0:
    logger.warning("Se descartaron %d filas por fallar validación de datos", n_descartadas)
```

Este patrón da control explícito y logging obligatorio del filtrado — preferible a `drop_invalid_rows=True` silencioso en pipelines de producción donde la cantidad de datos descartados es en sí misma una señal de calidad que vale la pena monitorear.

## Validar dtypes sin checks de valores — solo estructura

```python
schema = pa.DataFrameSchema({
    "office_id": pa.Column(int),          # sin Check — solo valida que el TIPO sea correcto
    "total_demand": pa.Column(float),
})
```

Útil como primera capa de validación rápida (¿la estructura básica es correcta?) antes de aplicar checks de negocio más costosos — permite fallar rápido ante problemas estructurales graves (columna faltante, tipo completamente incorrecto) sin gastar tiempo evaluando reglas de negocio sobre datos ya fundamentalmente rotos.

## Ver también

- [[02 - DataFrameSchema - API Funcional]]
- [[06 - Manejo de Errores y Validación Perezosa (Lazy)]]
- [[10 - Buenas Prácticas y Pandera en Producción]]
