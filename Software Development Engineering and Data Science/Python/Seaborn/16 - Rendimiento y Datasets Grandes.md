---
tags: [seaborn, python, data-science, visualization, performance, cheat-sheet]
---

# 16 — Rendimiento y Datasets Grandes

> Continúa de [[15 - Integración con Matplotlib y Pandas]].

## El costo oculto de la estadística automática

```python
# Con 1 millón de filas, esto recalcula un bootstrap de 1000 remuestreos POR CADA barra
sns.barplot(data=df_grande, x="region", y="ventas")
```

La conveniencia de [[13 - Estadística Integrada|estimadores e intervalos de confianza automáticos]] tiene un costo real de cómputo: cada barra/línea con banda de confianza implica un bootstrap de miles de remuestreos — en datasets de millones de filas esto puede tardar notablemente más que una agregación manual con Pandas seguida de un gráfico simple.

```python
# Más rápido con datasets grandes: agregar UNA VEZ con Pandas, graficar el resultado ya resumido
resumen = df_grande.groupby("region")["ventas"].agg(["mean", "std"]).reset_index()
fig, ax = plt.subplots()
ax.bar(resumen["region"], resumen["mean"], yerr=resumen["std"])     # Matplotlib puro, sin recalcular nada internamente
```

## `stripplot`/`swarmplot` con demasiados puntos

```python
# LENTO y visualmente saturado con más de unos pocos miles de puntos por categoría
sns.swarmplot(data=df_grande, x="region", y="ventas")

# ALTERNATIVAS más apropiadas para volumen alto
sns.boxenplot(data=df_grande, x="region", y="ventas")          # diseñado específicamente para datasets grandes (ver 06)
sns.violinplot(data=df_grande, x="region", y="ventas")           # densidad agregada, no un punto por observación
sns.stripplot(data=df_grande.sample(2000), x="region", y="ventas")  # o simplemente graficar una MUESTRA representativa
```

`swarmplot()` calcula posiciones para evitar superposición entre **todos** los puntos — ese cálculo escala mal; con datasets grandes, muestrear antes de graficar (`df.sample(n)`) es a menudo la solución más simple y honesta (una muestra aleatoria sigue representando fielmente la distribución real).

## `scatterplot` con muchos puntos: transparencia y tamaño

```python
sns.scatterplot(data=df_grande, x="precio", y="demanda", alpha=0.1, s=10)   # transparencia + puntos pequeños revela densidad real
```

Con decenas de miles de puntos, un scatter sin ajustar se vuelve una mancha sólida donde no se distingue dónde hay más o menos concentración — bajar `alpha` (transparencia) permite que las zonas con más puntos superpuestos se vean visualmente más oscuras/saturadas, aproximando un mapa de densidad sin cambiar de tipo de gráfico.

## Cuándo migrar a un histograma 2D o KDE en vez de scatter

```python
# En vez de un scatter saturado con 500,000 puntos:
sns.kdeplot(data=df_grande, x="precio", y="demanda", fill=True)      # densidad conjunta — comunica mejor la concentración real
```

Ver el catálogo completo de KDE bivariado en [[04 - Gráficos de Distribución#KDE bivariado — densidad conjunta de dos variables|Gráficos de Distribución]] — a partir de cierto volumen de puntos, una representación de densidad agregada comunica el patrón real mejor que miles de puntos individuales superpuestos.

## Reducir el costo de `pairplot()` en datasets con muchas columnas

```python
# Lento con muchas columnas numéricas (n² subplots) y muchas filas
sns.pairplot(df_grande)

# Más rápido: limitar columnas Y muestrear filas
sns.pairplot(df_grande.sample(1000), vars=["precio", "demanda", "stock"])
```

Ver la advertencia de rendimiento original en [[10 - Gráficos Multivariados#pairplot() — todas las relaciones por pares de un vistazo|Gráficos Multivariados]].

## Ver también

- [[15 - Integración con Matplotlib y Pandas]]
- [[13 - Estadística Integrada]]
- [[Python/Matplotlib/16 - Rendimiento y Grandes Volúmenes de Datos|Python/Matplotlib]]
- [[Python/Pandas/15 - Rendimiento y Optimización|Python/Pandas]]
