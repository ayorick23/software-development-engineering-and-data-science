---
tags: [streamlit, dashboards, widgets, cheat-sheet]
---

# 03 — Widgets de Entrada

> Continúa de [[02 - Elementos de Texto y Markdown]].

## El patrón universal de un widget

```python
valor = st.widget_lo_que_sea("Etiqueta visible", ...)
```

Todo widget de Streamlit **retorna el valor actual** de la interacción del usuario — no hay callbacks obligatorios para leer el valor (aunque existen, ver [[09 - Forms, Callbacks y Control de Reruns]]). El patrón más común es simplemente usar el valor de retorno directamente en el flujo del script.

## `st.button` — acción puntual

```python
if st.button("Entrenar modelo"):
    st.write("Entrenando...")
```

`st.button` retorna `True` **solo durante el rerun inmediatamente posterior al clic** — en el siguiente rerun (cualquier otra interacción), vuelve a `False`. Esto sorprende a quien empieza con Streamlit: no hay un "estado de botón presionado" persistente sin usar `session_state` explícitamente (ver [[07 - Session State - Estado entre Reruns]]).

## `st.checkbox`

```python
mostrar_detalle = st.checkbox("Mostrar detalle técnico", value=False)
if mostrar_detalle:
    st.write("Información adicional...")
```

## `st.radio` — selección única entre pocas opciones visibles

```python
modo = st.radio("Modo de visualización", ["Diario", "Semanal", "Mensual"], horizontal=True)
```

## `st.selectbox` — selección única en un dropdown

```python
region = st.selectbox("Región", options=["RD", "PR", "MX"], index=0)
```

```python
# Con formato personalizado para el texto mostrado (útil cuando el valor interno no es legible):
region = st.selectbox(
    "Región",
    options=["RD", "PR", "MX"],
    format_func=lambda x: {"RD": "República Dominicana", "PR": "Puerto Rico", "MX": "México"}[x],
)
```

## `st.multiselect` — selección múltiple

```python
regiones = st.multiselect("Regiones a comparar", options=["RD", "PR", "MX"], default=["RD"])
```

Retorna una **lista** de los valores seleccionados — vacía si no se selecciona nada, a menos que se especifique `default`.

## `st.slider` — rangos numéricos o de fecha

```python
dias_atras = st.slider("Días hacia atrás", min_value=7, max_value=365, value=90, step=7)

# Slider de RANGO (dos extremos, retorna una tupla):
rango_precio = st.slider("Rango de precio", 0.0, 1000.0, (100.0, 500.0))
minimo, maximo = rango_precio
```

```python
import datetime

fecha_rango = st.slider(
    "Rango de fechas",
    min_value=datetime.date(2026, 1, 1),
    max_value=datetime.date(2026, 12, 31),
    value=(datetime.date(2026, 1, 1), datetime.date(2026, 6, 30)),
)
```

## `st.select_slider` — slider sobre valores discretos no numéricos

```python
nivel = st.select_slider("Nivel de confianza", options=["Bajo", "Medio", "Alto"], value="Medio")
```

## `st.text_input` / `st.text_area`

```python
nombre = st.text_input("Nombre del experimento", placeholder="ej. xgboost-90d-window")
descripcion = st.text_area("Notas", height=150)

password = st.text_input("Contraseña", type="password")   # oculta el texto ingresado
```

## `st.number_input`

```python
n_estimators = st.number_input("n_estimators", min_value=100, max_value=1000, value=300, step=50)
```

## `st.date_input` / `st.time_input`

```python
fecha = st.date_input("Fecha de corte", value=datetime.date.today())
hora = st.time_input("Hora de corte")
```

## `st.file_uploader` — carga de archivos

```python
archivo = st.file_uploader("Sube el dataset", type=["csv", "xlsx"])

if archivo is not None:
    if archivo.name.endswith(".csv"):
        df = pd.read_csv(archivo)
    else:
        df = pd.read_excel(archivo)
```

```python
# Múltiples archivos a la vez:
archivos = st.file_uploader("Sube varios CSV", type="csv", accept_multiple_files=True)
for archivo in archivos:
    df = pd.read_csv(archivo)
```

El objeto retornado se comporta como un file-like object (compatible con `pd.read_csv`, `open()`, etc.) — no es necesario guardarlo a disco antes de procesarlo.

## `st.color_picker`

```python
color = st.color_picker("Color del gráfico", value="#1f77b4")
```

## `st.camera_input` / `st.audio_input` — captura multimedia

```python
foto = st.camera_input("Toma una foto")
audio = st.audio_input("Graba un audio")
```

## Argumentos comunes a todos los widgets

```python
st.slider(
    "Etiqueta",
    min_value=0, max_value=100,
    key="mi_slider_unico",       # identificador único — necesario si hay widgets duplicados en la página
    help="Texto de ayuda, aparece en un ícono (?) al pasar el cursor",
    disabled=False,
    label_visibility="visible",    # "visible", "hidden", "collapsed"
)
```

`key` es especialmente importante: permite acceder al valor del widget desde `st.session_state["mi_slider_unico"]` en cualquier parte del script (ver [[07 - Session State - Estado entre Reruns]]), y es obligatorio cuando dos widgets del mismo tipo con la misma etiqueta coexisten en la página (Streamlit los distingue por `key`, no solo por posición).

## Tabla resumen — qué widget usar según el tipo de input

| Necesitas | Widget |
|---|---|
| Disparar una acción puntual | `st.button` |
| Sí/no | `st.checkbox` |
| Una opción entre pocas, siempre visibles | `st.radio` |
| Una opción entre muchas, en dropdown | `st.selectbox` |
| Varias opciones a la vez | `st.multiselect` |
| Un número o rango dentro de límites | `st.slider` |
| Texto libre corto/largo | `st.text_input` / `st.text_area` |
| Un archivo | `st.file_uploader` |
| Una fecha/hora | `st.date_input` / `st.time_input` |

## Ver también

- [[04 - Layout y Contenedores]]
- [[07 - Session State - Estado entre Reruns]]
- [[09 - Forms, Callbacks y Control de Reruns]]
