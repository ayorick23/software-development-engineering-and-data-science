---
tags: [simpy, visualizacion, cheat-sheet]
---

# 06 — Visualización de Resultados

> Para la sintaxis completa de Matplotlib, ver [[Python/Matplotlib/01 - Introducción y Arquitectura]]. Este archivo cubre solo los gráficos más útiles específicamente para resultados de simulación.

## Histograma de tiempos de espera

El primer gráfico que conviene hacer con cualquier resultado de simulación — muestra la forma completa de la distribución, no solo el promedio:

```python
import matplotlib.pyplot as plt

fig, ax = plt.subplots()
ax.hist(tiempos_de_espera, bins=30, color="steelblue", edgecolor="white")
ax.set_xlabel("Tiempo de espera (min)")
ax.set_ylabel("Frecuencia")
ax.set_title("Distribución de tiempos de espera")
plt.show()
```

Es habitual que el promedio "se vea bien" pero el histograma revele una cola larga de clientes con esperas muy superiores — información que un solo número (la media) esconde.

## Ocupación del recurso a lo largo del tiempo

Usando el `historial_ocupacion` recolectado en [[05 - Monitoreo y Recolección de Estadísticas]]:

```python
import pandas as pd

df = pd.DataFrame(historial_ocupacion, columns=["tiempo", "en_uso", "en_cola"])

fig, ax = plt.subplots()
ax.plot(df["tiempo"], df["en_cola"], label="Clientes en cola")
ax.plot(df["tiempo"], df["en_uso"], label="Agentes ocupados")
ax.set_xlabel("Tiempo simulado (min)")
ax.legend()
plt.show()
```

Este gráfico revela patrones que un promedio agregado no muestra: por ejemplo, si la cola crece progresivamente sin bajar (el sistema está subdimensionado y colapsa con el tiempo) en vez de oscilar alrededor de un nivel estable.

## Comparar configuraciones (número de agentes) en un solo gráfico

```python
resultados_por_n = {1: [...], 2: [...], 3: [...], 4: [...]}   # de correr_simulacion() varias veces por cada n

fig, ax = plt.subplots()
ax.boxplot(resultados_por_n.values(), tick_labels=resultados_por_n.keys())
ax.set_xlabel("Número de agentes")
ax.set_ylabel("Espera promedio (min)")
plt.show()
```

Un boxplot por configuración comunica de un vistazo tanto el promedio como la variabilidad de cada escenario — la base visual para justificar una decisión de dimensionamiento de personal ante un tomador de decisiones no técnico.

## Ver también

- [[05 - Monitoreo y Recolección de Estadísticas]]
- [[Python/Matplotlib/01 - Introducción y Arquitectura]]
