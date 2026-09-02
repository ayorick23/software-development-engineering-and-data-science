---
tags: [fastapi, apis, bases-de-datos, python, cheat-sheet]
---

# 11 — Bases de Datos con SQLAlchemy y SQLModel

> Continúa de [[10 - Async y Concurrencia]].

## El patrón estándar: sesión de BD como dependencia con `yield`

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session

engine = create_engine("postgresql://usuario:pass@localhost/mibd")
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

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

Este es el patrón visto en [[06 - Dependency Injection]] aplicado específicamente a bases de datos: cada request obtiene su propia sesión, y `db.close()` se garantiza sin importar si el endpoint termina bien o lanza una excepción — evita conexiones colgadas, el problema más común en APIs que manejan la sesión manualmente.

## SQLModel — SQLAlchemy + Pydantic en una sola clase

**SQLModel**, del mismo autor de FastAPI, resuelve un problema recurrente: con SQLAlchemy clásico, defines el modelo ORM y el modelo Pydantic por separado (duplicación). SQLModel los fusiona:

```python
from sqlmodel import SQLModel, Field, Session, create_engine, select

class Item(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    nombre: str
    precio: float

engine = create_engine("sqlite:///./base.db")
SQLModel.metadata.create_all(engine)

def obtener_sesion():
    with Session(engine) as sesion:
        yield sesion

@app.post("/items", response_model=Item)
def crear_item(item: Item, sesion: Annotated[Session, Depends(obtener_sesion)]):
    sesion.add(item)
    sesion.commit()
    sesion.refresh(item)
    return item

@app.get("/items", response_model=list[Item])
def listar_items(sesion: Annotated[Session, Depends(obtener_sesion)]):
    return sesion.exec(select(Item)).all()
```

La misma clase `Item` sirve como tabla de base de datos (`table=True`), esquema de validación de entrada y `response_model` de salida — un solo lugar para mantener en vez de tres.

## Separar modelo de tabla y modelo de API (recomendado en proyectos reales)

```python
class ItemBase(SQLModel):
    nombre: str
    precio: float

class Item(ItemBase, table=True):        # tabla real en la BD
    id: int | None = Field(default=None, primary_key=True)

class ItemCrear(ItemBase):                # lo que el cliente envía (sin id)
    pass

class ItemLeer(ItemBase):                 # lo que el cliente recibe (con id)
    id: int
```

Igual que en [[04 - Response Models y Serialización]], separar el modelo de entrada del de salida evita exponer campos internos (por ejemplo, un `password_hash` que sí vive en la tabla pero nunca debe salir en una respuesta).

## Async SQLAlchemy — para endpoints `async def`

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession

engine = create_async_engine("postgresql+asyncpg://usuario:pass@localhost/mibd")

async def obtener_db():
    async with AsyncSession(engine) as sesion:
        yield sesion

@app.get("/items")
async def listar_items(db: Annotated[AsyncSession, Depends(obtener_db)]):
    resultado = await db.execute(select(Item))
    return resultado.scalars().all()
```

Solo tiene sentido combinarlo con endpoints `async def` — mezclar un engine async con un endpoint `def` normal pierde la ventaja (ver la tabla de decisión en [[10 - Async y Concurrencia]]).

## Migraciones con Alembic — mención breve

```bash
pip install alembic
alembic init migraciones
alembic revision --autogenerate -m "agregar tabla items"
alembic upgrade head
```

SQLModel/SQLAlchemy definen el esquema en Python, pero **no** modifican una base de datos ya existente en producción de forma segura por sí solos — Alembic genera y versiona los scripts de migración (`ALTER TABLE`, etc.) a partir de las diferencias entre el modelo en código y el estado actual de la BD.

## Ver también

- [[06 - Dependency Injection]]
- [[10 - Async y Concurrencia]]
- [[04 - Response Models y Serialización]]
