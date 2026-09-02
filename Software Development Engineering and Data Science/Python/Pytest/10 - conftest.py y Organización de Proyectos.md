---
tags: [pytest, python, testing, conftest, cheat-sheet]
---

# 10 — conftest.py y Organización de Proyectos

> Continúa de [[09 - Mocking y Monkeypatching]].

## Qué es `conftest.py`

`conftest.py` es un archivo especial que pytest carga **automáticamente** (sin necesidad de importarlo explícitamente) — cualquier fixture definida ahí queda disponible para todos los tests en su mismo directorio y subdirectorios, sin necesidad de un `import`.

```python
# tests/conftest.py
import pytest

@pytest.fixture
def cliente_api():
    return ClienteAPI(base_url="http://test.local")
```

```python
# tests/test_usuarios.py — usa 'cliente_api' SIN importarla de ningún lado
def test_obtener_usuario(cliente_api):
    usuario = cliente_api.obtener_usuario(1)
    assert usuario["id"] == 1
```

## Jerarquía de múltiples `conftest.py`

```
tests/
├── conftest.py              # fixtures globales, disponibles en TODO tests/
├── test_general.py
├── api/
│   ├── conftest.py            # fixtures específicas de api/, además de las globales
│   └── test_endpoints.py
└── db/
    ├── conftest.py              # fixtures específicas de db/
    └── test_modelos.py
```

Pytest busca fixtures empezando por el `conftest.py` más cercano al test y subiendo por el árbol de directorios — permite organizar fixtures compartidas ampliamente en la raíz y fixtures específicas de un subsistema solo donde se necesitan, sin contaminar el namespace global. Ver también cómo un `conftest.py` más específico puede **sobrescribir** una fixture del padre en [[04 - Fixtures Avanzadas#Sobrescribir una fixture en un nivel más específico|Fixtures Avanzadas]].

## Hooks básicos en `conftest.py`

```python
# conftest.py
def pytest_configure(config):
    """Se ejecuta una vez al inicio, antes de recolectar los tests."""
    config.addinivalue_line("markers", "lento: tests que tardan más de lo normal")

def pytest_collection_modifyitems(config, items):
    """Se ejecuta después de recolectar todos los tests — permite modificar la lista."""
    if config.getoption("--skip-lentos"):
        skip_lento = pytest.mark.skip(reason="--skip-lentos activo")
        for item in items:
            if "lento" in item.keywords:
                item.add_marker(skip_lento)
```

Los **hooks** son funciones con nombres reservados que pytest llama automáticamente en puntos específicos de su ciclo de vida (configuración, recolección, ejecución) — esta es la misma arquitectura de plugins mencionada en [[01 - Introducción y Arquitectura#Arquitectura basada en plugins|Introducción]], accesible directamente desde el `conftest.py` del propio proyecto sin necesitar empaquetar un plugin formal.

## Agregar opciones de línea de comandos personalizadas

```python
# conftest.py
def pytest_addoption(parser):
    parser.addoption("--skip-lentos", action="store_true", default=False, help="Saltar tests marcados como lentos")

@pytest.fixture
def entorno(request):
    return "produccion" if request.config.getoption("--skip-lentos") else "desarrollo"
```

```bash
pytest --skip-lentos
```

`pytest_addoption` permite definir flags propios del proyecto, accesibles después vía `request.config.getoption(...)` — útil para adaptar el comportamiento de la suite según el contexto de ejecución (local vs CI, rápido vs completo).

## Compartir fixtures entre proyectos: plugins internos

```python
# mi_paquete_de_testing/plugin.py
import pytest

@pytest.fixture
def usuario_de_prueba():
    return {"id": 1, "nombre": "Test User"}
```

```toml
# pyproject.toml de OTRO proyecto que reutiliza el plugin
[project.entry-points.pytest11]
mi_plugin = "mi_paquete_de_testing.plugin"
```

Cuando fixtures/hooks necesitan reutilizarse entre **múltiples proyectos** (no solo entre módulos del mismo proyecto), empaquetarlas como un plugin instalable vía `entry-points.pytest11` es el mecanismo formal — pytest lo descubre automáticamente en cualquier proyecto donde el paquete esté instalado, sin necesitar copiar el `conftest.py` manualmente.

## Ver también

- [[09 - Mocking y Monkeypatching]]
- [[04 - Fixtures Avanzadas]]
- [[11 - Plugins del Ecosistema]]
