---
tags: [pytest, python, testing, fixtures, cheat-sheet]
---

# 04 — Fixtures Avanzadas

> Continúa de [[03 - Fixtures - Fundamentos]].

## `autouse=True` — fixtures que se ejecutan sin ser solicitadas

```python
@pytest.fixture(autouse=True)
def resetear_configuracion_global():
    configuracion.reset()
    yield
    configuracion.reset()
```

Una fixture `autouse` se ejecuta automáticamente en **todos** los tests dentro de su alcance (el archivo, la clase, o el proyecto entero si está en `conftest.py`), sin que ningún test la declare explícitamente como parámetro — útil para setup/teardown que debe aplicarse universalmente (limpiar estado global, resetear una base de datos de prueba) sin repetir el nombre en cada firma de test.

## Fixtures como fábricas (factory fixtures)

```python
@pytest.fixture
def crear_usuario():
    usuarios_creados = []

    def _crear_usuario(nombre, edad=25):          # función interna que el test puede llamar CON ARGUMENTOS
        usuario = Usuario(nombre=nombre, edad=edad)
        usuarios_creados.append(usuario)
        return usuario

    yield _crear_usuario
    for usuario in usuarios_creados:                 # limpieza de TODO lo creado durante el test
        usuario.eliminar()

def test_multiples_usuarios(crear_usuario):
    ana = crear_usuario("Ana", edad=28)
    luis = crear_usuario("Luis")                        # usa el default edad=25
    assert ana.edad == 28
```

Una fixture normal devuelve un valor **fijo**; una factory fixture devuelve una **función** que el test puede llamar múltiples veces con argumentos distintos — necesario cuando un test requiere crear varias instancias con parámetros diferentes, algo que una fixture simple no puede resolver sin duplicar código.

## El objeto `request` — introspección dentro de una fixture

```python
@pytest.fixture
def recurso(request):
    print(f"Ejecutando para el test: {request.node.name}")
    tipo = getattr(request, "param", "default")
    yield crear_recurso(tipo)
```

`request` es una fixture especial provista automáticamente por pytest que da acceso a metadatos sobre el test que está solicitando la fixture actual — el nombre del test, sus marks (ver [[06 - Marks y Selección de Tests]]), y el parámetro actual si la fixture está parametrizada (ver la sección siguiente).

## Fixtures parametrizadas

```python
@pytest.fixture(params=["sqlite", "postgres", "mysql"])
def motor_db(request):
    return conectar(request.param)                # request.param toma cada valor de 'params', uno por ejecución

def test_insercion(motor_db):                        # este test se ejecuta 3 VECES, una por cada motor
    assert motor_db.insertar({"id": 1}) is True
```

A diferencia de `@pytest.mark.parametrize` (ver [[05 - Parametrización de Tests]], que parametriza un test específico), parametrizar una **fixture** hace que **todos los tests que la usan** se multipliquen automáticamente — útil cuando muchos tests distintos deben correr contra cada variante (ej. cada motor de base de datos soportado).

## `yield_fixture` vs `fixture` con `yield` — nota histórica

```python
@pytest.fixture     # forma moderna y única recomendada — funciona igual con return o yield
def algo():
    yield "valor"
```

`@pytest.yield_fixture` existió como decorador separado en versiones antiguas de pytest — está deprecado; `@pytest.fixture` normal ya soporta `yield` para setup/teardown desde hace años, no hay razón para usar la forma antigua en código nuevo.

## Sobrescribir una fixture en un nivel más específico

```python
# conftest.py (raíz del proyecto)
@pytest.fixture
def configuracion():
    return {"entorno": "produccion"}

# tests/modulo_especifico/conftest.py (más específico)
@pytest.fixture
def configuracion():                    # mismo nombre — SOBRESCRIBE la fixture del conftest.py padre
    return {"entorno": "test"}
```

Pytest resuelve fixtures buscando desde el `conftest.py` más cercano al test hacia arriba — permite que un módulo específico redefina una fixture compartida sin afectar al resto del proyecto. Ver la jerarquía completa de `conftest.py` en [[10 - conftest.py y Organización de Proyectos]].

## Ver también

- [[03 - Fixtures - Fundamentos]]
- [[05 - Parametrización de Tests]]
- [[10 - conftest.py y Organización de Proyectos]]
