---
tags: [simpy, interrupciones, cheat-sheet]
---

# 04 — Interrupciones y Procesos Condicionales

## Interrumpir un proceso en curso

Útil para modelar eventos que cortan una actividad antes de que termine naturalmente — una máquina que se descompone a mitad de un trabajo, un agente que recibe una llamada urgente.

```python
import simpy

def trabajo(env):
    try:
        print(f"Trabajo inicia en t={env.now}")
        yield env.timeout(10)
        print(f"Trabajo termina normalmente en t={env.now}")
    except simpy.Interrupt as interrupcion:
        print(f"Trabajo interrumpido en t={env.now}: {interrupcion.cause}")

def agente_externo(env, proceso_objetivo):
    yield env.timeout(4)
    proceso_objetivo.interrupt(cause="falla de máquina")

env = simpy.Environment()
p = env.process(trabajo(env))
env.process(agente_externo(env, p))
env.run()

# Trabajo inicia en t=0
# Trabajo interrumpido en t=4: falla de máquina
```

El proceso interrumpido debe capturar `simpy.Interrupt` con un `try/except` alrededor del `yield` — sin eso, la excepción se propaga y detiene el proceso sin oportunidad de reaccionar (por ejemplo, reprogramar el trabajo restante).

## Esperar el primero de varios eventos: `AnyOf`

Modelar "lo que ocurra primero" — por ejemplo, un cliente que se va si espera demasiado (abandono/reneging), una situación común en modelos de colas realistas:

```python
def cliente_impaciente(env, agentes, tiempo_maximo_espera):
    with agentes.request() as solicitud:
        resultado = yield solicitud | env.timeout(tiempo_maximo_espera)

        if solicitud in resultado:
            print(f"Atendido en t={env.now}")
            yield env.timeout(5)
        else:
            print(f"Cliente se fue sin ser atendido en t={env.now} (esperó demasiado)")
```

El operador `|` entre dos eventos (`solicitud | env.timeout(...)`) crea una condición "lo que pase primero" — si el timeout ocurre antes de que el recurso quede libre, el cliente abandona.

## Esperar todos los eventos: `AllOf`

```python
evento1 = env.timeout(5)
evento2 = env.timeout(3)

yield evento1 & evento2   # espera hasta que AMBOS hayan ocurrido (el más lento manda)
```

## Procesos que se reprograman a sí mismos

Un patrón común: un proceso de mantenimiento que se repite periódicamente mientras la simulación corre:

```python
def mantenimiento_periodico(env, maquina):
    while True:
        yield env.timeout(100)   # cada 100 unidades de tiempo
        print(f"Mantenimiento de {maquina} en t={env.now}")
        yield env.timeout(5)     # duración del mantenimiento
```

## Ver también

- [[01 - Introducción y Conceptos Fundamentales]]
- [[03 - Modelado de Colas]]
