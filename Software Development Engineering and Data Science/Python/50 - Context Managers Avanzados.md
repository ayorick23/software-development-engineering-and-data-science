# 50 - Context Managers Avanzados

> Continúa de [[49 - Excepciones Avanzadas|49]]. [[15 - Context Manager|15]] cubrió `with` y `@contextlib.contextmanager` a nivel básico; aquí se profundiza en el manejo fino de excepciones dentro de `__exit__`, en `ExitStack` para gestionar un número dinámico de recursos, y en la versión asíncrona del protocolo.

## Repaso: Qué Resuelve `with`

`with` garantiza que un recurso se libere **siempre**, incluso si ocurre una excepción a mitad del bloque — el patrón RAII (_Resource Acquisition Is Initialization_) mencionado en [[41 - Patrones de Diseño Pythonicos|41]].

```python
class ConexionDB:
    def __enter__(self):
        print("Abriendo conexión")
        return self

    def __exit__(self, tipo_exc, valor_exc, traceback):
        print("Cerrando conexión")
        return False  # no suprime la excepción

with ConexionDB() as conexion:
    print("Usando la conexión")
    raise ValueError("algo salió mal")
# Salida:
# Abriendo conexión
# Usando la conexión
# Cerrando conexión   <- se ejecuta SIEMPRE, incluso con la excepción
# ValueError: algo salió mal   <- se propaga después de cerrar
```

## El Valor de Retorno de `__exit__`: Suprimir o Propagar

```python
class IgnorarErroresDeRed:
    def __enter__(self):
        return self

    def __exit__(self, tipo_exc, valor_exc, traceback):
        if tipo_exc is ConnectionError:
            print(f"Error de red ignorado: {valor_exc}")
            return True   # True SUPRIME la excepción -- el programa continúa normalmente
        return False       # False/None: la excepción se propaga sin cambios

with IgnorarErroresDeRed():
    raise ConnectionError("timeout")
print("El programa continúa aquí, la excepción fue suprimida")
```

| Valor de retorno de `__exit__` | Efecto |
|---|---|
| `False` o `None` (o no retornar nada) | La excepción se propaga normalmente |
| `True` | La excepción se **suprime** — el bloque `with` termina como si nada hubiera pasado |

> **Precaución:** suprimir excepciones silenciosamente es peligroso si se hace de forma demasiado amplia (`return True` sin verificar `tipo_exc`) — puede ocultar bugs reales. Siempre verifica explícitamente qué tipo de excepción estás dispuesto a suprimir.

## `contextlib.ExitStack`: Gestionando un Número Dinámico de Recursos

`ExitStack` resuelve el problema de abrir una cantidad **variable** de recursos (no se sabe de antemano cuántos `with` anidar):

```python
from contextlib import ExitStack

def procesar_archivos(rutas: list[str]):
    with ExitStack() as stack:
        archivos = [stack.enter_context(open(ruta)) for ruta in rutas]
        # todos los archivos se cierran automáticamente al salir del 'with',
        # sin importar cuántos sean ni si alguno falla a mitad de la lista
        for archivo in archivos:
            print(archivo.readline())

procesar_archivos(["a.txt", "b.txt", "c.txt"])
```

```python
# Comparación: sin ExitStack, anidar un número dinámico de 'with' es engorroso
def procesar_dos_archivos_manual(ruta1, ruta2):
    with open(ruta1) as f1:
        with open(ruta2) as f2:
            print(f1.readline(), f2.readline())
    # ¿y si fueran 10 archivos? Anidar 10 'with' a mano no escala.
```

`ExitStack` también permite registrar funciones de limpieza arbitrarias, no solo context managers:

```python
with ExitStack() as stack:
    stack.callback(print, "Limpieza 1 ejecutada")
    stack.callback(print, "Limpieza 2 ejecutada")
    print("Trabajo principal")
# Trabajo principal
# Limpieza 2 ejecutada   <- se ejecutan en orden INVERSO (LIFO), como una pila
# Limpieza 1 ejecutada
```

## `@contextmanager` Anidados y con Manejo de Excepciones

```python
from contextlib import contextmanager

@contextmanager
def transaccion(conexion):
    print("BEGIN")
    try:
        yield conexion
        conexion.commit()
        print("COMMIT")
    except Exception:
        conexion.rollback()
        print("ROLLBACK")
        raise   # re-lanza la excepción original después de hacer rollback

class ConexionFalsa:
    def commit(self): pass
    def rollback(self): pass

try:
    with transaccion(ConexionFalsa()):
        print("Ejecutando consultas...")
        raise RuntimeError("Error en la consulta")
except RuntimeError:
    print("Excepción capturada fuera del context manager")

# BEGIN
# Ejecutando consultas...
# ROLLBACK
# Excepción capturada fuera del context manager
```

El bloque `try`/`except`/`raise` dentro de una función generadora decorada con `@contextmanager` es exactamente donde `__exit__` recibiría la excepción en una implementación basada en clase — la excepción se "inyecta" en el punto del `yield`.

## Context Managers Múltiples en una Sola Línea

```python
# Desde Python 3.10, se pueden agrupar varios context managers con paréntesis
with (
    open("entrada.txt") as entrada,
    open("salida.txt", "w") as salida,
):
    salida.write(entrada.read().upper())

# Equivalente en versiones anteriores (funciona igual, ambos se abren/cierran correctamente)
with open("entrada.txt") as entrada, open("salida.txt", "w") as salida:
    salida.write(entrada.read().upper())
```

## Async Context Managers: `async with`

```python
class ConexionAsincrona:
    async def __aenter__(self):
        print("Abriendo conexión async")
        return self

    async def __aexit__(self, tipo_exc, valor_exc, traceback):
        print("Cerrando conexión async")
        return False

async def main():
    async with ConexionAsincrona() as conexion:
        print("Usando conexión")

import asyncio
asyncio.run(main())
```

Se profundiza en `__aenter__`/`__aexit__` y `@asynccontextmanager` como parte del bloque de asincronía en [[55 - Async Generators y Async Context Managers|55]].

## Otras Utilidades de `contextlib`

```python
from contextlib import redirect_stdout, nullcontext
import io

# redirect_stdout: captura temporalmente todo lo que se imprime con print()
buffer = io.StringIO()
with redirect_stdout(buffer):
    print("Esto no aparece en consola")
print(buffer.getvalue())  # "Esto no aparece en consola\n"

# nullcontext: un context manager "vacío", útil para código condicional
def procesar(datos, usar_transaccion=True):
    gestor = transaccion(conexion) if usar_transaccion else nullcontext(conexion)
    with gestor as conn:
        ...  # el código no necesita ramificarse en dos bloques 'with' distintos
```

## Ver también

- [[15 - Context Manager]] — la base de `with` y `@contextmanager` que este archivo extiende.
- [[49 - Excepciones Avanzadas]] — el manejo de excepciones que interactúa con `__exit__`.
- [[55 - Async Generators y Async Context Managers]] — la versión `async with` de todo lo visto aquí.
- [[24 - Databases]] — transacciones de base de datos como caso de uso real de context managers.
