---
tags: [pytest, python, testing, exceptions, cheat-sheet]
---

# 07 — Manejo de Excepciones y Warnings

> Continúa de [[06 - Marks y Selección de Tests]].

## `pytest.raises()` — verificar que una excepción SÍ ocurre

```python
import pytest

def test_division_por_cero():
    with pytest.raises(ZeroDivisionError):
        dividir(10, 0)
```

Si el código dentro del bloque `with` **no** lanza la excepción esperada, el test falla — es la forma estándar de probar que una función maneja correctamente casos inválidos lanzando el error apropiado, en vez de fallar silenciosamente o devolver un valor incorrecto.

## Verificar el mensaje de la excepción con `match`

```python
def test_mensaje_de_error():
    with pytest.raises(ValueError, match="El precio no puede ser negativo"):
        validar_precio(-10)

def test_mensaje_con_regex():
    with pytest.raises(ValueError, match=r"^Error en la línea \d+$"):
        procesar_archivo_invalido()
```

`match` acepta una expresión regular que se busca (con `re.search()`, no `re.match()` completo pese al nombre del parámetro) dentro del mensaje de la excepción — verifica no solo **que** ocurrió el error correcto, sino que el mensaje comunica la razón esperada.

## Inspeccionar la excepción capturada

```python
def test_inspeccionar_excepcion():
    with pytest.raises(ValueError) as info_excepcion:
        validar_precio(-10)

    assert "negativo" in str(info_excepcion.value)
    assert info_excepcion.type is ValueError
```

`info_excepcion` (un objeto `ExceptionInfo`) da acceso al objeto de excepción real (`.value`), su tipo (`.type`), y el traceback (`.traceback`) — útil cuando verificar solo el mensaje con `match` no es suficiente y se necesita inspeccionar atributos específicos de una excepción personalizada.

## Verificar que NO se lanza ninguna excepción

```python
def test_no_lanza_excepcion():
    # simplemente llamar la función sin envolverla en pytest.raises —
    # si lanza una excepción inesperada, el test FALLA automáticamente
    resultado = funcion_segura(10)
    assert resultado == 20
```

No existe (ni hace falta) un `pytest.does_not_raise()` para el caso simple — cualquier excepción no capturada dentro de un test ya lo hace fallar por sí sola. `does_not_raise` (de `contextlib.nullcontext`, o una utilidad de terceros) solo es útil como placeholder dentro de una parametrización donde algunos casos SÍ deben lanzar y otros no.

```python
from contextlib import nullcontext as no_lanza

@pytest.mark.parametrize("valor, expectativa", [
    (10, no_lanza()),
    (-5, pytest.raises(ValueError)),
])
def test_validar_precio(valor, expectativa):
    with expectativa:
        validar_precio(valor)
```

## `pytest.warns()` — verificar que un warning SÍ se emite

```python
import warnings

def test_funcion_deprecada():
    with pytest.warns(DeprecationWarning, match="usar nueva_funcion en su lugar"):
        funcion_antigua()
```

Igual mecanismo que `pytest.raises()`, aplicado a `warnings.warn()` en vez de excepciones — necesario porque un warning no detiene la ejecución, así que sin `pytest.warns()` no habría forma directa de verificar que se emitió.

## Convertir warnings en errores dentro de la suite de tests

```ini
# pytest.ini
[pytest]
filterwarnings =
    error
    ignore::DeprecationWarning:some_third_party_library
```

`filterwarnings = error` hace que **cualquier** warning no filtrado explícitamente falle la suite — una práctica estricta mencionada en [[Machine Learning/25-Testing-Avanzado|Machine Learning/25 - Testing Avanzado]] para atrapar deprecaciones de dependencias antes de que se conviertan en errores reales en una versión futura de la librería.

## Ver también

- [[06 - Marks y Selección de Tests]]
- [[08 - Assertions Avanzadas]]
- [[Machine Learning/25-Testing-Avanzado|Machine Learning/25 - Testing Avanzado]]
