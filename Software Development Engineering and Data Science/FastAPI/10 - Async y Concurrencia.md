---
tags: [fastapi, apis, async, python, cheat-sheet]
---

# 10 — Async y Concurrencia

> Continúa de [[09 - Autenticación y Seguridad]].

## `async def` vs `def` — la decisión más malentendida de FastAPI

```python
@app.get("/rapido")
async def endpoint_async():
    resultado = await llamar_api_externa_async()   # no bloquea el event loop mientras espera
    return resultado

@app.get("/tambien-rapido")
def endpoint_sync():
    resultado = llamar_libreria_sincrona()   # FastAPI lo ejecuta en un threadpool aparte
    return resultado
```

**Ambos funcionan correctamente** — la diferencia está en qué pasa por debajo:

- Un endpoint `async def` corre directamente en el event loop principal. Si dentro de él haces una llamada **bloqueante** sin `await` (una librería síncrona pesada, un `time.sleep()`, un query de SQLAlchemy clásico), **bloqueas todo el event loop** — ninguna otra petición se procesa mientras tanto, aunque sean miles.
- Un endpoint `def` normal, FastAPI lo ejecuta automáticamente en un **threadpool** separado del event loop — es seguro poner ahí código bloqueante, a costa de usar un hilo del pool en vez de aprovechar la concurrencia nativa de asyncio.

## La regla práctica

| Situación | Usar |
|---|---|
| Llamas a una librería con soporte `async` nativo (httpx, asyncpg, motor) | `async def` + `await` |
| Llamas a una librería síncrona (requests, SQLAlchemy clásico, pandas, la mayoría del ecosistema ML) | `def` normal — FastAPI la aísla en threadpool automáticamente |
| Haces cómputo pesado de CPU (no I/O) — inferencia de un modelo grande, procesamiento de imágenes | `def` normal (el threadpool no libera el GIL para CPU-bound, pero al menos no bloquea el event loop de otras requests) |
| No estás seguro | `def` normal es la opción segura por defecto |

**El error más común**: declarar `async def` y luego llamar código síncrono bloqueante sin `await` dentro — eso combina lo peor de ambos mundos (bloquea el loop, sin ganar nada del async).

```mermaid
flowchart TD
    A["¿Tu endpoint hace I/O?"] -->|Sí, con librería async| B["async def + await"]
    A -->|Sí, con librería síncrona\n(requests, SQLAlchemy clásico)| C["def normal\n→ threadpool automático"]
    A -->|No, es cómputo puro\n(CPU-bound)| C
```

## `BackgroundTasks` — trabajo después de responder

```python
from fastapi import BackgroundTasks

def enviar_email_confirmacion(email: str, mensaje: str):
    # esto corre DESPUÉS de que la respuesta ya se envió al cliente
    time.sleep(2)   # simula latencia de un servicio de email real
    print(f"Email enviado a {email}: {mensaje}")

@app.post("/registrar")
def registrar_usuario(email: str, background_tasks: BackgroundTasks):
    crear_usuario_en_bd(email)
    background_tasks.add_task(enviar_email_confirmacion, email, "¡Bienvenido!")
    return {"mensaje": "Usuario registrado"}   # el cliente recibe esto de inmediato
```

El cliente recibe la respuesta sin esperar a que el email se envíe — ideal para trabajo que no necesita bloquear la respuesta (logging, notificaciones, limpieza). Para trabajo pesado o que debe sobrevivir un reinicio del servidor, `BackgroundTasks` **no es suficiente** — se necesita una cola real (Celery, RQ, Arq).

## Drivers async para bases de datos

```python
# síncrono (bloquea, corre en threadpool si el endpoint es `def`)
import psycopg2

# asíncrono (no bloquea, requiere `async def` + `await`)
import asyncpg

async def obtener_usuario(user_id: int):
    conn = await asyncpg.connect(DATABASE_URL)
    fila = await conn.fetchrow("SELECT * FROM usuarios WHERE id = $1", user_id)
    await conn.close()
    return fila
```

Mezclar un driver síncrono dentro de un endpoint `async def` es el error de rendimiento más común en APIs FastAPI mal implementadas — ver [[11 - Bases de Datos con SQLAlchemy y SQLModel]] para el patrón correcto con `Depends` + `yield` en ambos casos.

## `httpx` — el cliente HTTP recomendado (síncrono y async)

```python
import httpx

# uso síncrono
def consultar_api_externa():
    with httpx.Client() as client:
        return client.get("https://api.externa.com/datos").json()

# uso asíncrono
async def consultar_api_externa_async():
    async with httpx.AsyncClient() as client:
        respuesta = await client.get("https://api.externa.com/datos")
        return respuesta.json()
```

`httpx` reemplaza a `requests` cuando el proyecto es async — tiene la misma API familiar pero con soporte nativo para `async`/`await`, y es la misma librería que usa `TestClient` internamente (ver [[12 - Testing con TestClient y Pytest]]).

## Ver también

- [[06 - Dependency Injection]]
- [[11 - Bases de Datos con SQLAlchemy y SQLModel]]
- [[01 - Introducción y Arquitectura]]
