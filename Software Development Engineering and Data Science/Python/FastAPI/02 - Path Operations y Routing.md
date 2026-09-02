---
tags: [fastapi, apis, python, cheat-sheet]
---

# 02 — Path Operations y Routing

> Continúa de [[Python/FastAPI/01 - Introducción y Arquitectura]].

## Los verbos HTTP como decoradores

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/items")       # leer
def listar_items(): ...

@app.post("/items")      # crear
def crear_item(): ...

@app.put("/items/{id}")  # reemplazar completo
def reemplazar_item(id: int): ...

@app.patch("/items/{id}") # actualizar parcial
def actualizar_item(id: int): ...

@app.delete("/items/{id}") # eliminar
def borrar_item(id: int): ...
```

Cada decorador registra una **path operation**: una combinación única de método HTTP + ruta. Cada uno de estos conceptos (verbos, códigos de estado) ya se cubrió a nivel general en `Python/APIs.md` — aquí es cómo se implementan concretamente en FastAPI.

## Path parameters — parte de la URL

```python
@app.get("/usuarios/{usuario_id}")
def obtener_usuario(usuario_id: int):
    # si alguien pide /usuarios/abc, FastAPI responde 422 automáticamente
    # antes de que esta función se ejecute siquiera
    return {"usuario_id": usuario_id}
```

El type hint (`int`) no es decorativo: FastAPI lo usa para **convertir y validar** el valor que llega como string en la URL.

### Validación adicional con `Path()`

```python
from fastapi import Path
from typing import Annotated

@app.get("/usuarios/{usuario_id}")
def obtener_usuario(
    usuario_id: Annotated[int, Path(title="ID del usuario", ge=1, le=10_000)]
):
    return {"usuario_id": usuario_id}
```

`Annotated` (de `typing`) es la forma recomendada desde FastAPI 0.95+ de combinar el tipo real con los metadatos de validación — separa "qué tipo es" de "cómo se valida", y es la misma sintaxis que reutilizarás en query params y body.

## Query parameters — después del `?`

```python
@app.get("/items")
def listar_items(skip: int = 0, limit: int = 10, buscar: str | None = None):
    # /items?skip=20&limit=5&buscar=laptop
    ...
```

Cualquier parámetro de la función que **no** aparezca en la ruta (`{...}`) se interpreta automáticamente como query parameter. Si tiene un valor por defecto, es opcional; si no, es obligatorio.

### Validación con `Query()`

```python
from fastapi import Query

@app.get("/items")
def listar_items(
    q: Annotated[str | None, Query(min_length=3, max_length=50, regex="^[a-zA-Z]+$")] = None,
    limit: Annotated[int, Query(gt=0, le=100)] = 10,
):
    ...
```

## Combinando path + query + múltiples parámetros

```python
@app.get("/usuarios/{usuario_id}/items/{item_id}")
def obtener_item_de_usuario(
    usuario_id: int,          # path param
    item_id: str,             # path param
    q: str | None = None,     # query param opcional
    completo: bool = False,   # query param opcional, ?completo=true
):
    ...
```

FastAPI distingue automáticamente cada tipo de parámetro por dónde aparece (en la ruta o no) — no hace falta declararlo explícitamente salvo que quieras validación adicional.

## El orden de las rutas importa

```python
# INCORRECTO: "/usuarios/me" nunca se alcanza,
# porque "/usuarios/{usuario_id}" la intercepta primero
@app.get("/usuarios/{usuario_id}")
def obtener_usuario(usuario_id: str): ...

@app.get("/usuarios/me")
def obtener_usuario_actual(): ...

# CORRECTO: las rutas fijas van ANTES que las rutas con parámetros dinámicos
@app.get("/usuarios/me")
def obtener_usuario_actual(): ...

@app.get("/usuarios/{usuario_id}")
def obtener_usuario(usuario_id: str): ...
```

FastAPI evalúa las rutas en el orden en que se declaran — la primera que hace match gana.

## Parámetros con valores predefinidos — `Enum`

```python
from enum import Enum

class Modelo(str, Enum):
    resnet = "resnet"
    bert = "bert"
    xgboost = "xgboost"

@app.get("/modelos/{nombre}")
def obtener_modelo(nombre: Modelo):
    # Swagger UI muestra un dropdown con las 3 opciones válidas
    # cualquier otro valor recibe 422 automáticamente
    return {"modelo": nombre, "descripcion": f"Detalles de {nombre.value}"}
```

## Capturar rutas completas con `:path`

```python
@app.get("/archivos/{ruta_archivo:path}")
def leer_archivo(ruta_archivo: str):
    # /archivos/carpeta/subcarpeta/reporte.csv
    # ruta_archivo == "carpeta/subcarpeta/reporte.csv"
    return {"ruta": ruta_archivo}
```

Sin el sufijo `:path`, un parámetro de ruta normal no puede contener `/` — este convertidor especial lo permite.

## `status_code` por defecto en cada operación

```python
from fastapi import status

@app.post("/items", status_code=status.HTTP_201_CREATED)
def crear_item(): ...

@app.delete("/items/{id}", status_code=status.HTTP_204_NO_CONTENT)
def borrar_item(id: int): ...
```

Usar las constantes de `fastapi.status` en vez de números mágicos (`201`, `204`) hace el código autodocumentado y evita errores de tipeo.

## Ver también

- [[Python/FastAPI/01 - Introducción y Arquitectura]]
- [[03 - Request Body y Modelos Pydantic]]
- [[14 - Routers y Estructura de Proyectos Grandes]]
