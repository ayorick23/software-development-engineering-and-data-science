---
tags: [fastapi, apis, pydantic, python, cheat-sheet]
---

# 04 — Response Models y Serialización

> Continúa de [[03 - Request Body y Modelos Pydantic]].

## Por qué declarar `response_model`

```python
from pydantic import BaseModel

class UsuarioEntrada(BaseModel):
    nombre: str
    email: str
    password: str

class UsuarioSalida(BaseModel):
    nombre: str
    email: str
    # nótese: password NO está aquí

@app.post("/usuarios", response_model=UsuarioSalida)
def crear_usuario(usuario: UsuarioEntrada):
    guardar_en_bd(usuario)          # usuario.password sí existe internamente
    return usuario                    # FastAPI filtra el campo password al serializar
```

Este es el patrón más importante de esta nota: **el modelo de entrada y el modelo de salida casi nunca deben ser el mismo**. `response_model` garantiza que, sin importar qué devuelva la función, la respuesta HTTP solo incluya los campos declarados en el esquema de salida — evita fugas accidentales de datos sensibles (contraseñas, hashes, campos internos).

## Alternativa: type hint de retorno como `response_model`

```python
@app.post("/usuarios")
def crear_usuario(usuario: UsuarioEntrada) -> UsuarioSalida:
    return usuario
```

Desde FastAPI moderno, el type hint de retorno de la función cumple la misma función que `response_model=...` — ambos son válidos, `response_model` explícito es más claro cuando el tipo de retorno de Python difiere del esquema (por ejemplo, devuelves un objeto ORM en vez del `BaseModel` directamente).

## Excluir campos no seteados o `None`

```python
class Item(BaseModel):
    nombre: str
    descripcion: str | None = None
    tags: list[str] = []

@app.get("/items/{id}", response_model=Item, response_model_exclude_unset=True)
def obtener_item(id: int):
    return {"nombre": "Laptop"}
    # respuesta: {"nombre": "Laptop"}
    # NO: {"nombre": "Laptop", "descripcion": None, "tags": []}
```

- `response_model_exclude_unset=True` — omite campos que nunca se asignaron explícitamente (útiles en PATCH parciales).
- `response_model_exclude_none=True` — omite cualquier campo cuyo valor final sea `None`, sin importar si se seteó o no.
- `response_model_exclude={"campo1", "campo2"}` / `response_model_include={...}` — control explícito campo por campo.

## Códigos de estado y `responses={}` documentados

```python
from fastapi import status

@app.get(
    "/items/{id}",
    response_model=Item,
    responses={
        404: {"description": "Item no encontrado"},
        403: {"description": "Sin permisos para ver este item"},
    },
)
def obtener_item(id: int):
    ...
```

Esto no cambia el comportamiento en tiempo de ejecución — documenta en Swagger UI qué otros códigos de estado puede devolver el endpoint además del `200` implícito, algo que sin esto quedaría invisible para quien consuma la API.

## Devolver una `Response` directamente (bypass de `response_model`)

```python
from fastapi import Response
from fastapi.responses import JSONResponse, PlainTextResponse

@app.get("/items/{id}")
def obtener_item(id: int):
    if id == 0:
        return PlainTextResponse("ID inválido", status_code=400)
    return JSONResponse(content={"id": id}, status_code=200)
```

Cuando devuelves un objeto `Response` (o subclase) directamente, FastAPI **no** aplica `response_model` ni serialización automática — asumes control total de la respuesta. Útil para casos especiales (contenido no-JSON, headers custom) pero pierdes la validación/documentación automática en ese camino.

## Headers y cookies en la respuesta

```python
@app.get("/items/{id}")
def obtener_item(id: int, response: Response):
    response.headers["X-Version"] = "1.4.2"
    response.set_cookie(key="ultima_busqueda", value=str(id))
    return {"id": id}
```

Declarar un parámetro `response: Response` le da acceso a la función para modificar headers/cookies de la respuesta que FastAPI está construyendo, sin tener que devolver el objeto `Response` completo tú mismo (conservas `response_model` y serialización automática).

## Ver también

- [[03 - Request Body y Modelos Pydantic]]
- [[05 - Validación Avanzada con Pydantic]]
- [[07 - Manejo de Errores y Excepciones]]
