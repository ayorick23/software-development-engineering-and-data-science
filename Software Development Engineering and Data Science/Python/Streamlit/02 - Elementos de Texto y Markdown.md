---
tags: [streamlit, dashboards, cheat-sheet]
---

# 02 — Elementos de Texto y Markdown

> Continúa de [[01 - Introducción y Modelo de Ejecución]].

## Jerarquía de títulos

```python
import streamlit as st

st.title("Título principal de la app")
st.header("Encabezado de sección")
st.subheader("Subencabezado")
st.caption("Texto pequeño, gris — para notas al pie o aclaraciones")
```

```python
# Con ancla para navegación interna (útil combinado con un índice):
st.header("Resultados", anchor="resultados")
```

## `st.markdown` — texto con formato Markdown completo

```python
st.markdown("""
Este es un párrafo con **negrita**, *cursiva* y [un enlace](https://streamlit.io).

- Punto 1
- Punto 2

> Una cita destacada
""")
```

```python
# Con soporte de HTML crudo (usar con precaución, ver advertencia de seguridad más abajo):
st.markdown("<span style='color:red'>Texto en rojo</span>", unsafe_allow_html=True)
```

> **Advertencia de seguridad con `unsafe_allow_html=True`**: si el contenido que se pasa a `st.markdown` proviene de input de usuario (no de una constante controlada por el desarrollador), habilitar HTML crudo abre la puerta a inyección de HTML/JavaScript malicioso (XSS). Reservar `unsafe_allow_html=True` únicamente para contenido estático definido en el propio código.

## Texto plano sin interpretación de formato

```python
st.text("Este texto se muestra EXACTAMENTE como está escrito, sin interpretar Markdown")
```

Útil para mostrar output de logs, mensajes de error crudos, o cualquier texto donde caracteres como `*`/`_` no deben interpretarse como formato.

## `st.code` — bloques de código con resaltado de sintaxis

```python
st.code("""
def calcular_demanda(df):
    return df['cantidad'] * df['precio']
""", language="python")

st.code("SELECT * FROM demanda WHERE region = 'RD'", language="sql")
```

## `st.latex` — notación matemática

```python
st.latex(r"""
\text{MAE} = \frac{1}{n} \sum_{i=1}^{n} |y_i - \hat{y}_i|
""")
```

Útil para documentar la fórmula exacta detrás de una métrica mostrada en el dashboard — renderiza usando KaTeX, sin necesitar ninguna librería adicional.

## `st.divider` — separador visual horizontal

```python
st.header("Sección A")
st.write("contenido...")
st.divider()
st.header("Sección B")
```

## `st.badge` — etiquetas de estado inline

```python
st.badge("Producción", icon="✅", color="green")
st.badge("Beta", color="orange")
```

Útil para indicar visualmente el estado de un modelo/feature directamente junto a un título (ej. "Modelo v3 — 🟢 En producción").

## Emojis en títulos — atajo de sintaxis

```python
st.title(":bar_chart: Panel de Demanda")   # equivalente a incluir el emoji directamente: "📊 Panel de Demanda"
```

Streamlit interpreta shortcodes de emoji (sintaxis `:nombre:`) en cualquier texto — útil para mantener el código legible sin pegar el carácter Unicode directamente.

## Combinando Markdown con variables — f-strings

```python
mae_actual = 12.4
st.markdown(f"El **MAE** actual del modelo es de `{mae_actual:.2f}`")
```

Patrón habitual para mostrar métricas calculadas dinámicamente dentro de texto descriptivo — combina f-strings de Python normales con la sintaxis de Markdown de Streamlit.

## Mostrar excepciones de forma legible

```python
try:
    resultado = 1 / 0
except Exception as e:
    st.exception(e)   # muestra el traceback completo, formateado, dentro de la app
```

Útil durante desarrollo para depurar errores directamente en la interfaz sin tener que revisar la terminal — en producción, normalmente se reemplaza por un mensaje más amigable con `st.error` (ver [[04 - Layout y Contenedores]] para elementos de estado como `st.error`/`st.success`).

## Ver también

- [[01 - Introducción y Modelo de Ejecución]]
- [[03 - Widgets de Entrada]]
- [[05 - Mostrar Datos - DataFrames, Tablas y Métricas]]
