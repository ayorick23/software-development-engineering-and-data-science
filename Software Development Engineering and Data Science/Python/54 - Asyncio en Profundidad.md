# 54 - Asyncio en Profundidad

> Continúa de [[53 - Fechas y Zonas Horarias|53]] y abre el bloque de concurrencia y asincronía avanzada. [[23 - Concurrency|23]] presentó `asyncio` como una de las tres opciones de concurrencia en Python; [[35 - Generadores y Corutinas Clásicas|35]] mostró la base de "pausar y reanudar" con `yield`. Aquí se profundiza en el modelo real de `asyncio`: el event loop, `Task`, y patrones de uso correctos.

## `async`/`await`: la Sintaxis, Explicada

```python
import asyncio

async def saludar(nombre, segundos):
    print(f"Iniciando saludo a {nombre}")
    await asyncio.sleep(segundos)   # cede el control al event loop mientras "espera"
    print(f"Hola, {nombre}!")

asyncio.run(saludar("Ana", 1))
```

Una función `async def` es una **corutina**: al llamarla, no ejecuta nada — devuelve un objeto corutina que debe ser **esperado** (`await`) o **programado** como tarea para realmente ejecutarse.

```python
async def main():
    corutina = saludar("Luis", 1)
    print(type(corutina))   # <class 'coroutine'> -> todavía no se ejecutó nada
    await corutina           # AHORA se ejecuta

asyncio.run(main())
```

## El Event Loop: Cómo Funciona por Debajo

```mermaid
sequenceDiagram
    participant Loop as Event Loop
    participant T1 as Tarea A
    participant T2 as Tarea B

    Loop->>T1: ejecuta hasta el primer await
    T1-->>Loop: cede el control (esperando I/O)
    Loop->>T2: ejecuta hasta su primer await
    T2-->>Loop: cede el control (esperando I/O)
    Note over Loop: I/O de Tarea A completa
    Loop->>T1: reanuda desde donde se quedó
    T1-->>Loop: termina
    Note over Loop: I/O de Tarea B completa
    Loop->>T2: reanuda desde donde se quedó
    T2-->>Loop: termina
```

El event loop ejecuta **una sola corutina a la vez** (single-threaded, como se explicó en [[23 - Concurrency|23]]), pero cambia entre corutinas cada vez que una hace `await` sobre algo que implica esperar (I/O de red, disco, un timer). Esto da la **ilusión** de concurrencia sin paralelismo real de CPU.

## Ejecutando Varias Corutinas Concurrentemente: `gather` vs Secuencial

```python
import asyncio
import time

async def tarea(nombre, segundos):
    await asyncio.sleep(segundos)
    return f"{nombre} completada"

async def secuencial():
    inicio = time.perf_counter()
    r1 = await tarea("A", 1)
    r2 = await tarea("B", 1)
    r3 = await tarea("C", 1)
    print(f"Secuencial: {time.perf_counter() - inicio:.2f}s")  # ~3 segundos

async def concurrente():
    inicio = time.perf_counter()
    resultados = await asyncio.gather(
        tarea("A", 1),
        tarea("B", 1),
        tarea("C", 1),
    )
    print(f"Concurrente: {time.perf_counter() - inicio:.2f}s")  # ~1 segundo
    print(resultados)  # ['A completada', 'B completada', 'C completada']

asyncio.run(secuencial())
asyncio.run(concurrente())
```

`await` **uno tras otro** (secuencial) desperdicia la ventaja de `asyncio` — el punto es lanzar varias esperas de I/O **simultáneamente** con `gather`, para que el tiempo total sea el de la operación más lenta, no la suma de todas.

## `Task`: Programando Corutinas para que Corran en Segundo Plano

```python
import asyncio

async def descargar(url):
    print(f"Descargando {url}...")
    await asyncio.sleep(2)
    return f"Contenido de {url}"

async def main():
    tarea = asyncio.create_task(descargar("http://ejemplo.com"))  # empieza a ejecutarse YA
    print("Haciendo otra cosa mientras se descarga...")
    resultado = await tarea   # espera a que termine, si no había terminado ya
    print(resultado)

asyncio.run(main())
```

`asyncio.gather()` internamente crea `Task`s — la diferencia es que `create_task` te da el control explícito: puedes lanzar la tarea, hacer otro trabajo, y solo esperarla (`await`) cuando realmente necesites su resultado.

## `TaskGroup` (3.11+): la Forma Moderna y Segura de Manejar Múltiples Tareas

