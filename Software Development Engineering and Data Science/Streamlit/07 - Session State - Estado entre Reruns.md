---
tags: [streamlit, dashboards, session-state, cheat-sheet]
---

# 07 — Session State: Estado entre Reruns

> Continúa de [[06 - Gráficos y Visualización]]. Este es el concepto más importante y menos intuitivo de todo Streamlit — imprescindible entender [[01 - Introducción y Modelo de Ejecución]] primero.

## El problema que resuelve

Como el script completo se re-ejecuta en cada interacción (ver [[01 - Introducción y Modelo de Ejecución]]), **cualquier variable local normal se reinicia** en cada rerun:

```python
contador = 0   # esto se reinicia a 0 en CADA rerun — nunca acumula

if st.button("Incrementar"):
    contador += 1
st.write(contador)   # SIEMPRE muestra 0 o 1, nunca más — el botón no puede "recordar" clics anteriores
```

`st.session_state` es un diccionario persistente **por sesión de navegador** (no global entre usuarios) que sobrevive entre reruns — es el mecanismo para que la app "recuerde" cosas más allá de la interacción inmediata.

## Uso básico — leer y escribir

```python
import streamlit as st

if "contador" not in st.session_state:
    st.session_state.contador = 0   # inicialización — SOLO ocurre la primera vez

if st.button("Incrementar"):
    st.session_state.contador += 1

st.write(st.session_state.contador)   # ahora sí acumula correctamente entre clics
```

```python
# Sintaxis equivalente, como diccionario:
st.session_state["contador"] = 0
valor = st.session_state["contador"]
```

## `st.session_state` vinculado automáticamente a widgets con `key`

```python
st.slider("Días atrás", 7, 365, 90, key="dias_atras")

# El valor del widget está disponible en session_state con el MISMO nombre que `key`, sin código adicional:
st.write(st.session_state.dias_atras)
```

Cualquier widget con un `key` explícito registra su valor automáticamente en `session_state` bajo ese nombre — esto permite leer (e incluso modificar programáticamente, con matices) el valor de un widget desde cualquier parte del script, no solo desde su variable de retorno directa.

## Inicialización segura — el patrón `setdefault`

```python
st.session_state.setdefault("historial", [])

if st.button("Agregar entrada"):
    st.session_state.historial.append("nueva entrada")

st.write(st.session_state.historial)
```

Equivalente al patrón `if "clave" not in st.session_state`, pero más compacto — `setdefault` solo inicializa si la clave no existe todavía, sin sobrescribir el valor en reruns posteriores.

## Callbacks que modifican `session_state`

```python
def incrementar():
    st.session_state.contador += 1

st.session_state.setdefault("contador", 0)
st.button("Incrementar", on_click=incrementar)
st.write(st.session_state.contador)
```

Modificar `session_state` dentro de un callback (`on_click`/`on_change`, ver [[09 - Forms, Callbacks y Control de Reruns]]) es el patrón recomendado para lógica que debe ejecutarse **antes** de que el resto del script se vuelva a renderizar con el nuevo estado — evita inconsistencias de un frame donde el valor mostrado no refleja todavía la acción del usuario.

## Modificar el valor de un widget programáticamente

```python
st.session_state.setdefault("region", "RD")

if st.button("Resetear a RD"):
    st.session_state.region = "RD"   # esto SÍ actualiza el widget visualmente en el próximo rerun

st.selectbox("Región", ["RD", "PR", "MX"], key="region")
```

> **Restricción importante**: no se puede asignar directamente a `st.session_state[key]` de un widget **después** de que ese widget ya fue instanciado en el mismo rerun — Streamlit lanza un error (`StreamlitAPIException`). La asignación debe ocurrir **antes** de crear el widget (como en el ejemplo, donde el botón que modifica `session_state.region` aparece antes del `st.selectbox` en el código) o dentro de un callback ejecutado en el rerun anterior.

## Widgets que "olvidan" su valor al desaparecer

```python
if mostrar_avanzado:
    st.slider("Parámetro avanzado", 0, 100, key="param_avanzado")
# Si `mostrar_avanzado` se vuelve False, el widget deja de renderizarse Y SU VALOR EN session_state SE ELIMINA
```

Un comportamiento que sorprende: si un widget con `key` deja de ejecutarse en un rerun (por ejemplo, está dentro de un `if` que ahora es falso), Streamlit **elimina** su entrada de `session_state` — no la conserva "por si acaso" vuelve a aparecer. Si se necesita preservar ese valor incluso cuando el widget no se muestra, hay que guardarlo explícitamente en una clave separada antes de que desaparezca.

## Compartir estado entre páginas — apps multipágina

```python
# En pages/1_Analisis.py
st.session_state.setdefault("df_cargado", None)

if st.session_state.df_cargado is not None:
    st.dataframe(st.session_state.df_cargado)

# En pages/2_Prediccion.py — el MISMO session_state es accesible, es una sesión de navegador, no por página
if st.session_state.df_cargado is not None:
    predicciones = modelo.predict(st.session_state.df_cargado)
```

`session_state` es compartido entre todas las páginas de una app multipágina (ver [[10 - Multipage Apps y Navegación]]) dentro de la misma sesión de navegador — es el mecanismo estándar para pasar datos entre páginas sin recargarlos.

## `st.cache_resource`/`session_state` — cuándo usar cada uno

| Necesitas | Mecanismo |
|---|---|
| Un valor que persiste solo mientras el usuario tiene la pestaña abierta, específico de ESA sesión | `st.session_state` |
| El resultado de una función costosa, compartido entre TODOS los usuarios de la app | `st.cache_data` / `st.cache_resource` (ver [[08 - Caching - cache_data y cache_resource]]) |

Confundir ambos es un error común: `session_state` es por-usuario y se pierde al cerrar la pestaña; el cache es compartido globalmente entre todas las sesiones activas de la app y persiste mientras el servidor de Streamlit siga corriendo.

## Ejemplo completo — un carrito de "experimentos seleccionados"

```python
st.session_state.setdefault("experimentos_seleccionados", set())

for exp_id in ["exp-001", "exp-002", "exp-003"]:
    seleccionado = st.checkbox(f"Comparar {exp_id}", key=f"check_{exp_id}")
    if seleccionado:
        st.session_state.experimentos_seleccionados.add(exp_id)
    else:
        st.session_state.experimentos_seleccionados.discard(exp_id)

st.write("Comparando:", st.session_state.experimentos_seleccionados)
```

## Ver también

- [[01 - Introducción y Modelo de Ejecución]]
- [[08 - Caching - cache_data y cache_resource]]
- [[09 - Forms, Callbacks y Control de Reruns]]
- [[10 - Multipage Apps y Navegación]]
