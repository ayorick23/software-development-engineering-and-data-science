---
tags: [pytest, python, testing, fixtures, cheat-sheet]
---

# 03 — Fixtures: Fundamentos

> Continúa de [[02 - Escritura y Descubrimiento de Tests]]. El mecanismo más distintivo de pytest frente a `unittest`.

## Qué es una fixture

Una **fixture** es una función decorada con `@pytest.fixture` que provee un estado, dato o recurso que un test necesita (una conexión, un objeto ya inicializado, datos de prueba) — pytest la ejecuta automáticamente y **inyecta su resultado** en cualquier test que declare un parámetro con el mismo nombre.

```python
import pytest

@pytest.fixture
def calculadora():
    return Calculadora()

def test_suma(calculadora):          # 'calculadora' se resuelve automáticamente por NOMBRE
    assert calculadora.sumar(2, 3) == 5

def test_resta(calculadora):           # la MISMA fixture, reutilizada en otro test
    assert calculadora.restar(5, 3) == 2
```

Esto reemplaza el `setUp()`/`tearDown()` de `unittest` (ver [[Testing#Unittest (PyUnit) El Framework Estándar|Python/Testing]]) con inyección de dependencias explícita por nombre de parámetro — más flexible porque cada test declara exactamente qué fixtures necesita, en vez de heredar un `setUp()` compartido por toda una clase.

## `yield` — setup y teardown en una sola función

```python
@pytest.fixture
def conexion_db():
    conexion = crear_conexion()       # SETUP — se ejecuta antes del test
    yield conexion                       # esto es lo que recibe el test
    conexion.cerrar()                      # TEARDOWN — se ejecuta después del test, incluso si falló
```

Todo lo que va **antes** del `yield` es la fase de preparación; todo lo que va **después** es la limpieza — y esa limpieza se ejecuta **siempre**, incluso si el test lanza una excepción, porque pytest la maneja dentro de un mecanismo equivalente a `try/finally`.

## Scope — cuántas veces se ejecuta una fixture

```python
@pytest.fixture(scope="function")    # default — se crea una instancia NUEVA para CADA test
@pytest.fixture(scope="class")         # una instancia compartida por toda una clase de tests
@pytest.fixture(scope="module")          # una instancia compartida por todo un archivo de test
@pytest.fixture(scope="session")           # UNA sola instancia para TODA la ejecución de pytest
```

| Scope | Se recrea | Uso típico |
|---|---|---|
| `function` (default) | En cada test | Estado que no debe compartirse entre tests (aislamiento total) |
| `class` | Una vez por clase de test | Recursos costosos compartidos dentro de un grupo relacionado |
| `module` | Una vez por archivo | Configuración compartida dentro de un archivo de tests |
| `session` | Una vez en toda la ejecución | Recursos muy costosos (levantar un contenedor Docker, conexión real a BD) — ver [[14 - Testing de Bases de Datos e Infraestructura]] |

```python
@pytest.fixture(scope="session")
def servidor_de_pruebas():
    servidor = levantar_servidor()
    yield servidor
    servidor.detener()
```

**Regla práctica:** usar `scope="function"` (el default) salvo que exista una razón concreta de rendimiento para compartir el recurso — un scope más amplio que `function` introduce riesgo de que el estado de un test "contamine" a otro si no se limpia cuidadosamente entre usos.

## Fixtures que dependen de otras fixtures

```python
@pytest.fixture
def conexion_db():
    return crear_conexion()

@pytest.fixture
def usuario_de_prueba(conexion_db):     # depende de otra fixture, declarada como parámetro
    usuario = conexion_db.crear_usuario("test@example.com")
    yield usuario
    conexion_db.eliminar_usuario(usuario.id)

def test_login(usuario_de_prueba):        # pytest resuelve TODA la cadena de dependencias automáticamente
    assert login(usuario_de_prueba) is True
```

Pytest resuelve el grafo completo de dependencias entre fixtures automáticamente — no hace falta orquestar manualmente el orden de creación, similar a cómo funciona la inyección de dependencias en [[FastAPI/06 - Dependency Injection|FastAPI]].

## Usar una fixture sin necesitar su valor de retorno

```python
@pytest.fixture
def limpiar_cache():
    cache.clear()
    yield
    cache.clear()

def test_algo(limpiar_cache):     # se ejecuta el setup/teardown, aunque el test no use el valor devuelto
    ...
```

## Ver también

- [[02 - Escritura y Descubrimiento de Tests]]
- [[04 - Fixtures Avanzadas]]
- [[10 - conftest.py y Organización de Proyectos]]
- [[FastAPI/06 - Dependency Injection|FastAPI]]
