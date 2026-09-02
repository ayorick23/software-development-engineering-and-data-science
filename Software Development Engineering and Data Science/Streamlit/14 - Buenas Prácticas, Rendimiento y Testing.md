---
tags: [streamlit, dashboards, buenas-practicas, rendimiento, testing, cheat-sheet]
---

# 14 — Buenas Prácticas, Rendimiento y Testing

> Cierre del cheat-sheet. Se apoya en todos los archivos anteriores, especialmente [[08 - Caching - cache_data y cache_resource]] y [[09 - Forms, Callbacks y Control de Reruns]].

## El error más común: no cachear operaciones costosas

```python
# INCORRECTO — se re-ejecuta en CADA interacción del usuario, sin importar qué cambió
def cargar_datos():
    return pd.read_sql("SELECT * FROM demanda_historica", conexion)

df = cargar_datos()

# CORRECTO
@st.cache_data(ttl=600)
def cargar_datos():
    return pd.read_sql("SELECT * FROM demanda_historica", conexion)
```

Ver [[08 - Caching - cache_data y cache_resource]] — este es, con diferencia, el error de rendimiento más frecuente en apps de Streamlit nuevas: olvidar que **todo** el script se re-ejecuta en cada interacción, no solo la parte que cambió.

## Estructurar el script para minimizar trabajo por rerun

```python
# Patrón recomendado: cargar y cachear PRIMERO, filtrar/transformar DESPUÉS con los datos ya en memoria
@st.cache_data
def cargar_datos_completos():
    return pd.read_csv("dataset_grande.csv")   # costoso, se cachea

df_completo = cargar_datos_completos()

region = st.selectbox("Región", df_completo["region"].unique())
df_filtrado = df_completo[df_completo["region"] == region]   # barato, se recalcula cada rerun sin problema
```

Separar claramente "lo costoso que rara vez cambia" (cacheado) de "lo barato que cambia con cada interacción" (recalculado libremente) es la estructura base de cualquier app de Streamlit con buen rendimiento.

## Usar `st.fragment` para aislar partes costosas de renderizar

```python
@st.fragment
def seccion_grafico_pesado():
    fig = generar_grafico_3d_complejo(df)   # costoso de renderizar
    st.plotly_chart(fig)

seccion_grafico_pesado()

st.selectbox("Filtro rápido", opciones)   # interactuar con esto NO debería re-renderizar el gráfico pesado de arriba
```

Ver [[09 - Forms, Callbacks y Control de Reruns]] — sin `@st.fragment`, cualquier interacción en cualquier parte de la página re-ejecuta y re-renderiza **todo**, incluyendo elementos costosos que no tenían por qué cambiar.

## Evitar recrear objetos costosos dentro del cuerpo principal del script

```python
# INCORRECTO — el cliente HTTP/conexión se recrea en cada rerun
def obtener_datos():
    cliente = crear_cliente_api()   # esto NO debería estar aquí sin cache
    return cliente.get_datos()

# CORRECTO
@st.cache_resource
def obtener_cliente():
    return crear_cliente_api()

cliente = obtener_cliente()
datos = cliente.get_datos()
```

## Estructura de proyecto recomendada para apps medianas-grandes

```
mi_app/
├── app.py                      # navegación principal (st.navigation)
├── paginas/
│   ├── analisis.py
│   └── prediccion.py
├── componentes/                 # funciones reutilizables de UI, NO scripts de página
│   ├── __init__.py
│   └── graficos.py
├── logica/                       # lógica de negocio SIN dependencia de Streamlit — testeable con pytest normal
│   ├── __init__.py
│   └── calculos.py
├── requirements.txt
└── .streamlit/
    ├── config.toml
    └── secrets.toml              # NO versionado en Git
```

Separar la **lógica de negocio** (funciones puras de Python, sin ningún `st.*`) de la **capa de presentación** (los scripts de página que llaman `st.write`, `st.dataframe`, etc.) es la práctica más importante para que el código sea testeable — una función que mezcla cálculos con llamadas a `st.metric` no se puede probar con `pytest` estándar sin simular todo el entorno de Streamlit.

