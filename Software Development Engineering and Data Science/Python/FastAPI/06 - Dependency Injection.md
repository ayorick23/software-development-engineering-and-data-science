---
tags: [fastapi, apis, python, cheat-sheet]
---

# 06 — Dependency Injection

> Continúa de [[05 - Validación Avanzada con Pydantic]].

## El problema que resuelve `Depends`

Sin inyección de dependencias, lógica repetida (verificar autenticación, abrir una conexión a BD, extraer parámetros comunes) se copia y pega en cada endpoint. `Depends()` declara esa lógica **una vez** y FastAPI se encarga de ejecutarla y pasar su resultado a cualquier endpoint que la declare.

## Dependencia como función simple

```python
from fastapi import Depends

def parametros_comunes(skip: int = 0, limit: int = 10, q: str | None = None):
    return {"skip": skip, "limit": limit, "q": q}

@app.get("/items")
def listar_items(comunes: Annotated[dict, Depends(parametros_comunes)]):
    return comunes

@app.get("/usuarios")
def listar_usuarios(comunes: Annotated[dict, Depends(parametros_comunes)]):
    return comunes
```

Ambos endpoints comparten la misma lógica de paginación/búsqueda sin duplicar la firma de parámetros — cambiar `parametros_comunes` una vez afecta a todos los endpoints que dependen de ella.

## Dependencia como clase — cuando necesitas más estructura

```python
class ParametrosPaginacion:
    def __init__(self, skip: int = 0, limit: int = 10):
        self.skip = skip
        self.limit = limit

@app.get("/items")
def listar_items(paginacion: Annotated[ParametrosPaginacion, Depends()]):
    return {"skip": paginacion.skip, "limit": paginacion.limit}
```

`Depends()` sin argumentos, cuando el tipo ya está anotado, usa la clase misma como "fábrica" — FastAPI la instancia extrayendo los parámetros del request igual que con una función.

## Sub-dependencias — dependencias que dependen de otras

```python
def obtener_token(authorization: str = Header(...)) -> str:
    return authorization.replace("Bearer ", "")

def obtener_usuario_actual(token: Annotated[str, Depends(obtener_token)]) -> dict:
    usuario = verificar_token_y_buscar_usuario(token)
    if not usuario:
        raise HTTPException(status_code=401, detail="Token inválido")
    return usuario

@app.get("/perfil")
def ver_perfil(usuario: Annotated[dict, Depends(obtener_usuario_actual)]):
    return usuario
```

FastAPI resuelve la cadena completa: para ejecutar `ver_perfil`, primero ejecuta `obtener_usuario_actual`, que a su vez primero ejecuta `obtener_token` — cada nivel puede reutilizarse independientemente en otros endpoints.

## Dependencias con `yield` — setup y cleanup garantizado

```python
def obtener_db():
    db = SessionLocal()
    try:
        yield db          # esto es lo que recibe el endpoint
    finally:
        db.close()          # se ejecuta SIEMPRE, incluso si el endpoint lanza una excepción

@app.get("/items")
def listar_items(db: Annotated[Session, Depends(obtener_db)]):
    return db.query(Item).all()
```

Este es el patrón estándar para conexiones a bases de datos (ver [[11 - Bases de Datos con SQLAlchemy y SQLModel]]): todo lo antes del `yield` es setup, todo lo después (en el `finally`) es cleanup, garantizado por FastAPI sin importar cómo termine la request.

## Dependencias a nivel de router o de toda la app

```python
# aplica a un solo endpoint, sin usar su valor de retorno
@app.get("/items", dependencies=[Depends(verificar_api_key)])
def listar_items(): ...

# aplica a TODOS los endpoints de un router
router = APIRouter(dependencies=[Depends(verificar_api_key)])

# aplica a TODA la aplicación
app = FastAPI(dependencies=[Depends(verificar_api_key)])
```

Cuando una dependencia solo verifica algo (lanza excepción si falla) pero su valor de retorno no se usa dentro del endpoint, `dependencies=[Depends(...)]` evita tener que declarar un parámetro que nunca se usa.

## `Depends` con caché — comportamiento por defecto

```python
def dependencia_costosa():
    print("ejecutándose...")
    return calcular_algo_caro()

@app.get("/ejemplo")
def endpoint(
    a: Annotated[int, Depends(dependencia_costosa)],
    b: Annotated[int, Depends(dependencia_costosa)],
):
    # "ejecutándose..." se imprime UNA sola vez, no dos
    # FastAPI cachea el resultado dentro del mismo request
    return {"a": a, "b": b}
```

Si necesitas que se ejecute de nuevo cada vez dentro del mismo request, usa `Depends(dependencia_costosa, use_cache=False)`.

## Ver también

- [[07 - Manejo de Errores y Excepciones]]
- [[09 - Autenticación y Seguridad]]
- [[11 - Bases de Datos con SQLAlchemy y SQLModel]]
