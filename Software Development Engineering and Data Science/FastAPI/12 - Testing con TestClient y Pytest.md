---
tags: [fastapi, apis, testing, python, cheat-sheet]
---

# 12 — Testing con TestClient y Pytest

> Continúa de [[11 - Bases de Datos con SQLAlchemy y SQLModel]].

## `TestClient` — probar la API sin levantar un servidor real

```python
# main.py
from fastapi import FastAPI
app = FastAPI()

@app.get("/items/{id}")
def obtener_item(id: int):
    if id == 0:
        return {"error": "no existe"}, 404
    return {"id": id, "nombre": "Laptop"}
```

```python
# test_main.py
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_obtener_item():
    respuesta = client.get("/items/1")
    assert respuesta.status_code == 200
    assert respuesta.json() == {"id": 1, "nombre": "Laptop"}

def test_item_invalido():
    respuesta = client.get("/items/abc")
    assert respuesta.status_code == 422   # falla la validación de tipo, ni siquiera llega al endpoint
```

`TestClient` está construido sobre `httpx` (ver [[10 - Async y Concurrencia]]) y simula requests HTTP reales contra la app **sin** necesitar un proceso Uvicorn corriendo — las pruebas son rápidas porque todo ocurre en memoria, en el mismo proceso de pytest.

## Sobrescribir dependencias en los tests — la técnica clave

```python
def obtener_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/items")
def listar_items(db: Annotated[Session, Depends(obtener_db)]):
    return db.query(Item).all()
```

```python
# test_main.py
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine_test = create_engine("sqlite:///./test.db")
SessionLocalTest = sessionmaker(bind=engine_test)

def obtener_db_test():
    db = SessionLocalTest()
    try:
        yield db
    finally:
        db.close()

app.dependency_overrides[obtener_db] = obtener_db_test   # reemplaza la dependencia real

client = TestClient(app)

def test_listar_items():
    respuesta = client.get("/items")
    assert respuesta.status_code == 200
```

`app.dependency_overrides` es un diccionario que FastAPI consulta antes de ejecutar la dependencia real — permite sustituir la base de datos de producción por una de test, o un servicio externo por un mock, sin tocar el código del endpoint.

## Fixtures de pytest para setup/teardown reutilizable

```python
import pytest

@pytest.fixture
def client():
    SQLModel.metadata.create_all(engine_test)
    app.dependency_overrides[obtener_db] = obtener_db_test
    with TestClient(app) as c:
        yield c
    SQLModel.metadata.drop_all(engine_test)   # limpia la BD de test después de cada test
    app.dependency_overrides.clear()

def test_crear_item(client):
    respuesta = client.post("/items", json={"nombre": "Mouse", "precio": 25.0})
    assert respuesta.status_code == 200
```

Usar `with TestClient(app) as c:` (en vez de instanciarlo directo) dispara los eventos de `startup`/`shutdown` de la app — necesario si tu aplicación tiene lógica en esos hooks (cargar un modelo de ML al iniciar, por ejemplo, como en [[49-APIs-con-FastAPI-para-Servir-Modelos]]).

## Probar endpoints protegidos por autenticación

```python
def test_endpoint_protegido_sin_token():
    respuesta = client.get("/perfil")
    assert respuesta.status_code == 401

def test_endpoint_protegido_con_token():
    token = obtener_token_de_prueba()
    respuesta = client.get("/perfil", headers={"Authorization": f"Bearer {token}"})
    assert respuesta.status_code == 200
```

O, más simple para tests unitarios que no quieren probar el flujo de login completo: sobrescribir directamente la dependencia `obtener_usuario_actual` (ver [[09 - Autenticación y Seguridad]]) con `app.dependency_overrides`, devolviendo un usuario de prueba fijo.

## Probar endpoints async con `httpx.AsyncClient`

```python
import pytest
from httpx import AsyncClient, ASGITransport

@pytest.mark.anyio   # requiere: pip install anyio, o usar pytest-asyncio
async def test_endpoint_async():
    async with AsyncClient(transport=ASGITransport(app=app), base_url="http://test") as ac:
        respuesta = await ac.get("/items")
    assert respuesta.status_code == 200
```

`TestClient` normal funciona incluso para endpoints `async def` (los ejecuta de forma síncrona por debajo) — `AsyncClient` solo es necesario cuando el propio test necesita ser `async` (por ejemplo, para llamar a otras corutinas de setup).

## Ver también

- [[06 - Dependency Injection]]
- [[10 - Async y Concurrencia]]
- [[14 - Routers y Estructura de Proyectos Grandes]]