```python
# logica/calculos.py — testeable con pytest normal, SIN Streamlit involucrado
def calcular_mae_ponderado(y_true, y_pred, pesos):
    error = abs(y_pred - y_true) * pesos
    return error.sum() / pesos.sum()

# paginas/analisis.py — solo orquesta UI, delega el cálculo
from logica.calculos import calcular_mae_ponderado
mae = calcular_mae_ponderado(df["real"], df["prediccion"], df["peso"])
st.metric("MAE ponderado", f"{mae:.2f}")
```

## `AppTest` — testing automatizado de la app completa

```python
from streamlit.testing.v1 import AppTest

def test_app_muestra_titulo():
    at = AppTest.from_file("app.py")
    at.run()
    assert at.title[0].value == "Panel de Demanda"

def test_slider_actualiza_prediccion():
    at = AppTest.from_file("app.py")
    at.run()
    at.slider[0].set_value(180).run()   # simula que el usuario mueve el slider a 180
    assert "180" in at.metric[0].value
```

`AppTest` permite escribir tests de `pytest` que simulan interacciones de usuario sobre la app completa (mover sliders, hacer clic en botones, escribir texto) y verificar el estado resultante de los elementos renderizados — sin necesitar un navegador real ni herramientas de testing E2E más pesadas (Selenium/Playwright) para la mayoría de los casos de verificación funcional básica.

```bash
pytest tests/
```

Ver `Machine Learning/13-Testing-en-Machine-Learning.md` para el contexto general de testing en proyectos de ML — `AppTest` cubre específicamente la capa de interfaz, complementando (no reemplazando) los tests de lógica de negocio con `pytest` estándar sobre las funciones puras.

## Manejo de errores — no dejar que una excepción rompa toda la página

```python
try:
    df = cargar_datos_externos()
except Exception as e:
    st.error(f"No se pudieron cargar los datos: {e}")
    st.stop()   # ver 09 — detiene el resto del script de forma controlada
```

Sin manejo explícito, una excepción no capturada muestra un traceback completo de Python directamente en la interfaz — aceptable durante desarrollo, pero poco profesional (y potencialmente revelador de detalles internos del sistema) en una app expuesta a usuarios finales.

## Seguridad — resumen de puntos ya cubiertos

- Nunca usar `unsafe_allow_html=True` con contenido proveniente de input de usuario (ver [[02 - Elementos de Texto y Markdown]]).
- Credenciales siempre vía `st.secrets` o variables de entorno, nunca hardcodeadas (ver [[12 - Despliegue - Community Cloud, Docker y Secrets]]).
- Si la app maneja datos sensibles, la autenticación real debe resolverse a nivel de reverse proxy/gateway, no confiar solo en lógica de `session_state` dentro del propio script de Streamlit (que corre del lado del servidor, pero sin las garantías de un sistema de auth dedicado).

## Checklist final antes de considerar una app "lista para compartir"

1. ¿Toda operación costosa (carga de datos, modelos, llamadas a APIs) está envuelta en `st.cache_data`/`st.cache_resource`?
2. ¿La lógica de negocio vive separada de la capa de presentación, en funciones testeables sin Streamlit?
3. ¿Hay manejo explícito de errores en las operaciones que pueden fallar (carga de archivos, conexiones externas)?
4. ¿Las credenciales están en `secrets.toml`/variables de entorno, excluidas de Git?
5. ¿Existen al menos algunos tests con `AppTest` cubriendo el flujo principal de la app?
6. Si hay secciones costosas de renderizar que no dependen de todas las interacciones, ¿se usó `@st.fragment` para aislarlas?

## Ver también

- [[08 - Caching - cache_data y cache_resource]]
- [[09 - Forms, Callbacks y Control de Reruns]]
- `Machine Learning/13-Testing-en-Machine-Learning.md`
- `Machine Learning/25-Testing-Avanzado.md`
