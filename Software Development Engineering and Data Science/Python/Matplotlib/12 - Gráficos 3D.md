---
tags: [matplotlib, python, data-science, visualization, 3d, cheat-sheet]
---

# 12 — Gráficos 3D

> Continúa de [[11 - Mapas de Calor, Imágenes y Colorbars]].

## Crear un `Axes` 3D

```python
fig = plt.figure(figsize=(8, 6))
ax = fig.add_subplot(projection="3d")     # el argumento clave: projection="3d"
```

Toda la funcionalidad 3D de Matplotlib vive en el submódulo `mpl_toolkits.mplot3d`, pero desde versiones recientes se activa simplemente pasando `projection="3d"` a `add_subplot()`, sin necesidad de un import adicional explícito en la mayoría de los casos.

## Dispersión 3D

```python
x = np.random.rand(100)
y = np.random.rand(100)
z = np.random.rand(100)

ax.scatter(x, y, z, c=z, cmap="viridis")
ax.set_xlabel("X")
ax.set_ylabel("Y")
ax.set_zlabel("Z")
```

## Líneas 3D

```python
theta = np.linspace(0, 4 * np.pi, 100)
z = np.linspace(0, 2, 100)
x = np.cos(theta)
y = np.sin(theta)

ax.plot(x, y, z)      # una hélice 3D
```

## Superficies 3D

```python
x = np.linspace(-5, 5, 50)
y = np.linspace(-5, 5, 50)
X, Y = np.meshgrid(x, y)
Z = np.sin(np.sqrt(X**2 + Y**2))

superficie = ax.plot_surface(X, Y, Z, cmap="viridis", edgecolor="none")
fig.colorbar(superficie, ax=ax, shrink=0.5)
```

`plot_surface()` requiere una malla `meshgrid` — la misma técnica usada en `pcolormesh()`/`contourf()` (ver [[11 - Mapas de Calor, Imágenes y Colorbars]]), aplicada ahora en el eje Z como altura en vez de como color.

## Wireframe (solo la malla, sin relleno)

```python
ax.plot_wireframe(X, Y, Z, color="black", linewidth=0.5)
```

Más ligero de renderizar que `plot_surface()` — útil para visualizar la estructura de la malla sin el costo de sombreado/color de una superficie sólida, especialmente con datasets grandes.

## Controlar el ángulo de vista

```python
ax.view_init(elev=30, azim=45)     # elevación y azimut en grados
```

## Cuándo (no) usar 3D

Los gráficos 3D en una pantalla plana son notoriamente difíciles de leer con precisión — la profundidad se pierde en la proyección 2D, y comparar valores exactos entre dos puntos es mucho más difícil que en un gráfico 2D equivalente (ej. un `pcolormesh` con colorbar). **Regla práctica:** usar 3D solo cuando la forma tridimensional en sí es el punto central del mensaje (una superficie física real, una nube de puntos genuinamente 3D) — para la mayoría de datos con 3 variables, un scatter 2D con color como tercera dimensión (ver [[04 - Gráficos de Líneas y Dispersión#scatter() con color y tamaño variable por punto (cuarta y quinta dimensión)|scatter con color]]) comunica mejor. Ver más en [[17 - Buenas Prácticas, Errores Comunes y Comparativa]].

## Ver también

- [[11 - Mapas de Calor, Imágenes y Colorbars]]
- [[04 - Gráficos de Líneas y Dispersión]]
- [[17 - Buenas Prácticas, Errores Comunes y Comparativa]]
