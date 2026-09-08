# 55 - Async Generators y Async Context Managers

> Continúa de [[54 - Asyncio en Profundidad|54]]. Ya se vieron generadores síncronos ([[35 - Generadores y Corutinas Clásicas|35]]) y context managers síncronos ([[50 - Context Managers Avanzados|50]]) — este archivo cubre sus contrapartes asíncronas, necesarias cuando la propia iteración o la propia entrada/salida de un recurso requiere `await`.

## `async for`: Iterando sobre Datos que Llegan de Forma Asíncrona

Un iterable asíncrono implementa `__aiter__`/`__anext__` en vez de `__iter__`/`__next__` — necesario cuando **obtener el siguiente elemento** requiere una operación de I/O (leer el siguiente chunk de una respuesta HTTP, la siguiente fila de una consulta a base de datos asíncrona).

```python
import asyncio

class FuenteDeDatosAsincrona:
    def __init__(self, total):
        self.total = total
        self.actual = 0

    def __aiter__(self):
        return self

    async def __anext__(self):
        if self.actual >= self.total:
            raise StopAsyncIteration
        await asyncio.sleep(0.5)   # simula esperar el siguiente dato (red, disco, etc.)
        self.actual += 1
        return self.actual

async def main():
    async for valor in FuenteDeDatosAsincrona(3):
        print(f"Recibido: {valor}")

asyncio.run(main())
```

## Generadores Asíncronos: la Forma Simple con `async def` + `yield`

Igual que un generador síncrono simplifica implementar `__iter__`/`__next__` a mano, un **generador asíncrono** simplifica `__aiter__`/`__anext__`:

```python
import asyncio

async def leer_lineas_de_stream(lineas_simuladas):
    for linea in lineas_simuladas:
        await asyncio.sleep(0.3)   # simula latencia de red al leer cada línea
        yield linea

async def main():
    async for linea in leer_lineas_de_stream(["línea 1", "línea 2", "línea 3"]):
        print(f"Procesando: {linea}")

asyncio.run(main())
```

Este patrón es exactamente lo que usan clientes HTTP asíncronos (`httpx`, `aiohttp`) para exponer streaming de respuestas grandes sin cargar todo en memoria de una vez — combinando la eficiencia de memoria de [[35 - Generadores y Corutinas Clásicas|35]] con la no-bloqueante de [[54 - Asyncio en Profundidad|54]].

### Comprensiones Asíncronas

```python
async def obtener_datos():
    return [1, 2, 3]

async def main():
    # comprensión de lista sobre un generador asíncrono
    resultados = [x async for x in leer_lineas_de_stream(["a", "b", "c"])]
    print(resultados)  # ['a', 'b', 'c']

    # 'await' dentro de una comprensión normal también es válido
    cuadrados = [x ** 2 for x in await obtener_datos()]
    print(cuadrados)  # [1, 4, 9]

asyncio.run(main())
```

## Async Context Managers: `__aenter__` / `__aexit__`

Ya se mostró la sintaxis básica en [[50 - Context Managers Avanzados|50]] — aquí, un caso realista: una conexión de red que necesita `await` tanto para abrirse como para cerrarse.

```python
import asyncio

class ConexionAsincrona:
    def __init__(self, host):
        self.host = host

    async def __aenter__(self):
        print(f"Conectando a {self.host}...")
        await asyncio.sleep(0.5)   # simula el handshake de red
        self.conectado = True
        return self

    async def __aexit__(self, tipo_exc, valor_exc, traceback):
        print(f"Cerrando conexión a {self.host}...")
        await asyncio.sleep(0.2)   # simula el cierre ordenado de la conexión
        self.conectado = False
        return False

async def main():
    async with ConexionAsincrona("api.ejemplo.com") as conexion:
        print(f"Conectado: {conexion.conectado}")

asyncio.run(main())
```

## `@asynccontextmanager`: la Versión "Con Generador" del Protocolo

```python
from contextlib import asynccontextmanager
import asyncio

@asynccontextmanager
async def sesion_db(url):
    print(f"Abriendo sesión hacia {url}")
    await asyncio.sleep(0.3)
    sesion = {"url": url, "activa": True}
    try:
        yield sesion
    finally:
        print("Cerrando sesión")
        await asyncio.sleep(0.1)
        sesion["activa"] = False

async def main():
    async with sesion_db("postgresql://localhost/mi_db") as sesion:
        print(f"Usando sesión: {sesion}")

asyncio.run(main())
```

Este es exactamente el patrón que usa FastAPI para el **lifespan** de la aplicación (`@asynccontextmanager` sobre una función que hace `yield` una vez, con setup antes y teardown en el `finally`) — ver `Python/FastAPI/14 - Routers y Estructura de Proyectos Grandes.md`.

## Combinando Todo: un Pipeline de Procesamiento Async Real

```python
import asyncio

async def obtener_registros(fuente):
    """Generador async que simula leer registros de una API paginada."""
    for pagina in range(3):
        await asyncio.sleep(0.3)
        for i in range(3):
            yield f"{fuente}-pagina{pagina}-registro{i}"

async def procesar_registro(registro):
    await asyncio.sleep(0.1)
    return registro.upper()

async def main():
    async with ConexionAsincrona("api.ejemplo.com"):
        tareas = []
        async for registro in obtener_registros("usuarios"):
            tareas.append(asyncio.create_task(procesar_registro(registro)))
        resultados = await asyncio.gather(*tareas)
        print(resultados)

asyncio.run(main())
```

Este pipeline combina: un context manager async para la conexión, un generador async para paginar resultados, y `Task`/`gather` (ver [[54 - Asyncio en Profundidad|54]]) para procesar cada registro concurrentemente — la composición típica de una aplicación async real (scraping, ETL asíncrono, consumo de APIs paginadas).

## Tabla de Equivalencias: Síncrono vs Asíncrono

| Concepto | Síncrono | Asíncrono |
|---|---|---|
| Definir función | `def` | `async def` |
| Esperar un resultado | (llamada directa) | `await` |
| Iterar | `__iter__`/`__next__`, `for` | `__aiter__`/`__anext__`, `async for` |
| Generador | `yield` | `async def` + `yield` |
| Context manager | `__enter__`/`__exit__`, `with` | `__aenter__`/`__aexit__`, `async with` |
| Decorador de contextlib | `@contextmanager` | `@asynccontextmanager` |
| Excepción de fin de iteración | `StopIteration` | `StopAsyncIteration` |

## Ver también

- [[54 - Asyncio en Profundidad]] — el event loop y `Task` sobre los que corre todo este archivo.
- [[35 - Generadores y Corutinas Clásicas]] — la base síncrona de generadores que este archivo extiende.
- [[50 - Context Managers Avanzados]] — la base síncrona de `with` que este archivo extiende.
- `Python/FastAPI/14 - Routers y Estructura de Proyectos Grandes.md` — `@asynccontextmanager` aplicado al lifespan de una aplicación real.
