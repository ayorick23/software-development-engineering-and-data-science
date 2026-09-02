---
tags: [matplotlib, python, data-science, visualization, animation, cheat-sheet]
---

# 13 — Animaciones

> Continúa de [[12 - Gráficos 3D]].

## `FuncAnimation` — el mecanismo estándar de animación

```python
from matplotlib.animation import FuncAnimation

fig, ax = plt.subplots()
x = np.linspace(0, 2 * np.pi, 100)
linea, = ax.plot(x, np.sin(x))     # la coma es importante: plot() devuelve una LISTA, se desempaqueta el único elemento

def actualizar(frame):
    linea.set_ydata(np.sin(x + frame * 0.1))    # modifica los datos del artist existente, NO crea uno nuevo
    return linea,

anim = FuncAnimation(fig, actualizar, frames=100, interval=50, blit=True)
plt.show()
```

`FuncAnimation` llama repetidamente a la función `actualizar()`, una vez por frame — la clave de rendimiento es **modificar los datos de un `Artist` ya existente** (`linea.set_ydata(...)`) en vez de volver a llamar `ax.plot()` en cada frame (lo que acumularía líneas nuevas y degradaría el rendimiento progresivamente).

## `blit=True` — renderizar solo lo que cambió

```python
anim = FuncAnimation(fig, actualizar, frames=100, interval=50, blit=True)
```

*Blitting* le dice a Matplotlib que solo redibuje los `Artists` que la función `actualizar()` devuelve (no la figura completa en cada frame) — mejora notablemente el rendimiento de la animación, pero requiere que `actualizar()` devuelva explícitamente una tupla/lista de los artists modificados y que la función `init()` (si se usa) prepare el estado inicial correctamente.

## Animación con `init_func` explícita

```python
def inicializar():
    linea.set_ydata(np.full_like(x, np.nan))    # estado inicial vacío antes del primer frame real
    return linea,

anim = FuncAnimation(fig, actualizar, frames=100, init_func=inicializar, interval=50, blit=True)
```

## Guardar una animación como archivo

```python
anim.save("animacion.gif", writer="pillow", fps=20)
anim.save("animacion.mp4", writer="ffmpeg", fps=30, dpi=150)     # requiere ffmpeg instalado en el sistema
```

`writer="pillow"` no requiere dependencias externas y es suficiente para GIFs; `writer="ffmpeg"` produce mejor compresión/calidad para video pero necesita el binario `ffmpeg` disponible en el sistema.

## Animación de dispersión (actualizar posiciones de puntos)

```python
scatter = ax.scatter([], [])

def actualizar(frame):
    x_actual = np.random.rand(50)
    y_actual = np.random.rand(50)
    scatter.set_offsets(np.column_stack([x_actual, y_actual]))    # actualiza posiciones de TODOS los puntos a la vez
    return scatter,

anim = FuncAnimation(fig, actualizar, frames=50, interval=100, blit=True)
```

`set_offsets()` es el método específico para reposicionar los puntos de un `scatter()` ya creado — no existe un `set_xdata`/`set_ydata` directo para scatter como sí lo hay para `plot()`.

## Cuándo NO usar animación

Para exploración de datos y reportes estáticos, una animación rara vez comunica mejor que un buen gráfico estático o un pequeño grid de subplots mostrando distintos momentos en el tiempo — reservar animaciones para presentaciones en vivo, contenido para redes/video, o cuando el propio *cambio a través del tiempo* es el mensaje central (no solo un efecto visual).

## Ver también

- [[12 - Gráficos 3D]]
- [[14 - Guardado y Exportación]]
- [[17 - Buenas Prácticas, Errores Comunes y Comparativa]]
