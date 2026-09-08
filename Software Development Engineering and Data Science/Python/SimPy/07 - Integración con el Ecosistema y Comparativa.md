---
tags: [simpy, ecosistema, cheat-sheet]
---

# 07 — Integración con el Ecosistema y Comparativa

## Con NumPy/SciPy para muestrear distribuciones más allá de la Exponencial

Aunque `random.expovariate` alcanza para casos simples, `scipy.stats` da acceso a distribuciones más ricas y a la misma API consistente vista en [[Python/SciPy/04 - Estadística I - Distribuciones (scipy.stats)|SciPy]]:

```python
from scipy import stats

# Tiempos de servicio con Gamma en vez de Exponencial pura (más realista cuando el servicio
# tiene una duración "típica" con menos variabilidad extrema que la Exponencial)
tiempo_servicio = stats.gamma(a=2, scale=2).rvs()
```

## Con pandas para analizar resultados

Como se vio en [[05 - Monitoreo y Recolección de Estadísticas]], el resultado natural de una simulación (una lista de tiempos, una lista de tuplas de estado) se convierte directamente en un `DataFrame` para su análisis con las herramientas ya cubiertas en [[Python/Pandas/01 - Introducción y Arquitectura Interna|Pandas]].

## Con Streamlit para un dashboard interactivo de "qué pasaría si"

Un caso de uso natural: envolver la simulación en una app de [[Python/Streamlit/01 - Introducción y Arquitectura|Streamlit]] donde un usuario de negocio mueve un slider (número de agentes, tasa de llegada) y ve el resultado de la simulación en tiempo real, sin tocar código.

```python
import streamlit as st

num_agentes = st.slider("Número de agentes", 1, 10, 2)
espera = correr_simulacion(num_agentes)
st.metric("Espera promedio (min)", f"{espera:.2f}")
```

## SimPy vs Monte Carlo simple

No son excluyentes, pero resuelven preguntas distintas:

| | SimPy (DES) | Simulación Monte Carlo simple |
|---|---|---|
| Modela | Un sistema con estado, tiempo y recursos compartidos | Un cálculo probabilístico repetido, sin "reloj" ni estado compartido |
| Ejemplo típico | Colas, líneas de producción, dimensionamiento de personal | Estimar un intervalo de confianza remuestreando (ver [[Python/SciPy/05 - Estadística II - Pruebas de Hipótesis (scipy.stats)#Bootstrap: intervalos de confianza sin supuestos de distribución|bootstrap]]) |
| Complejidad | Modela interacción entre entidades a lo largo del tiempo | Repite un cálculo independiente muchas veces |

## SimPy vs herramientas de simulación dedicadas (Arena, AnyLogic, Simio)

SimPy es código puro (más flexible, gratuito, versionable con Git, integrable con todo el stack de Python), mientras que las herramientas comerciales ofrecen interfaz visual de arrastrar y soltar y animación 2D/3D lista para usar. Para prototipos rápidos, análisis reproducible, e integración con pipelines de datos existentes, SimPy es la opción más natural en un stack ya basado en Python; para presentaciones ejecutivas con animación visual del proceso, las herramientas comerciales tienen ventaja.

## Ver también

- [[01 - Introducción y Conceptos Fundamentales]]
- [[Python/SciPy/01 - Introducción y Arquitectura]]
- [[Estadística Aplicada/03 - Muestreo, TCL y Bootstrap en la Práctica]]
