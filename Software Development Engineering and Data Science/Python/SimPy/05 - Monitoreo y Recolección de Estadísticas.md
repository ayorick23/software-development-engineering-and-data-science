---
tags: [simpy, estadisticas, cheat-sheet]
---

# 05 — Monitoreo y Recolección de Estadísticas

## El patrón básico: listas externas

SimPy no recolecta métricas automáticamente — el patrón estándar (ya usado en [[03 - Modelado de Colas]]) es declarar listas fuera de los procesos y hacerles `.append()` desde dentro de ellos:

```python
import simpy
import random

tiempos_de_espera = []
tiempos_en_sistema = []

def cliente(env, agentes):
    llegada = env.now
    with agentes.request() as solicitud:
        yield solicitud
        tiempos_de_espera.append(env.now - llegada)
        yield env.timeout(random.expovariate(1/4))
        tiempos_en_sistema.append(env.now - llegada)
```

## Monitorear el estado de un recurso a lo largo del tiempo

Para saber no solo promedios finales sino **cómo cambió la ocupación con el tiempo** (útil para identificar horas pico), se registra el estado en cada evento relevante:

```python
historial_ocupacion = []

def monitor_recurso(env, recurso, intervalo=1):
    while True:
        historial_ocupacion.append((env.now, recurso.count, len(recurso.queue)))
        yield env.timeout(intervalo)

env = simpy.Environment()
agentes = simpy.Resource(env, capacity=2)
env.process(monitor_recurso(env, agentes))
# ... el resto de los procesos ...
env.run(until=480)
```

`historial_ocupacion` queda como una lista de tuplas `(tiempo, en_uso, en_cola)` — directamente exportable a un `DataFrame` de pandas para análisis posterior.

## Convertir resultados a pandas para análisis

```python
import pandas as pd

df = pd.DataFrame(historial_ocupacion, columns=["tiempo", "en_uso", "en_cola"])
df["ocupacion_pct"] = df["en_uso"] / agentes.capacity * 100

df["en_cola"].describe()   # estadística descriptiva estándar, ver Python/Pandas
```

## Calcular métricas de negocio típicas al final de la simulación

```python
espera_promedio = sum(tiempos_de_espera) / len(tiempos_de_espera)
espera_maxima = max(tiempos_de_espera)
pct_esperaron_mas_de_5min = sum(1 for t in tiempos_de_espera if t > 5) / len(tiempos_de_espera) * 100

print(f"Espera promedio: {espera_promedio:.2f} min")
print(f"% que esperó más de 5 min: {pct_esperaron_mas_de_5min:.1f}%")
```

## Repetir la simulación para estimar variabilidad

Una sola corrida da un solo resultado, que puede no ser representativo por el azar de los tiempos aleatorios — el mismo principio de [[University Degree Notes/Ciclo 04/Estadística Inferencial/Clase 07 - Estimación e Intervalos de Confianza#Un Estimador es una Variable Aleatoria|un estimador como variable aleatoria]] visto en Estadística Inferencial aplica aquí: correr la simulación muchas veces con distintas semillas y reportar un intervalo, no un solo número.

```python
import random

resultados = []
for semilla in range(100):
    random.seed(semilla)
    tiempos_de_espera.clear()
    # ... reconstruir env y correr la simulación ...
    resultados.append(sum(tiempos_de_espera) / len(tiempos_de_espera))

import numpy as np
print(f"Espera promedio: {np.mean(resultados):.2f} ± {np.std(resultados):.2f} min (100 corridas)")
```

## Ver también

- [[03 - Modelado de Colas]]
- [[06 - Visualización de Resultados]]
- [[Estadística Aplicada/03 - Muestreo, TCL y Bootstrap en la Práctica]]