```python
import asyncio

async def procesar_pedido(id_pedido):
    await asyncio.sleep(1)
    if id_pedido == 3:
        raise ValueError(f"Pedido {id_pedido} inválido")
    return f"Pedido {id_pedido} procesado"

async def main():
    try:
        async with asyncio.TaskGroup() as tg:
            tareas = [tg.create_task(procesar_pedido(i)) for i in range(5)]
    except* ValueError as eg:   # ver 49 - Excepciones Avanzadas para 'except*'
        for error in eg.exceptions:
            print(f"Error: {error}")

    for t in tareas:
        if not t.cancelled() and t.exception() is None:
            print(t.result())

asyncio.run(main())
```

`TaskGroup` mejora sobre `gather`: si **una** tarea falla, las demás se **cancelan automáticamente** (evitando fugas de tareas huérfanas), y todos los errores se agrupan en un `ExceptionGroup` en vez de perder los errores de las tareas que no fallaron primero.

## Cancelación de Tareas

```python
import asyncio

async def tarea_larga():
    try:
        await asyncio.sleep(10)
    except asyncio.CancelledError:
        print("La tarea fue cancelada, limpiando recursos...")
        raise   # SIEMPRE re-lanzar CancelledError, salvo que sepas exactamente por qué no

async def main():
    tarea = asyncio.create_task(tarea_larga())
    await asyncio.sleep(1)
    tarea.cancel()
    try:
        await tarea
    except asyncio.CancelledError:
        print("Confirmado: la tarea terminó cancelada")

asyncio.run(main())
```

`cancel()` inyecta un `CancelledError` en el punto donde la corutina está pausada — el mismo mecanismo conceptual que `generador.throw()` en [[35 - Generadores y Corutinas Clásicas|35]].

## `asyncio.timeout` (3.11+): Límites de Tiempo Sin Lógica Manual

```python
import asyncio

async def operacion_lenta():
    await asyncio.sleep(10)
    return "completado"

async def main():
    try:
        async with asyncio.timeout(2):
            resultado = await operacion_lenta()
    except TimeoutError:
        print("La operación excedió el tiempo límite")

asyncio.run(main())
```

## El Error Más Común: Mezclar Código Bloqueante con `async`

```python
import asyncio
import time

async def MAL_bloqueante():
    time.sleep(2)   # BLOQUEA todo el event loop -- ninguna otra tarea puede avanzar
    return "listo"

async def BIEN_no_bloqueante():
    await asyncio.sleep(2)  # cede el control real al event loop
    return "listo"
```

```python
# Para código bloqueante que NO se puede evitar (una librería sync, cómputo CPU-intensivo),
# ejecutarlo en un thread/proceso aparte sin congelar el event loop:
async def envolver_bloqueante():
    resultado = await asyncio.to_thread(time.sleep, 2)
    return resultado
```

Mezclar un driver de base de datos síncrono, `requests` (en vez de `httpx`/`aiohttp`), o cualquier operación de CPU pesada directamente dentro de una función `async def` **bloquea todo el event loop**, anulando cualquier ventaja de concurrencia — es el error de rendimiento más común en aplicaciones async mal implementadas (ver también `Python/FastAPI/10 - Async y Concurrencia.md`).

## `asyncio` vs Threading vs Multiprocessing: Revisitado

| | `asyncio` | `threading` | `multiprocessing` |
|---|---|---|---|
| Ideal para | Miles de conexiones I/O concurrentes | Decenas de operaciones I/O concurrentes | Cómputo CPU-intensivo paralelo real |
| Requiere reescribir código con `async`/`await` | Sí | No | No |
| Overhead por unidad de concurrencia | Muy bajo | Bajo-medio | Alto |
| Comparte memoria fácilmente | Sí (single-thread) | Sí (con locks) | No (requiere IPC) |

Ver [[56 - Sincronización y Modelos de Concurrencia|56]] y [[57 - Paralelismo Real|57]] para profundizar en las otras dos opciones.

## Ver también

- [[23 - Concurrency]] — la introducción al GIL y las tres opciones de concurrencia de Python.
- [[35 - Generadores y Corutinas Clásicas]] — la base de "pausar y reanudar" sobre la que se construye `async`/`await`.
- [[55 - Async Generators y Async Context Managers]] — extendiendo `async`/`await` a iteradores y `with`.
- [[49 - Excepciones Avanzadas]] — `except*` y `ExceptionGroup`, usados por `TaskGroup`.
