---
tags: [streamlit, dashboards, multipage, navegacion, cheat-sheet]
---

# 10 — Multipage Apps y Navegación

> Continúa de [[09 - Forms, Callbacks y Control de Reruns]].

## Método clásico — carpeta `pages/`

```
mi_app/
├── app.py              # página principal (la que se ejecuta con `streamlit run app.py`)
└── pages/
    ├── 1_Analisis.py
    ├── 2_Prediccion.py
    └── 3_Configuracion.py
```

Streamlit detecta automáticamente cualquier archivo `.py` dentro de `pages/` y genera navegación en el sidebar sin código adicional — el prefijo numérico (`1_`, `2_`) controla el **orden** en que aparecen en el menú, y no se muestra en el título visible (Streamlit lo oculta automáticamente, junto con guiones bajos convertidos a espacios).

```python
# pages/1_Analisis.py — es un script de Streamlit completo, independiente
import streamlit as st

st.title("Análisis Exploratorio")
st.write("Contenido de esta página...")
```

Cada página es un script separado que se ejecuta de forma independiente al navegar hacia ella — **no** comparte variables locales con `app.py`, solo `st.session_state` (ver [[07 - Session State - Estado entre Reruns]]) es compartido entre páginas.

## `st.navigation` + `st.Page` — API moderna, más flexible

```python
# app.py
import streamlit as st

pagina_analisis = st.Page("paginas/analisis.py", title="Análisis", icon="📊")
pagina_prediccion = st.Page("paginas/prediccion.py", title="Predicción", icon="🔮")
pagina_config = st.Page("paginas/configuracion.py", title="Configuración", icon="⚙️")

pg = st.navigation([pagina_analisis, pagina_prediccion, pagina_config])
pg.run()
```

A diferencia del método de carpeta `pages/` (donde la estructura de archivos determina automáticamente todo), `st.navigation` da control **explícito y programático** sobre qué páginas existen, su orden, íconos, y — crucialmente — permite mostrar páginas distintas según condiciones (ej. rol del usuario).

## Agrupar páginas por secciones

```python
pg = st.navigation({
    "Análisis": [pagina_resumen, pagina_detalle],
    "Modelos": [pagina_entrenamiento, pagina_prediccion],
    "Administración": [pagina_config, pagina_usuarios],
})
pg.run()
```

Genera un menú de navegación con encabezados de sección — útil en apps con muchas páginas donde una lista plana se vuelve difícil de escanear visualmente.

## Navegación condicional — mostrar páginas según el rol del usuario

```python
paginas_base = [pagina_resumen, pagina_analisis]

if st.session_state.get("es_admin", False):
    paginas_base.append(pagina_administracion)

pg = st.navigation(paginas_base)
pg.run()
```

Este es el caso de uso central que motivó `st.navigation` sobre el método de carpeta `pages/` — con la carpeta, **todas** las páginas son siempre visibles a cualquier usuario; con `st.navigation`, la lista de páginas se puede construir dinámicamente en tiempo de ejecución según el estado de la sesión (autenticación, permisos).

## Navegar programáticamente entre páginas

```python
if st.button("Ir a Predicción"):
    st.switch_page("paginas/prediccion.py")
```

```python
# Con st.Page, se puede navegar pasando el objeto directamente:
if st.button("Ir a Predicción"):
    st.switch_page(pagina_prediccion)
```

Útil para flujos guiados (ej. tras completar un formulario en una página, redirigir automáticamente a la página de resultados) sin depender de que el usuario haga clic manualmente en el menú de navegación.

## `st.Page` con funciones en vez de archivos

```python
def vista_resumen():
    st.title("Resumen")
    st.write("...")

def vista_detalle():
    st.title("Detalle")
    st.write("...")

pg = st.navigation([
    st.Page(vista_resumen, title="Resumen", default=True),
    st.Page(vista_detalle, title="Detalle"),
])
pg.run()
```

Alternativa a archivos separados — cada página es simplemente una función de Python, útil en apps pequeñas donde crear archivos individuales por página añade fricción innecesaria, o cuando se prefiere mantener toda la app en un solo módulo por razones de organización del proyecto.

## Parámetros de URL — leer query params

```python
region = st.query_params.get("region", "RD")
st.write(f"Mostrando datos de: {region}")

# Modificar los query params programáticamente (actualiza la URL visible del navegador):
st.query_params["region"] = "PR"
```

Permite construir URLs "linkeables" a un estado específico de la app (ej. compartir un link que abre directamente en una región/filtro específico) — relevante para dashboards donde compartir una vista exacta con un colega es un caso de uso común.

## Patrón completo — app con autenticación simple y navegación condicional

```python
import streamlit as st

st.session_state.setdefault("autenticado", False)

if not st.session_state.autenticado:
    st.title("Iniciar sesión")
    usuario = st.text_input("Usuario")
    clave = st.text_input("Contraseña", type="password")
    if st.button("Entrar") and verificar_credenciales(usuario, clave):
        st.session_state.autenticado = True
        st.rerun()
    st.stop()   # detiene el resto del script si no está autenticado — ver 09

pg = st.navigation([pagina_resumen, pagina_analisis, pagina_prediccion])
pg.run()
```

## Ver también

- [[07 - Session State - Estado entre Reruns]]
- [[09 - Forms, Callbacks y Control de Reruns]]
- [[12 - Despliegue - Community Cloud, Docker y Secrets]]
