---
tags: [fastapi, apis, pydantic, python, cheat-sheet]
---

# 03 — Request Body y Modelos Pydantic

> Continúa de [[02 - Path Operations y Routing]].

## Por qué `BaseModel` en vez de un `dict`

```python
from pydantic import BaseModel

class Item(BaseModel):
    nombre: str
    precio: float
    en_stock: bool = True       # con valor por defecto → campo opcional
    descripcion: str | None = None
```

Cualquier clase que herede de `BaseModel` se convierte en un esquema: FastAPI la usa para (1) validar el JSON que llega en el body, (2) convertirlo a un objeto Python real con autocompletado, y (3) generar el esquema de OpenAPI que ves en `/docs`. Todo desde una sola declaración.

## Usar el modelo como parámetro del endpoint

```python
@app.post("/items")
def crear_item(item: Item):
    # item ya es un objeto Item validado, no un dict crudo
    item_dict = item.model_dump()   # Pydantic v2 (en v1 era .dict())
    if item.descripcion:
        item_dict["descripcion_larga"] = f"{item.nombre}: {item.descripcion}"
    return item_dict
```

Si el JSON recibido no cumple el esquema (falta `nombre`, `precio` es un string no convertible, etc.), FastAPI responde **422 Unprocessable Entity** con el detalle exacto de qué campo falló — antes de que la función `crear_item` se ejecute.

## Modelos anidados

```python
class Direccion(BaseModel):
    calle: str
    ciudad: str
    codigo_postal: str

class Usuario(BaseModel):
    nombre: str
    email: str
    direccion: Direccion            # modelo dentro de modelo
    direcciones_previas: list[Direccion] = []   # lista de modelos

@app.post("/usuarios")
def crear_usuario(usuario: Usuario):
    return {"ciudad": usuario.direccion.ciudad}
```

```json
{
  "nombre": "Ana",
  "email": "ana@example.com",
  "direccion": {"calle": "Calle 1", "ciudad": "Santo Domingo", "codigo_postal": "10101"},
  "direcciones_previas": []
}
```

Pydantic valida la estructura completa recursivamente — un error en `direccion.ciudad` se reporta con la ruta exacta (`direccion -> ciudad`) en la respuesta de error.

## Combinar body, path y query en el mismo endpoint

```python
@app.put("/items/{item_id}")
def actualizar_item(
    item_id: int,          # viene de la ruta
    item: Item,             # viene del body (es un BaseModel → FastAPI lo infiere)
    q: str | None = None,   # viene de query string (tipo simple, sin ruta)
):
    resultado = {"item_id": item_id, **item.model_dump()}
    if q:
        resultado["q"] = q
    return resultado
```

FastAPI decide de dónde viene cada parámetro por su tipo: si es un `BaseModel`, del body; si es un tipo simple y está en la ruta, de la ruta; si no está en la ruta, de la query string.

## Múltiples modelos en el mismo body con `Body(embed=True)`

```python
from fastapi import Body

@app.post("/items/{item_id}")
def actualizar(
    item_id: int,
    item: Item,
    usuario: Usuario,
    importancia: Annotated[int, Body(gt=0)],
):
    # el body esperado ahora es:
    # {"item": {...}, "usuario": {...}, "importancia": 5}
    # en vez de fusionar los campos de Item y Usuario en el nivel raíz
    ...
```

Cuando hay más de un modelo `BaseModel` como parámetro, FastAPI automáticamente los anida bajo su propio nombre en el JSON esperado — no hace falta `embed=True` explícito salvo con un único modelo que quieras forzar a anidarse.

## `model_config` — comportamiento del modelo

```python
from pydantic import BaseModel, ConfigDict

class Item(BaseModel):
    model_config = ConfigDict(str_strip_whitespace=True, extra="forbid")

    nombre: str
    precio: float
```

- `extra="forbid"` — rechaza el request si llega un campo no declarado en el modelo (por defecto Pydantic los ignora silenciosamente).
- `str_strip_whitespace=True` — recorta espacios en strings automáticamente antes de validar.

## Ejemplo de schema para la documentación

```python
class Item(BaseModel):
    nombre: str
    precio: float

    model_config = ConfigDict(
        json_schema_extra={
            "example": {"nombre": "Laptop", "precio": 999.99}
        }
    )
```

Este ejemplo aparece precargado en el formulario de prueba de Swagger UI (`/docs`), útil para que cualquiera que consuma la API entienda el formato esperado sin leer código.

## Ver también

- [[02 - Path Operations y Routing]]
- [[04 - Response Models y Serialización]]
- [[05 - Validación Avanzada con Pydantic]]
