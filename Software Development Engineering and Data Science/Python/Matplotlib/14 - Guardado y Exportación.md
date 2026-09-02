---
tags: [matplotlib, python, data-science, visualization, export, cheat-sheet]
---

# 14 — Guardado y Exportación

> Continúa de [[13 - Animaciones]].

## `savefig()` — lo básico

```python
fig.savefig("grafico.png")
fig.savefig("grafico.png", dpi=300)                # resolución — 300 DPI es estándar para impresión/publicación
fig.savefig("grafico.svg")                            # vectorial — escala sin perder calidad, ideal para diagramas
fig.savefig("grafico.pdf")                              # vectorial, ideal para incluir en documentos LaTeX/reportes
```

| Formato | Tipo | Cuándo usarlo |
|---|---|---|
| `.png` | Raster (píxeles) | Web, presentaciones, uso general |
| `.jpg` | Raster con compresión con pérdida | Fotografías reales, NO para gráficos con texto/líneas finas |
| `.svg` | Vectorial | Diagramas, íconos, cuando se necesita editar después en Illustrator/Inkscape |
| `.pdf` | Vectorial | Reportes académicos, papers, documentos impresos |

**Regla práctica:** para gráficos con líneas, texto y formas geométricas (la inmensa mayoría de gráficos de datos), preferir formatos vectoriales (`.svg`, `.pdf`) o `.png` en alta resolución — nunca `.jpg`, cuya compresión con pérdida introduce artefactos visibles alrededor de líneas y texto nítido.

## Controlar el recorte y los márgenes

```python
fig.savefig("grafico.png", bbox_inches="tight")            # recorta espacio en blanco sobrante alrededor de la figura
fig.savefig("grafico.png", bbox_inches="tight", pad_inches=0.1)   # con un pequeño margen controlado
```

`bbox_inches="tight"` es casi siempre deseable — sin él, Matplotlib a veces deja márgenes grandes o corta leyendas/etiquetas que quedaron fuera del área original de la figura (especialmente con `legend(bbox_to_anchor=...)`, ver [[09 - Texto, Anotaciones y Leyendas]]).

## Fondo transparente

```python
fig.savefig("grafico.png", transparent=True)     # sin fondo blanco — para superponer sobre otro fondo (slides, web)
```

## Guardar en memoria (sin escribir a disco) — útil para servir vía API/web

```python
import io

buffer = io.BytesIO()
fig.savefig(buffer, format="png", dpi=150, bbox_inches="tight")
buffer.seek(0)
# 'buffer' ahora contiene los bytes de la imagen PNG, listos para enviar en una respuesta HTTP
```

Este patrón es el que se usa para servir un gráfico generado dinámicamente desde un endpoint de [[04 - Response Models y Serialización|FastAPI]] sin necesidad de escribir un archivo temporal en disco.

## Resolución consistente vía `rcParams`

```python
plt.rcParams["savefig.dpi"] = 300
plt.rcParams["savefig.bbox"] = "tight"
```

Fijar estos valores una vez en `rcParams` (ver [[10 - Estilos y rcParams]]) evita repetir `dpi=300, bbox_inches="tight"` en cada llamada individual a `savefig()` a lo largo de un notebook con muchos gráficos.

## Cerrar figuras para liberar memoria

```python
fig.savefig("grafico.png")
plt.close(fig)          # libera la memoria de la figura — importante en loops que generan MUCHOS gráficos
```

Generar decenas/cientos de figuras en un loop (por ejemplo, un gráfico por cliente en un reporte automatizado) sin cerrar cada una acumula memoria progresivamente hasta agotar RAM — `plt.close(fig)` inmediatamente después de guardar cada una es la práctica correcta en ese escenario.

## Ver también

- [[13 - Animaciones]]
- [[10 - Estilos y rcParams]]
- [[04 - Response Models y Serialización|FastAPI]]
- [[06 - Gráficos y Visualización|Streamlit]]
