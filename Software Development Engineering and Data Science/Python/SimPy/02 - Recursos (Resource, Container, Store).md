---
tags: [simpy, recursos, cheat-sheet]
---

# 02 — Recursos (`Resource`, `Container`, `Store`)

## `Resource` — capacidad limitada, unidades indistinguibles

El recurso más común: representa algo con un número fijo de "espacios" que los procesos solicitan y liberan (un cajero, una mesa, un puerto de carga).

```python
import simpy

def cliente(env, nombre, cajero):
    print(f"{nombre} llega en t={env.now}")
    with cajero.request() as solicitud:   # pide usar el recurso
        yield solicitud                    # espera hasta que haya un cajero libre
        print(f"{nombre} es atendido en t={env.now}")
        yield env.timeout(3)               # duración de la atención
        print(f"{nombre} termina en t={env.now}")
    # al salir del "with", el recurso se libera automáticamente

env = simpy.Environment()
cajero = simpy.Resource(env, capacity=1)   # un solo cajero disponible

for i in range(3):
    env.process(cliente(env, f"Cliente-{i}", cajero))

env.run()
```

El bloque `with recurso.request() as solicitud: yield solicitud` es el patrón estándar: garantiza que el recurso se libere aunque el proceso termine antes de tiempo o lance una excepción.

## Inspeccionar el estado de un `Resource`

```python
cajero.count       # cuántas unidades están en uso ahora mismo
cajero.capacity    # capacidad total
len(cajero.queue)  # cuántos procesos esperan en la cola
```

## `PriorityResource` — atención con prioridad

```python
cajero = simpy.PriorityResource(env, capacity=1)

with cajero.request(priority=1) as solicitud:   # menor número = mayor prioridad
    yield solicitud
    # ...
```

## `Container` — cantidad continua o a granel, no unidades discretas

Para modelar algo que se mide en cantidad (litros de combustible, unidades de inventario) en vez de "espacios" ocupables uno a la vez:

```python
tanque = simpy.Container(env, capacity=100, init=50)   # capacidad 100, empieza con 50

yield tanque.get(20)    # extraer 20 unidades (espera si no hay suficiente disponible)
yield tanque.put(10)    # agregar 10 unidades (espera si no cabe)

tanque.level   # cantidad actual
```

## `Store` — cola de objetos Python arbitrarios

Cuando lo que se comparte no es "capacidad" sino objetos concretos (piezas fabricadas, mensajes en una cola, tareas pendientes):

```python
almacen = simpy.Store(env, capacity=10)

yield almacen.put({"id": 1, "producto": "widget"})   # agregar un objeto
item = yield almacen.get()                            # obtener el siguiente objeto (FIFO por defecto)
```

`FilterStore` extiende `Store` permitiendo pedir un objeto que cumpla una condición específica en vez de tomar el siguiente de la cola sin más.

## Cuál usar

| Recurso | Cuándo usarlo |
|---|---|
| `Resource` | Unidades idénticas e indivisibles (servidores, cajeros, máquinas) |
| `PriorityResource` | Igual que `Resource`, pero con orden de atención por prioridad |
| `Container` | Cantidad continua/a granel (combustible, inventario en volumen) |
| `Store` / `FilterStore` | Objetos discretos con identidad propia (piezas, mensajes, tareas) |

## Ver también

- [[01 - Introducción y Conceptos Fundamentales]]
- [[03 - Modelado de Colas]]
