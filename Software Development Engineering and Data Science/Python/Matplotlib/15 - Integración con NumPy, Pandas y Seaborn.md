---
tags: [matplotlib, python, data-science, integrations, cheat-sheet]
---

# 15 — Integración con NumPy, Pandas y Seaborn

> Continúa de [[14 - Guardado y Exportación]].

## NumPy — el input nativo de toda la API

```python
x = np.linspace(0, 10, 100)
y = np.sin(x)
ax.plot(x, y)                    # Matplotlib espera arrays (o listas); NumPy es la forma nativa e idiomática
```

Toda función de graficación de Matplotlib acepta `ndarray`s directamente y opera eficientemente sobre ellos — ver la estructura subyacente en [[Python/NumPy/01 - Introducción y Arquitectura Interna|Python/NumPy]].

## Pandas — el método `.plot()`

```python
df["ventas"].plot(kind="line")                     # Series.plot() — wrapper directo sobre Matplotlib
df.plot(x="fecha", y="ventas", kind="bar")            # DataFrame.plot()
df.groupby("region")["monto"].sum().plot(kind="pie", autopct="%.1f%%")

fig, ax = plt.subplots()
df.plot(x="fecha", y="ventas", ax=ax)                  # pasar un Axes explícito — necesario para combinar con subplots propios
ax.set_title("Ventas por fecha")                          # seguir personalizando con la API normal de Matplotlib después
```

`df.plot(ax=ax)` es la pieza clave para combinar la conveniencia de Pandas con el control total de Matplotlib: Pandas dibuja los datos, pero el `Axes` resultante sigue siendo un objeto Matplotlib normal, personalizable con todo lo visto en este cheat-sheet. Ver [[Python/Pandas/16 - Integración con el Ecosistema|Python/Pandas]].

## Graficar directamente un DataFrame completo

```python
df[["ventas", "costos", "utilidad"]].plot(subplots=True, figsize=(10, 8))   # un subplot por columna, automático
df.plot.scatter(x="precio", y="demanda", c="region_cod", cmap="viridis")       # scatter con color por categoría codificada
```

## Seaborn — construido encima de Matplotlib

```python
import seaborn as sns

fig, ax = plt.subplots(figsize=(8, 5))
sns.barplot(data=df, x="region", y="monto", ax=ax)    # Seaborn también acepta 'ax=' explícito
ax.set_title("Monto por región")                          # se sigue personalizando con Matplotlib normal
```

Toda función de Seaborn devuelve (o acepta) un `Axes` de Matplotlib — Seaborn agrega una capa de estadística y estética por default sobre Matplotlib, pero **no** reemplaza su motor de renderizado. Esto significa que cualquier técnica de este cheat-sheet (anotaciones, ejes gemelos, colormaps custom, `savefig`) funciona exactamente igual sobre una figura generada con Seaborn.

## Cuándo usar Matplotlib puro vs Pandas `.plot()` vs Seaborn

| | Matplotlib puro | Pandas `.plot()` | Seaborn |
|---|---|---|---|
| Control | Máximo, pero más verboso | Medio — rápido para exploración | Medio-alto, con defaults estadísticos listos |
| Mejor para | Gráficos custom, publicación, dashboards | Exploración rápida directo desde un DataFrame | Gráficos estadísticos (distribuciones, regresión, categóricas) con estética cuidada por default |
| Curva de aprendizaje | Más alta | Mínima si ya se conoce Pandas | Baja-media |

**Regla práctica:** usar Pandas `.plot()`/Seaborn para exploración rápida y primeras versiones; caer al `Axes` de Matplotlib subyacente (`ax=`) en cuanto se necesite personalización fina — nunca hace falta elegir "uno u otro" de forma excluyente.

## Ver también

- [[14 - Guardado y Exportación]]
- [[16 - Rendimiento y Grandes Volúmenes de Datos]]
- [[Python/NumPy/01 - Introducción y Arquitectura Interna|Python/NumPy]]
- [[Python/Pandas/16 - Integración con el Ecosistema|Python/Pandas]]
