---
tags: [pytest, python, testing, cheat-sheet]
---

# 01 — Introducción y Arquitectura

> Complementa la sección `## Pytest` de [[Testing#Pytest El Estándar Moderno de la Comunidad|Python/Testing]] con la profundidad práctica de sintaxis.

**Pytest** es el framework de testing de facto de la comunidad Python — no viene en la librería estándar (a diferencia de [[Testing#Unittest (PyUnit) El Framework Estándar|unittest]]), pero se ha vuelto el estándar de la industria por su sintaxis mínima, su sistema de fixtures, y su enorme ecosistema de plugins.

```bash
pip install pytest
```

## La diferencia filosófica con `unittest`: `assert` plano

```python
# unittest — requiere métodos de aserción específicos, y clases que hereden de TestCase
import unittest

class TestSuma(unittest.TestCase):
    def test_suma(self):
        self.assertEqual(sumar(2, 3), 5)

# pytest — 'assert' normal de Python, funciones sueltas, sin clases obligatorias
def test_suma():
    assert sumar(2, 3) == 5
```

Pytest no requiere heredar de ninguna clase ni memorizar decenas de métodos `assertX()` — usa el `assert` nativo de Python, y reescribe internamente la expresión al fallar para mostrar un mensaje de error detallado (ver [[08 - Assertions Avanzadas]]) sin que el desarrollador tenga que hacer nada especial.

## Descubrimiento automático de tests

```bash
pytest                    # busca y ejecuta TODOS los tests del directorio actual y subdirectorios
pytest test_calculadora.py   # un archivo específico
pytest test_calculadora.py::test_suma   # una función específica dentro de un archivo
```

Pytest descubre tests automáticamente según convenciones de nombres (ver el detalle completo en [[02 - Escritura y Descubrimiento de Tests]]) — no requiere un `if __name__ == "__main__": unittest.main()` ni registrar manualmente cada test en una suite.

## Ejecutar y leer la salida

```bash
pytest -v                  # verbose — muestra el nombre de cada test individual, no solo un resumen con puntos
pytest -x                    # detiene la ejecución en el PRIMER fallo
pytest --tb=short              # traceback más corto en los fallos (por default es bastante largo)
pytest -q                        # quiet — solo el resumen final
```

```
test_calculadora.py::test_suma PASSED                                  [ 50%]
test_calculadora.py::test_resta FAILED                                 [100%]

=================================== FAILURES ===================================
_________________________________ test_resta ___________________________________

    def test_resta():
>       assert restar(5, 3) == 3
E       assert 2 == 3
E        +  where 2 = restar(5, 3)
```

La salida de un fallo muestra la expresión exacta que fue evaluada y el valor real obtenido (`assert 2 == 3`, `where 2 = restar(5, 3)`) — esto es la **reescritura de aserciones** de pytest en acción, generando este nivel de detalle a partir de un `assert` completamente normal de Python.

## Arquitectura basada en plugins

Pytest está construido internamente sobre un sistema de **hooks** y **plugins** — la mayoría de sus propias características "core" (fixtures, marks, parametrización) están implementadas como plugins internos usando el mismo mecanismo que cualquier plugin de terceros (`pytest-cov`, `pytest-xdist`, ver [[11 - Plugins del Ecosistema]]) usaría. Esto explica por qué el ecosistema de plugins de pytest es tan rico: extender pytest no requiere modificar su núcleo, solo engancharse al mismo sistema de hooks que usa internamente.

## Ver también

- [[02 - Escritura y Descubrimiento de Tests]]
- [[03 - Fixtures - Fundamentos]]
- [[17 - Buenas Prácticas, Errores Comunes y Comparativa]]
- [[Testing#Pytest El Estándar Moderno de la Comunidad|Python/Testing]]
