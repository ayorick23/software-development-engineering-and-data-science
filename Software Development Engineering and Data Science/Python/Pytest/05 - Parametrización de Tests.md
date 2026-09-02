---
tags: [pytest, python, testing, parametrize, cheat-sheet]
---

# 05 — Parametrización de Tests

> Continúa de [[04 - Fixtures Avanzadas]].

## `@pytest.mark.parametrize` — un test, múltiples casos

```python
import pytest

@pytest.mark.parametrize("a, b, esperado", [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0),
    (100, 200, 300),
])
def test_suma(a, b, esperado):
    assert sumar(a, b) == esperado
```

Esto ejecuta la **misma** función de test 4 veces, una por cada tupla en la lista — evita escribir 4 funciones casi idénticas (`test_suma_positivos`, `test_suma_ceros`, `test_suma_negativos`...) que solo difieren en los valores de entrada/salida esperados.

```bash
pytest -v
# test_suma.py::test_suma[2-3-5] PASSED
# test_suma.py::test_suma[0-0-0] PASSED
# test_suma.py::test_suma[-1-1-0] PASSED
# test_suma.py::test_suma[100-200-300] PASSED
```

Cada combinación aparece como un test **individual** en la salida (con los valores entre corchetes) — si una combinación específica falla, se sabe exactamente cuál sin tener que inspeccionar un solo test genérico que agrupe varios casos con un loop manual.

## IDs personalizados para casos más legibles

```python
@pytest.mark.parametrize("entrada, esperado", [
    pytest.param("", False, id="string_vacio"),
    pytest.param("hola", True, id="string_normal"),
    pytest.param(None, False, id="valor_none"),
])
def test_es_valido(entrada, esperado):
    assert es_valido(entrada) == esperado
```

`pytest.param(..., id="...")` reemplaza el ID generado automáticamente (que puede ser difícil de leer con valores complejos) por un nombre descriptivo — el test aparece como `test_es_valido[string_vacio]` en vez de algo menos claro.

## Marcar un caso específico dentro de una parametrización

```python
@pytest.mark.parametrize("valor, esperado", [
    (10, 100),
    (0, 0),
    pytest.param(-5, 25, marks=pytest.mark.xfail(reason="bug conocido con negativos")),   # este caso se espera que falle
])
def test_cuadrado(valor, esperado):
    assert cuadrado(valor) == esperado
```

`pytest.param(..., marks=...)` permite aplicar un mark (ver [[06 - Marks y Selección de Tests]]) a un caso **individual** dentro de una lista parametrizada, sin afectar a los demás casos de la misma función de test.

## Apilar múltiples `parametrize` — producto cartesiano

```python
@pytest.mark.parametrize("region", ["Norte", "Sur"])
@pytest.mark.parametrize("canal", ["Online", "Tienda"])
def test_combinacion(region, canal):
    ...     # se ejecuta 4 veces: (Norte,Online), (Norte,Tienda), (Sur,Online), (Sur,Tienda)
```

Dos decoradores `@pytest.mark.parametrize` apilados generan el **producto cartesiano** de ambas listas — útil para probar todas las combinaciones de dos dimensiones independientes sin escribir una lista de tuplas manualmente combinada.

## Parametrizar con datos generados dinámicamente

```python
def generar_casos():
    return [(i, i ** 2) for i in range(5)]

@pytest.mark.parametrize("valor, esperado", generar_casos())
def test_cuadrado(valor, esperado):
    assert cuadrado(valor) == esperado
```

Los valores de `parametrize` pueden venir de cualquier función Python normal que devuelva una lista/iterable de tuplas — no tienen que estar escritos literalmente en el decorador.

## Parametrizar con `indirect=True` — pasar el parámetro a través de una fixture

```python
@pytest.fixture
def conexion(request):
    return conectar(request.param)     # recibe el valor a través de request.param, no directamente

@pytest.mark.parametrize("conexion", ["sqlite", "postgres"], indirect=True)
def test_query(conexion):
    assert conexion.ejecutar("SELECT 1") is not None
```

`indirect=True` redirige el valor parametrizado hacia una fixture (vía `request.param`) en vez de pasarlo directo al test — útil cuando el valor "crudo" (`"sqlite"`) necesita procesamiento (conectar, inicializar) antes de llegar al test, combinando parametrización con el patrón de fixtures parametrizadas visto en [[04 - Fixtures Avanzadas#Fixtures parametrizadas|Fixtures Avanzadas]].

## Ver también

- [[04 - Fixtures Avanzadas]]
- [[06 - Marks y Selección de Tests]]
- [[15 - Property-Based Testing con Hypothesis]]
