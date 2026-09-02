---
tags: [fastapi, apis, python, cheat-sheet]
---

# 14 — Routers y Estructura de Proyectos Grandes

> Continúa de [[13 - WebSockets y Streaming Responses]].

## El problema: todo en un solo `main.py`

Los ejemplos de esta serie de notas viven todos en un `main.py` para simplicidad — en un proyecto real con decenas de endpoints, eso se vuelve inmanejable rápido. `APIRouter` permite dividir la app en módulos independientes que se ensamblan al final.

## `APIRouter` — un mini-`FastAPI` por módulo

```python
# routers/items.py
from fastapi import APIRouter, Depends

router = APIRouter(
    prefix="/items",
    tags=["items"],                          # agrupa estos endpoints en Swagger UI
    dependencies=[Depends(verificar_api_key)], # aplica a TODOS los endpoints de este router
)

@router.get("/")
def listar_items(): ...

@router.get("/{id}")
def obtener_item(id: int): ...

@router.post("/")
def crear_item(): ...
```

```python
# routers/usuarios.py
from fastapi import APIRouter

router = APIRouter(prefix="/usuarios", tags=["usuarios"])

@router.get("/")
def listar_usuarios(): ...
```

```python
# main.py
from fastapi import FastAPI
from routers import items, usuarios

app = FastAPI()
app.include_router(items.router)
app.include_router(usuarios.router)
# el endpoint final queda en /items/... y /usuarios/... respectivamente
```

`prefix` evita repetir `/items` en cada ruta del router, `tags` agrupa visualmente los endpoints relacionados en `/docs`, y `dependencies` a nivel de router (ver [[06 - Dependency Injection]]) aplica autenticación/validación a todo el módulo de una sola vez.

## Estructura de carpetas recomendada

```
mi_proyecto/
├── app/
│   ├── main.py                  # crea la app, incluye los routers
│   ├── dependencies.py          # dependencias compartidas (get_db, get_current_user)
│   ├── config.py                # settings con pydantic-settings, ver 15
│   ├── models/                  # modelos de base de datos (SQLAlchemy/SQLModel)
│   │   ├── item.py
│   │   └── usuario.py
│   ├── schemas/                 # modelos Pydantic de entrada/salida (si se separan de models/)
│   │   ├── item.py
│   │   └── usuario.py
│   ├── routers/                 # un archivo por recurso/dominio
│   │   ├── items.py
│   │   └── usuarios.py
│   └── services/                # lógica de negocio, separada de los endpoints
│       └── item_service.py
├── tests/
│   ├── test_items.py
│   └── test_usuarios.py
├── alembic/                     # migraciones, ver 11
├── requirements.txt
└── Dockerfile
```

La separación clave: los **endpoints** (`routers/`) solo deberían orquestar — validar entrada, llamar a un servicio, devolver una respuesta — mientras que la **lógica de negocio** real vive en `services/`, testeable independientemente de FastAPI (ver [[12 - Testing con TestClient y Pytest]]).

## Routers anidados

```python
# routers/admin.py
router_admin = APIRouter(prefix="/admin")
router_admin.include_router(items.router)   # /admin/items/...
router_admin.include_router(usuarios.router) # /admin/usuarios/...

app.include_router(router_admin)
```

Un `APIRouter` puede incluir a otros routers — útil para agrupar un conjunto completo de recursos bajo un prefijo común (por ejemplo, todo lo que requiere permisos de administrador).

## `APIRouter` con su propio prefijo de versión

```python
router_v1 = APIRouter(prefix="/api/v1")
router_v1.include_router(items.router)
router_v1.include_router(usuarios.router)

app.include_router(router_v1)
```

Patrón común para versionar una API sin duplicar código de endpoints — cuando llegue `/api/v2`, se define un nuevo router con los cambios y ambos coexisten mientras se migra a los clientes.

## Eventos de ciclo de vida — `lifespan`

```python
from contextlib import asynccontextmanager

modelo_ml = None

@asynccontextmanager
async def lifespan(app: FastAPI):
    global modelo_ml
    modelo_ml = cargar_modelo_pesado()   # se ejecuta UNA VEZ al iniciar
    yield
    modelo_ml = None                       # cleanup al apagar (cerrar conexiones, liberar memoria)

app = FastAPI(lifespan=lifespan)
```

Reemplaza a los antiguos decoradores `@app.on_event("startup")`/`@app.on_event("shutdown")` (deprecados) — es el lugar correcto para cargar un modelo de ML una sola vez al arrancar el proceso, en vez de en cada request (ver el antipatrón cubierto en [[49-APIs-con-FastAPI-para-Servir-Modelos]]).

## Ver también

- [[06 - Dependency Injection]]
- [[12 - Testing con TestClient y Pytest]]
- [[15 - Despliegue y Producción]]
