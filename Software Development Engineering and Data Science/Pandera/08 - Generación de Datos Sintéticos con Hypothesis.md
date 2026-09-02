---
tags: [pandera, testing, hypothesis, property-based-testing, cheat-sheet]
---

# 08 — Generación de Datos Sintéticos con Hypothesis

> Continúa de [[07 - Coerción, Tipos de Datos y Filtrado de Filas Inválidas]]. Complementa `Machine Learning/25-Testing-Avanzado.md` (testing basado en propiedades).

## El problema: escribir DataFrames de prueba a mano es tedioso y limitado

Escribir manualmente casos de prueba (`pd.DataFrame({"office_id": [1, 2, 3], ...})`) para cada función que consume un DataFrame cubre solo los casos que el desarrollador pensó de antemano — deja sin probar combinaciones de valores extremos, nulos en posiciones inesperadas, o edge cases que un humano no anticiparía.

## La idea central: el esquema YA describe qué datos son válidos

Un `DataFrameSchema` o `DataFrameModel` ya contiene toda la información necesaria para **generar** datos que lo cumplen (o que lo violan deliberadamente) — Pandera integra la librería `hypothesis` para esto, evitando escribir generadores de datos de prueba a mano.

```bash
pip install "pandera[hypotheses]"
```

## `schema.example()` — generar un único DataFrame de ejemplo

```python
import pandera as pa

schema = pa.DataFrameSchema({
    "office_id": pa.Column(int, pa.Check.greater_than(0)),
    "total_demand": pa.Column(float, pa.Check.in_range(0, 100_000)),
})

df_ejemplo = schema.example(size=10)   # genera un DataFrame de 10 filas que CUMPLE el esquema
print(df_ejemplo)
```

Útil para prototipar rápidamente contra una función sin tener que escribir datos de prueba manualmente, o para generar fixtures de ejemplo en documentación.

## `schema.strategy()` — integración directa con `hypothesis` para property-based testing

```python
from hypothesis import given
import pandera as pa

schema = pa.DataFrameSchema({
    "office_id": pa.Column(int, pa.Check.greater_than(0)),
    "total_demand": pa.Column(float, pa.Check.greater_than_or_equal_to(0)),
})

@given(schema.strategy(size=20))
def test_calcular_demanda_por_hora_nunca_es_negativa(df):
    resultado = calcular_demanda_por_hora(df)
    assert (resultado["demanda_por_hora"] >= 0).all()
```

Este es el patrón de **property-based testing**: en vez de escribir casos de prueba específicos, se declara una **propiedad** que debe cumplirse para *cualquier* dato válido según el esquema, y `hypothesis` genera automáticamente decenas de combinaciones distintas (incluyendo casos extremos: valores muy grandes, muy pequeños, cercanos a los límites de los `Check`) para intentar encontrar un contraejemplo que rompa la propiedad. Ver `Machine Learning/25-Testing-Avanzado.md` para la teoría general de property-based testing con `hypothesis` fuera del contexto de Pandera.

## Generar datos que violan el esquema deliberadamente — testear el manejo de errores

```python
from pandera.strategies import pandas_dtype_strategy

# Generar datos INVÁLIDos a propósito, para probar que la función de validación los rechaza correctamente:
df_invalido = pd.DataFrame({
    "office_id": [-1, -2, -3],   # viola greater_than(0) intencionalmente
    "total_demand": [10.0, 20.0, 30.0],
})

def test_schema_rechaza_office_id_negativo():
    with pytest.raises(pa.errors.SchemaError):
        schema.validate(df_invalido)
```

Aunque este ejemplo específico usa datos escritos a mano (más simple para un caso puntual), el mismo principio de "generar datos que rompen una regla específica" se puede combinar con estrategias de `hypothesis` para explorar sistemáticamente el espacio de violaciones posibles, no solo un ejemplo fijo.

## Control fino sobre la generación — tamaño y semilla

```python
df_ejemplo = schema.example(size=100)

# Reproducibilidad de los datos generados, para debugging de un fallo específico:
from hypothesis import seed

@seed(42)
@given(schema.strategy(size=20))
def test_propiedad(df):
    ...
```

## Combinar `schema.strategy()` con `@given` de múltiples esquemas

```python
schema_input = pa.DataFrameSchema({"cantidad": pa.Column(int, pa.Check.greater_than(0))})
schema_precios = pa.DataFrameSchema({"precio": pa.Column(float, pa.Check.greater_than(0))})

@given(schema_input.strategy(size=10), schema_precios.strategy(size=10))
def test_calculo_total(df_cantidad, df_precio):
    # hypothesis genera combinaciones de AMBOS DataFrames simultáneamente
    ...
```

## Cuándo vale la pena esta técnica

Property-based testing con Pandera+Hypothesis aporta más valor en:
- Funciones de transformación con lógica matemática/estadística donde los edge cases son difíciles de anticipar manualmente (divisiones, normalizaciones, agregaciones).
- Pipelines donde la robustez frente a datos "raros pero válidos" importa (un `office_id` extremadamente grande, una fecha en el límite de un rango).
- Verificar que el manejo de errores de un pipeline realmente atrapa los casos inválidos que se supone debe atrapar, no solo los casos obvios que un desarrollador ya probó a mano.

Para la mayoría de tests unitarios simples de lógica de negocio determinista, los tests con datos fijos (`pytest` estándar, ver `Machine Learning/13-Testing-en-Machine-Learning.md`) siguen siendo más simples de escribir y leer — property-based testing es una herramienta complementaria para los casos donde la robustez frente a un espacio amplio de inputs es el objetivo específico, no un reemplazo universal del testing tradicional.

## Ver también

- [[07 - Coerción, Tipos de Datos y Filtrado de Filas Inválidas]]
- `Machine Learning/25-Testing-Avanzado.md`
- `Machine Learning/13-Testing-en-Machine-Learning.md`
