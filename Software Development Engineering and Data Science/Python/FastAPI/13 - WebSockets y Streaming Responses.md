---
tags: [fastapi, apis, websockets, python, cheat-sheet]
---

# 13 — WebSockets y Streaming Responses

> Continúa de [[12 - Testing con TestClient y Pytest]].

## WebSockets — comunicación bidireccional persistente

A diferencia de un endpoint HTTP normal (request → response → conexión cerrada), un **WebSocket** mantiene la conexión abierta y permite que cliente y servidor se envíen mensajes en cualquier momento, en cualquier dirección.

```python
from fastapi import WebSocket, WebSocketDisconnect

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    try:
        while True:
            datos = await websocket.receive_text()
            await websocket.send_text(f"Recibido: {datos}")
    except WebSocketDisconnect:
        print("Cliente desconectado")
```

`websocket.accept()` completa el handshake inicial; a partir de ahí, `receive_text()`/`send_text()` (o sus variantes `_json`, `_bytes`) intercambian mensajes mientras la conexión siga abierta. `WebSocketDisconnect` es la excepción que Starlette lanza cuando el cliente cierra la conexión — capturarla evita que quede como un error no manejado en los logs.

## Caso de uso típico: progreso en tiempo real

```python
@app.websocket("/ws/entrenamiento/{job_id}")
async def progreso_entrenamiento(websocket: WebSocket, job_id: str):
    await websocket.accept()
    try:
        while not job_terminado(job_id):
            porcentaje = obtener_progreso(job_id)
            await websocket.send_json({"job_id": job_id, "progreso": porcentaje})
            await asyncio.sleep(1)
        await websocket.send_json({"job_id": job_id, "progreso": 100, "estado": "completo"})
    except WebSocketDisconnect:
        pass
```

Útil para notificar avance de un entrenamiento de modelo, procesamiento de un dataset grande, o cualquier tarea larga donde hacer polling HTTP repetido sería más costoso que mantener una conexión abierta.

## Manejar múltiples conexiones — un gestor simple

```python
class GestorConexiones:
    def __init__(self):
        self.activas: list[WebSocket] = []

    async def conectar(self, websocket: WebSocket):
        await websocket.accept()
        self.activas.append(websocket)

    def desconectar(self, websocket: WebSocket):
        self.activas.remove(websocket)

    async def difundir(self, mensaje: str):
        for conexion in self.activas:
            await conexion.send_text(mensaje)

gestor = GestorConexiones()

@app.websocket("/ws/chat")
async def chat(websocket: WebSocket):
    await gestor.conectar(websocket)
    try:
        while True:
            mensaje = await websocket.receive_text()
            await gestor.difundir(mensaje)
    except WebSocketDisconnect:
        gestor.desconectar(websocket)
```

Para producción con múltiples workers/procesos (ver [[15 - Despliegue y Producción]]), este patrón en memoria no basta — cada worker tendría su propia lista de conexiones desconectada de las demás. Se necesita un broker compartido (Redis Pub/Sub es el más común) para difundir mensajes entre procesos.

## `StreamingResponse` — enviar datos en chunks sin WebSocket

```python
from fastapi.responses import StreamingResponse

def generar_csv():
    yield "id,nombre,precio\n"
    for item in obtener_items_de_bd_uno_por_uno():
        yield f"{item.id},{item.nombre},{item.precio}\n"

@app.get("/exportar")
def exportar_csv():
    return StreamingResponse(generar_csv(), media_type="text/csv")
```

A diferencia de WebSockets (bidireccional), `StreamingResponse` sigue siendo una respuesta HTTP normal de una sola dirección — útil para archivos grandes generados sobre la marcha (exports, reportes) sin acumular todo en memoria antes de responder.

## Streaming de respuestas de un LLM — patrón común hoy

```python
async def generar_tokens(prompt: str):
    async for chunk in cliente_llm.stream(prompt):
        yield chunk.texto

@app.post("/chat")
async def chat_endpoint(prompt: str):
    return StreamingResponse(generar_tokens(prompt), media_type="text/plain")
```

El mismo patrón de `StreamingResponse` es lo que permite que un frontend muestre la respuesta de un modelo de lenguaje "escribiéndose" token por token, en vez de esperar la respuesta completa.

## Ver también

- [[10 - Async y Concurrencia]]
- [[15 - Despliegue y Producción]]
- [[49-APIs-con-FastAPI-para-Servir-Modelos]]
