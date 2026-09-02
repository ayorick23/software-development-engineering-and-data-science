---
tags: [pytest, python, testing, cheat-sheet]
---

# 02 — Escritura y Descubrimiento de Tests

> Continúa de [[01 - Introducción y Arquitectura]].

## Convenciones de nombres para el descubrimiento automático

Pytest busca tests siguiendo un patrón de nombres específico — sin estas convenciones, un test simplemente **no se ejecuta**, sin ningún error visible:

| Elemento | Convención requerida |
|---|---|
| Archivos | `test_*.py` o `*_test.py` |
| Funciones | `test_*` (prefijo obligatorio) |
| Clases | `Test*` (prefijo, y **sin** `__init__`) |
| Métodos dentro de una clase | `test_*` |

```python
# test_calculadora.py — el archivo SÍ será descubierto

def test_suma():                    # SÍ se ejecuta — empieza con 'test_'
    assert sumar(2, 3) == 5

def suma_helper():                   # NO se ejecuta — no empieza con 'test_', se trata como función auxiliar normal
    return 2 + 3

class TestOperaciones:                 # SÍ se ejecuta — empieza con 'Test'
    def test_multiplicacion(self):
        assert multiplicar(2, 3) == 6
```

## Estructura de carpetas típica de un proyecto

```
mi_proyecto/
├── src/
│   └── calculadora.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py          # fixtures compartidas — ver 10
│   ├── test_calculadora.py
│   └── test_integracion.py
├── pytest.ini                 # o pyproject.toml — configuración de pytest
```

Mantener los tests en un directorio `tests/` separado del código fuente (en vez de mezclados) es la convención más común y facilita excluir tests de builds de producción sin configuración adicional.

## Opciones de línea de comandos más usadas

```bash
pytest -k "suma"                    # ejecuta solo tests cuyo nombre contenga "suma"
pytest -k "suma and not resta"         # combinar con expresiones booleanas
pytest -m "lento"                        # ejecuta solo tests marcados con @pytest.mark.lento — ver 06
pytest --collect-only                      # muestra QUÉ tests se ejecutarían, sin correrlos
pytest -x --pdb                              # detiene en el primer fallo y abre un debugger interactivo ahí mismo
pytest --lf                                    # "last failed" — re-ejecuta solo los tests que fallaron la última vez
pytest --ff                                      # "failed first" — ejecuta primero los que fallaron, luego el resto
```

`--lf`/`--ff` son extremadamente útiles en el ciclo de desarrollo iterativo: al arreglar un bug, no hace falta esperar a que corra toda la suite completa para verificar que el fix funcionó.

## Configuración: `pytest.ini` / `pyproject.toml`

```ini
# pytest.ini
[pytest]
testpaths = tests
python_files = test_*.py
python_functions = test_*
addopts = -v --tb=short
```

```toml
# pyproject.toml — forma moderna preferida, todo en un solo archivo de configuración del proyecto
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --tb=short"
markers = [
    "lento: marca tests que tardan más de lo normal",
]
```

`addopts` fija opciones de línea de comandos que se aplican **siempre** que se corre `pytest` en el proyecto, sin tener que escribirlas manualmente cada vez — el lugar recomendado para opciones que el equipo quiere como default (verbose, formato de traceback, etc.).

## `assert` con mensaje personalizado

```python
def test_suma():
    resultado = sumar(2, 3)
    assert resultado == 5, f"Se esperaba 5, se obtuvo {resultado}"
```

El mensaje personalizado se muestra **además** del detalle automático que pytest ya genera por la reescritura de aserciones (ver [[08 - Assertions Avanzadas]]) — útil para agregar contexto de negocio que el valor crudo no comunica por sí solo.

## Ver también

- [[01 - Introducción y Arquitectura]]
- [[03 - Fixtures - Fundamentos]]
- [[06 - Marks y Selección de Tests]]
- [[10 - conftest.py y Organización de Proyectos]]
