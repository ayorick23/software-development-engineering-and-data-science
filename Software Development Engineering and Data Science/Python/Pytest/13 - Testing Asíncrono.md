---
tags: [pytest, python, testing, async, cheat-sheet]
---

# 13 — Testing Asíncrono

> Continúa de [[12 - Cobertura de Código]].

## El problema: pytest no ejecuta corrutinas por default

```python
async def test_llamada_async():        # SIN pytest-asyncio, este test se SALTA silenciosamente (con un warning)
    resultado = await obtener_datos_async()
    assert resultado == 200
```

Una función `async def test_...` es una corrutina, no una función normal — pytest no sabe ejecutarla dentro de un event loop sin ayuda adicional. Sin el plugin correcto, el test ni siquiera falla claramente: se marca como `PASSED` sin haber ejecutado realmente el cuerpo, lo cual es peor que un fallo explícito.

## `pytest-asyncio` — la solución estándar

```bash
pip install pytest-asyncio
```

```python
import pytest

@pytest.mark.asyncio
async def test_llamada_async():
    resultado = await obtener_datos_async()
    assert resultado == 200
```

El mark `@pytest.mark.asyncio` le indica a pytest que ejecute esta función dentro de un event loop administrado por el plugin — sin él, incluso con `pytest-asyncio` instalado, el test seguiría sin ejecutarse correctamente (salvo con el modo automático, ver abajo).

## Modo automático — sin decorar cada test individualmente

```ini
# pytest.ini
[pytest]
asyncio_mode = auto
```

```python
async def test_llamada_async():     # con asyncio_mode = auto, NO hace falta el decorador @pytest.mark.asyncio
    resultado = await obtener_datos_async()
    assert resultado == 200
```

`asyncio_mode = auto` trata **cualquier** función `async def test_*` como un test asíncrono automáticamente — recomendado en proyectos donde la mayoría de los tests son async, para evitar repetir el decorador en cada uno.

## Fixtures asíncronas

```python
@pytest.fixture
async def cliente_http_async():
    async with httpx.AsyncClient() as cliente:
        yield cliente

@pytest.mark.asyncio
async def test_con_fixture_async(cliente_http_async):
    respuesta = await cliente_http_async.get("http://test.local/api")
    assert respuesta.status_code == 200
```

Una fixture también puede ser `async def` con `yield` — pytest-asyncio la resuelve dentro del mismo event loop que el test que la solicita, permitiendo `async with`/`await` dentro del propio setup/teardown.

## Testing de endpoints async de FastAPI

```python
import pytest
from httpx import AsyncClient, ASGITransport

@pytest.mark.asyncio
async def test_endpoint_async():
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as cliente:
        respuesta = await cliente.get("/usuarios/1")
        assert respuesta.status_code == 200
```

Para endpoints verdaderamente `async def` en FastAPI, usar un cliente async (`httpx.AsyncClient`) prueba el código en un contexto más fiel a como corre en producción que el `TestClient` síncrono — ver la cobertura completa de testing de FastAPI (incluyendo cuándo el `TestClient` síncrono es suficiente) en [[FastAPI/12 - Testing con TestClient y Pytest|FastAPI]].

## Configurar el scope del event loop

```python
@pytest.fixture(scope="session")
def event_loop():
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()
```

Por default, `pytest-asyncio` crea un event loop **nuevo por cada test** (`function` scope) — sobrescribir la fixture `event_loop` con un scope más amplio permite compartir un único loop entre tests que lo necesiten (por ejemplo, para reutilizar una conexión async costosa), a costa de perder el aislamiento total entre tests.

## Ver también

- [[12 - Cobertura de Código]]
- [[FastAPI/12 - Testing con TestClient y Pytest|FastAPI]]
- [[FastAPI/10 - Async y Concurrencia|FastAPI]]
