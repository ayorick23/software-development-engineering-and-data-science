---
tags: [simpy, colas, cheat-sheet]
---

# 03 — Modelado de Colas

## Ejemplo completo: un call center con agentes limitados

Este ejemplo integra lo visto en los dos archivos anteriores: llegadas aleatorias ([[01 - Introducción y Conceptos Fundamentales|Introducción]]) compitiendo por un recurso limitado ([[02 - Recursos (Resource, Container, Store)|Resource]]).

```python
import simpy
import random

TIEMPO_ENTRE_LLEGADAS = 2   # media, en minutos (Exponencial)
TIEMPO_ATENCION = 4         # media, en minutos (Exponencial)
NUM_AGENTES = 2

tiempos_de_espera = []

def cliente(env, nombre, agentes):
    llegada = env.now
    with agentes.request() as solicitud:
        yield solicitud
        espera = env.now - llegada
        tiempos_de_espera.append(espera)

        yield env.timeout(random.expovariate(1 / TIEMPO_ATENCION))

def generador_de_clientes(env, agentes):
    i = 0
    while True:
        yield env.timeout(random.expovariate(1 / TIEMPO_ENTRE_LLEGADAS))
        i += 1
        env.process(cliente(env, f"Cliente-{i}", agentes))

env = simpy.Environment()
agentes = simpy.Resource(env, capacity=NUM_AGENTES)
env.process(generador_de_clientes(env, agentes))
env.run(until=480)   # simular 8 horas (480 minutos)

print(f"Clientes atendidos: {len(tiempos_de_espera)}")
print(f"Espera promedio: {sum(tiempos_de_espera) / len(tiempos_de_espera):.2f} min")
```

## El patrón "generador de llegadas + proceso de atención"

Casi toda simulación de colas sigue esta misma estructura de dos funciones: una que genera llegadas indefinidamente a intervalos aleatorios, y otra que describe qué le pasa a cada llegada individual al competir por un recurso compartido. Una vez interiorizado este patrón, se adapta a la mayoría de escenarios de colas (bancos, hospitales, líneas de producción, centros de llamadas).

## Comparar escenarios: el verdadero valor de la simulación

El propósito real no es simular una sola vez, sino **comparar configuraciones**:

```python
def correr_simulacion(num_agentes, duracion=480):
    tiempos_de_espera.clear()
    env = simpy.Environment()
    agentes = simpy.Resource(env, capacity=num_agentes)
    env.process(generador_de_clientes(env, agentes))
    env.run(until=duracion)
    return sum(tiempos_de_espera) / len(tiempos_de_espera)

for n in [1, 2, 3, 4]:
    espera = correr_simulacion(n)
    print(f"Con {n} agentes: espera promedio = {espera:.2f} min")
```

Esto responde directamente la pregunta de dimensionamiento de personal ("¿cuántos agentes necesito para que la espera promedio no supere X minutos?") sin necesidad de experimentar con el sistema real.

## Modelar rutas y transferencias entre etapas

Un modelo más realista de un proceso con varias etapas encadena recursos: el cliente pasa por una etapa, y solo entonces solicita el siguiente recurso.

```python
def cliente_multietapa(env, nombre, recepcion, especialista):
    with recepcion.request() as sol1:
        yield sol1
        yield env.timeout(random.expovariate(1/2))   # atención en recepción

    with especialista.request() as sol2:
        yield sol2
        yield env.timeout(random.expovariate(1/10))  # atención especializada
```

## Ver también

- [[01 - Introducción y Conceptos Fundamentales]]
- [[04 - Interrupciones y Procesos Condicionales]]
- [[05 - Monitoreo y Recolección de Estadísticas]]
