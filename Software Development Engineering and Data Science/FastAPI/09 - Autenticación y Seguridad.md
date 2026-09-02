---
tags: [fastapi, apis, seguridad, python, cheat-sheet]
---

# 09 — Autenticación y Seguridad

> Continúa de [[08 - Middleware y CORS]].

## API Key — el mecanismo más simple

```python
from fastapi import Security, HTTPException, status
from fastapi.security import APIKeyHeader

API_KEYS_VALIDAS = {"clave-secreta-123"}
api_key_header = APIKeyHeader(name="X-API-Key")

def verificar_api_key(api_key: str = Security(api_key_header)) -> str:
    if api_key not in API_KEYS_VALIDAS:
        raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="API Key inválida")
    return api_key

@app.get("/datos-protegidos", dependencies=[Depends(verificar_api_key)])
def datos_protegidos():
    return {"secreto": 42}
```

Suficiente para APIs internas o de servicio-a-servicio; insuficiente para autenticación de usuarios individuales (no hay noción de "quién es" más allá de "tiene la clave").

## OAuth2 con Password Flow — login de usuarios

```python
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.post("/token")
def login(form_data: Annotated[OAuth2PasswordRequestForm, Depends()]):
    usuario = autenticar_usuario(form_data.username, form_data.password)
    if not usuario:
        raise HTTPException(status_code=401, detail="Usuario o contraseña incorrectos")
    token = crear_access_token(data={"sub": usuario.username})
    return {"access_token": token, "token_type": "bearer"}

@app.get("/perfil")
def ver_perfil(token: Annotated[str, Depends(oauth2_scheme)]):
    usuario = decodificar_token(token)
    return usuario
```

`OAuth2PasswordRequestForm` espera un body `application/x-www-form-urlencoded` con `username` y `password` (el estándar de OAuth2, no JSON) — Swagger UI incluso muestra un botón "Authorize" que usa este flujo automáticamente una vez declarado.

## JWT — generar y verificar el token

```python
from datetime import datetime, timedelta, timezone
from jose import jwt, JWTError   # pip install python-jose[cryptography]

SECRET_KEY = "usar-una-clave-real-desde-variables-de-entorno"
ALGORITHM = "HS256"

def crear_access_token(data: dict, expira_en_minutos: int = 30) -> str:
    datos = data.copy()
    expiracion = datetime.now(timezone.utc) + timedelta(minutes=expira_en_minutos)
    datos.update({"exp": expiracion})
    return jwt.encode(datos, SECRET_KEY, algorithm=ALGORITHM)

def decodificar_token(token: str = Depends(oauth2_scheme)) -> dict:
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise HTTPException(status_code=401, detail="Token inválido")
        return {"username": username}
    except JWTError:
        raise HTTPException(
            status_code=401,
            detail="No se pudo validar el token",
            headers={"WWW-Authenticate": "Bearer"},
        )
```

La clave `SECRET_KEY` **nunca** debe vivir en el código fuente en producción — cargarla desde variables de entorno (ver `pydantic-settings` en [[15 - Despliegue y Producción]]).

## Dependencia reutilizable: "usuario actual"

```python
def obtener_usuario_actual(payload: Annotated[dict, Depends(decodificar_token)]) -> Usuario:
    usuario = buscar_usuario_en_bd(payload["username"])
    if usuario is None:
        raise HTTPException(status_code=401, detail="Usuario no encontrado")
    return usuario

def requiere_usuario_activo(usuario: Annotated[Usuario, Depends(obtener_usuario_actual)]) -> Usuario:
    if not usuario.activo:
        raise HTTPException(status_code=400, detail="Usuario inactivo")
    return usuario

@app.get("/mis-items")
def mis_items(usuario: Annotated[Usuario, Depends(requiere_usuario_activo)]):
    return buscar_items_de(usuario.id)
```

Este patrón de sub-dependencias (ver [[06 - Dependency Injection]]) es lo que hace que "requerir un usuario autenticado y activo" sea una sola línea (`Depends(requiere_usuario_activo)`) en cualquier endpoint que lo necesite.

## Scopes — permisos granulares dentro de OAuth2

```python
oauth2_scheme = OAuth2PasswordBearer(
    tokenUrl="token",
    scopes={"items:leer": "Leer items", "items:escribir": "Crear o modificar items"},
)

from fastapi.security import SecurityScopes

def verificar_scopes(security_scopes: SecurityScopes, token: Annotated[str, Depends(oauth2_scheme)]):
    payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
    scopes_token = payload.get("scopes", [])
    for scope in security_scopes.scopes:
        if scope not in scopes_token:
            raise HTTPException(status_code=403, detail=f"Falta el permiso: {scope}")
    return payload

@app.post("/items", dependencies=[Security(verificar_scopes, scopes=["items:escribir"])])
def crear_item(): ...
```

Útil cuando distintos endpoints requieren distintos niveles de permiso dentro del mismo sistema de autenticación (lectura vs escritura, admin vs usuario normal), en vez de un simple "autenticado sí/no".

## Hashing de contraseñas — nunca almacenarlas en texto plano

```python
from passlib.context import CryptContext   # pip install passlib[bcrypt]

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hashear_password(password: str) -> str:
    return pwd_context.hash(password)

def verificar_password(password_plano: str, password_hasheado: str) -> bool:
    return pwd_context.verify(password_plano, password_hasheado)
```

## Ver también

- [[06 - Dependency Injection]]
- [[07 - Manejo de Errores y Excepciones]]
- [[15 - Despliegue y Producción]]
