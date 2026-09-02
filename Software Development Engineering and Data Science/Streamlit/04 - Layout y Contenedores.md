---
tags: [streamlit, dashboards, layout, cheat-sheet]
---

# 04 — Layout y Contenedores

> Continúa de [[03 - Widgets de Entrada]].

Por defecto, todo elemento de Streamlit se apila verticalmente en el orden en que aparece en el script. Este archivo cubre cómo organizar la página en columnas, pestañas, y secciones colapsables.

## `st.sidebar` — panel lateral

```python
with st.sidebar:
    st.header("Filtros")
    region = st.selectbox("Región", ["RD", "PR", "MX"])
    dias = st.slider("Días", 7, 365, 90)
```

```python
# Sintaxis alternativa sin `with`, usando el objeto sidebar directamente:
region = st.sidebar.selectbox("Región", ["RD", "PR", "MX"])
```

El patrón estándar en dashboards: controles de filtrado en el sidebar, resultados/visualizaciones en el área principal — separa claramente "controles" de "contenido".

## `st.columns` — layout horizontal

```python
col1, col2, col3 = st.columns(3)   # tres columnas de igual ancho

with col1:
    st.metric("MAE", "12.4")
with col2:
    st.metric("RMSE", "18.7")
with col3:
    st.metric("R²", "0.89")
```

```python
# Anchos proporcionales (relación 2:1:1, no necesariamente igual):
col_grande, col_a, col_b = st.columns([2, 1, 1])

# Con espacio/gap entre columnas:
col1, col2 = st.columns(2, gap="large")   # "small", "medium", "large"

# Alineación vertical del contenido dentro de cada columna:
col1, col2 = st.columns(2, vertical_alignment="center")
```

## `st.tabs` — pestañas

```python
tab_resumen, tab_detalle, tab_config = st.tabs(["Resumen", "Detalle", "Configuración"])

with tab_resumen:
    st.line_chart(df_demanda)

with tab_detalle:
    st.dataframe(df_demanda)

with tab_config:
    st.write("Parámetros del modelo...")
```

Útil para organizar contenido relacionado pero extenso sin saturar la página con scroll infinito — cada tab se renderiza en el mismo rerun (a diferencia de páginas separadas, ver [[10 - Multipage Apps y Navegación]]), por lo que cambiar de tab no dispara un nuevo rerun del script completo.

## `st.expander` — secciones colapsables

```python
with st.expander("Ver detalles técnicos del modelo"):
    st.write("XGBoost, 300 estimadores, max_depth=6...")
    st.dataframe(df_hiperparametros)
```

```python
with st.expander("Configuración avanzada", expanded=False):   # expanded=True para que inicie abierto
    ...
```

Ideal para información secundaria (logs, configuración avanzada, detalles técnicos) que no todos los usuarios necesitan ver de inmediato, pero que debe estar accesible sin navegar a otra página.

## `st.container` — agrupación lógica sin efecto visual directo

```python
contenedor = st.container()
contenedor.write("Esto aparece DENTRO del contenedor")

st.write("Esto aparece DESPUÉS, aunque el código esté antes en algunos casos")
contenedor.write("Se puede seguir escribiendo en el contenedor definido antes, fuera de orden")
```

`st.container()` permite **reservar un lugar** en el layout y llenarlo con contenido definido en otro punto del script — útil para casos donde el orden lógico del código no coincide con el orden visual deseado (por ejemplo, mostrar un resumen calculado más abajo en el script, pero visualmente arriba de todo).

```python
# Container con borde visible y altura fija (útil para paneles tipo "card"):
with st.container(border=True, height=300):
    st.write("Contenido dentro de una tarjeta con scroll si excede la altura")
```

## `st.empty` — placeholder actualizable

```python
placeholder = st.empty()

for i in range(10):
    placeholder.write(f"Procesando paso {i+1}/10...")
    time.sleep(0.5)

placeholder.success("¡Completado!")   # reemplaza el contenido anterior del placeholder, no lo apila
```

A diferencia de llamadas sucesivas a `st.write` (que se apilan una debajo de otra), `st.empty()` crea un espacio que se **sobrescribe** en cada actualización — el patrón estándar para mostrar progreso en tiempo real sin llenar la página de mensajes intermedios.

## Elementos de estado — alertas visuales

```python
st.success("Modelo entrenado exitosamente")
st.info("El pipeline se ejecuta cada domingo a las 3am")
st.warning("El dataset tiene 15% de valores faltantes")
st.error("No se pudo conectar a la base de datos")
```

## `st.progress` y `st.spinner` — feedback de operaciones largas

```python
barra = st.progress(0)
for i in range(100):
    barra.progress(i + 1)
    time.sleep(0.01)

with st.spinner("Entrenando modelo..."):
    modelo = entrenar_modelo_costoso()
st.success("Listo")
```

`st.spinner` es un context manager — muestra un indicador de carga mientras el bloque `with` se ejecuta, y lo retira automáticamente al terminar.

## `st.status` — bloque de progreso con múltiples pasos

```python
with st.status("Ejecutando pipeline...", expanded=True) as status:
    st.write("Cargando datos...")
    df = cargar_datos()
    st.write("Entrenando modelo...")
    modelo = entrenar(df)
    status.update(label="Pipeline completado", state="complete")
```

Más rico que `st.spinner` para procesos con varias etapas — muestra un log de pasos dentro de un contenedor colapsable, con un estado final (`complete`/`error`) explícito.

## `st.dialog` — ventanas modales

```python
@st.dialog("Confirmar acción")
def confirmar_promocion():
    st.write("¿Promover este modelo a producción?")
    if st.button("Confirmar"):
        promover_modelo()
        st.rerun()

if st.button("Promover a producción"):
    confirmar_promocion()
```

Útil para confirmaciones de acciones destructivas o irreversibles (promover un modelo, eliminar un experimento) sin navegar fuera de la página actual.

## Ver también

- [[03 - Widgets de Entrada]]
- [[05 - Mostrar Datos - DataFrames, Tablas y Métricas]]
- [[10 - Multipage Apps y Navegación]]
