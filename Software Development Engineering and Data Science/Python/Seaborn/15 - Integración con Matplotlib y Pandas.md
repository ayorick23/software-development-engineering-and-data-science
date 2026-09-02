---
tags: [seaborn, python, data-science, integrations, cheat-sheet]
---

# 15 — Integración con Matplotlib y Pandas

> Continúa de [[14 - La Nueva API Declarativa (seaborn.objects)]].

## Seaborn devuelve objetos de Matplotlib — siempre se puede seguir personalizando

```python
ax = sns.scatterplot(data=df, x="precio", y="demanda", hue="region")

ax.set_title("Relación Precio-Demanda")           # cualquier método de Axes de Matplotlib sigue disponible
ax.set_xlabel("Precio (USD)")
ax.axhline(y=100, color="red", linestyle="--")       # mezclar libremente elementos de Matplotlib puro
```

Toda función **axes-level** (`scatterplot`, `lineplot`, `boxplot`, `histplot`, `heatmap`...) devuelve un `Axes` de Matplotlib normal — ver la arquitectura completa en [[Python/Matplotlib/01 - Introducción y Arquitectura|Python/Matplotlib]]. No hay ninguna limitación real después de graficar con Seaborn: todo lo de este cheat-sheet de Matplotlib sigue aplicando.

## Insertar un gráfico de Seaborn en subplots propios de Matplotlib

```python
fig, axes = plt.subplots(1, 2, figsize=(12, 5))

sns.scatterplot(data=df, x="precio", y="demanda", ax=axes[0])
sns.histplot(data=df, x="ventas", ax=axes[1])

fig.suptitle("Panel de análisis exploratorio")
fig.tight_layout()
```

El argumento `ax=` es la pieza clave para combinar varios gráficos de Seaborn (funciones axes-level) dentro de una cuadrícula de subplots creada manualmente con `plt.subplots()` (ver [[Python/Matplotlib/03 - Creación de Figuras y Subplots|Python/Matplotlib]]) — las funciones **figure-level** (`relplot`, `catplot`...) no aceptan `ax=` porque administran su propia `Figure` completa (ver [[01 - Introducción y Arquitectura]]).

## Guardar una figura de Seaborn

```python
fig = ax.get_figure()          # desde un Axes (funciones axes-level)
fig.savefig("grafico.png", dpi=300, bbox_inches="tight")

g = sns.relplot(data=df, x="precio", y="demanda", col="region")   # funciones figure-level devuelven un FacetGrid
g.savefig("grafico_facetado.png", dpi=300)                          # FacetGrid tiene su propio .savefig(), atajo sobre .fig.savefig()
```

Ver el catálogo completo de formatos/opciones de exportación en [[Python/Matplotlib/14 - Guardado y Exportación|Python/Matplotlib]] — aplica igual sobre figuras generadas por Seaborn.

## Pandas — el input nativo de toda la API

```python
sns.scatterplot(data=df, x="precio", y="demanda", hue="region")     # 'df' es un DataFrame; los strings son nombres de COLUMNA
sns.lineplot(data=df.set_index("fecha")["ventas"])                    # también acepta una Series directamente
```

El parámetro `data=` combinado con nombres de columna como strings (en vez de pasar arrays sueltos) es la firma característica de toda la API de Seaborn — diseñada explícitamente para recibir un DataFrame de Pandas y referenciar sus columnas por nombre, sin extraer arrays manualmente primero. Ver [[Python/Pandas/01 - Introducción y Arquitectura Interna|Python/Pandas]].

## El flujo típico completo: Pandas → Seaborn → Matplotlib

```python
resumen = (
    df
    .groupby(["region", "mes"])["ventas"]
    .sum()
    .reset_index()
)                                                    # 1. preparar datos con Pandas (ver Python/Pandas/08 y 10)

fig, ax = plt.subplots(figsize=(10, 6))
sns.lineplot(data=resumen, x="mes", y="ventas", hue="region", ax=ax)   # 2. graficar con Seaborn

ax.set_title("Ventas mensuales por región")            # 3. pulir con Matplotlib
ax.legend(loc="upper left", bbox_to_anchor=(1, 1))
fig.savefig("ventas_mensuales.png", dpi=300, bbox_inches="tight")     # 4. exportar
```

Este es, en la práctica, el flujo estándar de todo el trío Pandas+Seaborn+Matplotlib: transformar/agregar con Pandas, graficar la relación estadística con Seaborn, pulir detalles finos y exportar con Matplotlib.

## Ver también

- [[14 - La Nueva API Declarativa (seaborn.objects)]]
- [[16 - Rendimiento y Datasets Grandes]]
- [[Python/Matplotlib/01 - Introducción y Arquitectura|Python/Matplotlib]]
- [[Python/Pandas/01 - Introducción y Arquitectura Interna|Python/Pandas]]
