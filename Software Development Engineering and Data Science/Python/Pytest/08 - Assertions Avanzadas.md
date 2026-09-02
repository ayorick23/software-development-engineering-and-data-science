---
tags: [pytest, python, testing, assertions, cheat-sheet]
---

# 08 — Assertions Avanzadas

> Continúa de [[07 - Manejo de Excepciones y Warnings]].

## Cómo funciona la reescritura de aserciones (assert rewriting)

```python
def test_ejemplo():
    lista = [1, 2, 3]
    assert 4 in lista
```

```
E   assert 4 in [1, 2, 3]
```

Pytest **reescribe** el bytecode de cada `assert` al importar el módulo de test, insertando instrumentación que captura los valores intermedios de la expresión — así puede mostrar `assert 4 in [1, 2, 3]` en vez de solo "AssertionError" genérico, sin que el desarrollador tenga que llamar ningún método especial de comparación. Esto solo ocurre en archivos de test descubiertos por pytest, no en código de aplicación normal.

## Comparar números de punto flotante con `pytest.approx`

```python
def test_promedio():
    resultado = calcular_promedio([1.1, 2.2, 3.3])
    assert resultado == pytest.approx(2.2)                     # tolerancia relativa default (~1e-6)
    assert resultado == pytest.approx(2.2, rel=1e-3)             # tolerancia relativa explícita
    assert resultado == pytest.approx(2.2, abs=0.01)               # tolerancia absoluta en vez de relativa
```

Igual que `np.isclose()`/`np.allclose()` de NumPy (ver [[Python/NumPy/10 - Operaciones Matemáticas y Vectorización#Operadores de comparación (devuelven arrays booleanos)|Python/NumPy]]), comparar floats con `==` directo es propenso a fallos por errores de precisión — `pytest.approx()` es la forma idiomática de manejar esto dentro de una aserción de pytest.

```python
assert [1.0, 2.0, 3.0] == pytest.approx([1.001, 1.999, 3.0002], rel=1e-2)   # también funciona con listas/arrays completos
```

## Comparar estructuras de datos complejas

```python
def test_respuesta_api():
    respuesta = obtener_usuario(1)
    assert respuesta == {
        "id": 1,
        "nombre": "Ana",
        "activo": True,
    }
```

Pytest compara diccionarios/listas/sets anidados recursivamente con `==` normal, y al fallar muestra un **diff estructurado** señalando exactamente qué claves/valores difieren — no hace falta ninguna librería adicional para comparar JSON/diccionarios anidados, a diferencia de `unittest` donde a veces se recurre a `assertDictEqual` explícito.

## `assert all(...)` / `assert any(...)` sobre colecciones

```python
def test_todos_positivos():
    resultados = [calcular(x) for x in datos]
    assert all(r > 0 for r in resultados)

def test_al_menos_uno_valido():
    assert any(es_valido(item) for item in items)
```

**Cuidado con el mensaje de error:** `assert all(...)` con un generador solo reporta `assert False` sin indicar **cuál** elemento falló — para depuración más clara, a veces conviene un loop explícito con `assert` individual por elemento, o usar `pytest.mark.parametrize` (ver [[05 - Parametrización de Tests]]) para que cada caso sea un test separado con su propio resultado.

## Excepciones personalizadas con mensajes ricos (`__repr__`)

```python
class ResultadoInvalido(AssertionError):
    def __init__(self, esperado, obtenido):
        super().__init__(f"Se esperaba {esperado!r}, se obtuvo {obtenido!r}")

def test_con_excepcion_custom():
    resultado = procesar()
    if resultado != "ok":
        raise ResultadoInvalido("ok", resultado)
```

Para la gran mayoría de casos, un `assert` con mensaje f-string (ver [[02 - Escritura y Descubrimiento de Tests#assert con mensaje personalizado|Escritura de Tests]]) es suficiente y más simple — una excepción personalizada solo aporta valor cuando se reutiliza la misma lógica de comparación/mensaje en muchos tests distintos.

## Ver también

- [[07 - Manejo de Excepciones y Warnings]]
- [[09 - Mocking y Monkeypatching]]
- [[Python/NumPy/10 - Operaciones Matemáticas y Vectorización|Python/NumPy]]
