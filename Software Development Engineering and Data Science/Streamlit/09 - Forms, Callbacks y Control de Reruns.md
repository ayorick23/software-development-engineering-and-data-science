---
tags: [streamlit, dashboards, forms, callbacks, cheat-sheet]
---

# 09 — Forms, Callbacks y Control de Reruns

> Continúa de [[08 - Caching - cache_data y cache_resource]].

## El problema: un rerun por cada widget individual

Sin agrupar widgets, cada uno dispara su propio rerun independiente — si un formulario tiene 5 campos, cambiar el primero re-ejecuta el script completo, luego cambiar el segundo lo re-ejecuta de nuevo, y así sucesivamente, incluso si el usuario todavía no terminó de llenar el formulario completo.

## `st.form` — agrupar widgets, un solo rerun al enviar

```python
with st.form("formulario_entrenamiento"):
    n_estimators = st.number_input("n_estimators", 100, 1000, 300)
    max_depth = st.slider("max_depth", 3, 15, 6)
    learning_rate = st.number_input("learning_rate", 0.01, 0.5, 0.05)

    enviado = st.form_submit_button("Entrenar modelo")

if enviado:
    st.write(f"Entrenando con n_estimators={n_estimators}, max_depth={max_depth}")
    modelo = entrenar(n_estimators, max_depth, learning_rate)
```

Dentro de un `st.form`, **ningún widget dispara un rerun individual** — los valores solo se "confirman" (y el script se re-ejecuta con ellos) cuando el usuario pulsa el botón `st.form_submit_button`. Esto evita reruns costosos e innecesarios mientras el usuario todavía está ajustando múltiples parámetros relacionados.

```python
with st.form("form_filtros", clear_on_submit=False):   # clear_on_submit=True resetea los widgets tras enviar
    ...
```

## Cuándo usar `st.form` — regla práctica

| Situación | ¿Usar form? |
|---|---|
| Varios inputs relacionados que se procesan juntos (ej. hiperparámetros de un entrenamiento) | Sí |
| Un solo widget que debe reaccionar inmediatamente (ej. un filtro de región que actualiza un gráfico al instante) | No |
| Un formulario de configuración largo, donde reruns intermedios serían costosos/confusos | Sí |

## `on_click` — callback al presionar un botón

```python
def registrar_clic():
    st.session_state.clics = st.session_state.get("clics", 0) + 1

st.button("Registrar evento", on_click=registrar_clic)
st.write(f"Clics totales: {st.session_state.get('clics', 0)}")
```

El callback se ejecuta **antes** de que el resto del script se re-renderice — es el punto correcto para modificar `st.session_state` de forma que el resto del script ya vea el valor actualizado en el mismo rerun (ver [[07 - Session State - Estado entre Reruns]]).

### Pasar argumentos a un callback

```python
def eliminar_experimento(exp_id):
    st.session_state.experimentos.remove(exp_id)

for exp_id in st.session_state.experimentos:
    st.button(f"Eliminar {exp_id}", on_click=eliminar_experimento, args=(exp_id,), key=f"del_{exp_id}")
```

```python
# También acepta kwargs:
st.button("Actualizar", on_click=mi_funcion, kwargs={"parametro": valor})
```

## `on_change` — callback al modificar cualquier widget (no solo botones)

```python
def actualizar_filtro():
    st.session_state.filtro_aplicado = st.session_state.region_seleccionada

st.selectbox("Región", ["RD", "PR", "MX"], key="region_seleccionada", on_change=actualizar_filtro)
```

Disponible en prácticamente todos los widgets (`selectbox`, `slider`, `text_input`, `checkbox`, etc.) — se dispara cada vez que el valor del widget cambia, antes del rerun que muestra el nuevo estado.

## `st.form_submit_button` con callback

```python
def procesar_formulario():
    st.session_state.procesado = True

with st.form("mi_form"):
    valor = st.text_input("Valor")
    st.form_submit_button("Enviar", on_click=procesar_formulario)
```

## `st.rerun` — forzar un rerun manualmente

```python
if st.button("Reiniciar app"):
    st.session_state.clear()
    st.rerun()   # fuerza que el script se re-ejecute desde el principio, con el estado ya limpio
```

Útil cuando una acción (limpiar estado, cambiar de página programáticamente, tras completar una operación asíncrona) necesita reflejarse inmediatamente en la interfaz, sin esperar a que el usuario dispare la siguiente interacción de forma natural.

## `st.stop` — detener la ejecución del script en un punto específico

```python
archivo = st.file_uploader("Sube un archivo")
if archivo is None:
    st.warning("Por favor sube un archivo para continuar")
    st.stop()   # el resto del script NO se ejecuta hasta que haya un archivo

df = pd.read_csv(archivo)
st.dataframe(df)   # esta línea nunca se alcanza sin un archivo cargado
```

Patrón de guard clause para evitar que el resto del script intente procesar datos que todavía no existen — más limpio que anidar todo el resto del script dentro de un único `if archivo is not None:`.

## Fragmentos — `st.fragment` para reruns parciales (rendimiento)

```python
@st.fragment
def panel_metricas():
    st.metric("MAE", calcular_mae())   # solo ESTE fragmento se re-ejecuta al interactuar dentro de él

    if st.button("Actualizar métricas"):
        st.rerun(scope="fragment")   # rerun LIMITADO al fragmento, no a toda la app

panel_metricas()
st.line_chart(df_completo)   # esto NO se vuelve a ejecutar cuando el fragment se actualiza
```

`@st.fragment` permite que una **porción** de la app se re-ejecute de forma aislada, sin disparar un rerun del script completo — relevante en apps grandes donde una sección (por ejemplo, un panel que se actualiza automáticamente cada pocos segundos) no debería forzar que el resto de la página, potencialmente costosa de renderizar, se recalcule también. Ver [[14 - Buenas Prácticas, Rendimiento y Testing]] para cuándo vale la pena esta optimización.

### Auto-refresco con fragmentos

```python
@st.fragment(run_every=5)   # se re-ejecuta automáticamente cada 5 segundos, SIN interacción del usuario
def panel_en_vivo():
    st.metric("Demanda actual", obtener_demanda_tiempo_real())

panel_en_vivo()
```

Patrón estándar para dashboards con datos "en vivo" (métricas de un sistema en producción, precios de mercado) — evita tener que implementar polling manual con JavaScript o recargar la página completa.

## Ver también

- [[07 - Session State - Estado entre Reruns]]
- [[08 - Caching - cache_data y cache_resource]]
- [[14 - Buenas Prácticas, Rendimiento y Testing]]
