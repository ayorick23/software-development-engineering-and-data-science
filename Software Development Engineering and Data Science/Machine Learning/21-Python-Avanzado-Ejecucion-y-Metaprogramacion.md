---
tags: [python, avanzado, decoradores, generadores]
---

# 21 — Python Avanzado: Ejecución y Metaprogramación (Context Managers, Decorators, Iteradores, Generadores, Closures, Properties, Magic Methods)

> Nota del mentor: esta es la parte de Python que separa a quien "escribe scripts que funcionan" de quien "diseña librerías que otros usan sin sorpresas". No es sintaxis avanzada por presumir — cada una de estas herramientas resuelve un problema real y recurrente de un pipeline de ML.

## 1. Context Managers — garantizar que algo se limpie, pase lo que pase

```python
from contextlib import contextmanager
import time

@contextmanager
def medir_tiempo(nombre_paso: str, logger):
    inicio = time.perf_counter()
    try:
        yield
    finally:
        duracion = time.perf_counter() - inicio
        logger.info(f"{nombre_paso} completado en {duracion:.2f}s")

# Uso:
with medir_tiempo("Entrenamiento XGBoost", logger):
    modelo.fit(X_train, y_train)
```

El punto clave del `finally`: el tiempo se loguea **incluso si `model.fit()` lanza una excepción**. Esto es exactamente lo que necesitas para conexiones de base de datos (`with pyodbc.connect(...) as conn:`), archivos, y transacciones — garantizar liberación de recursos sin importar si el bloque tuvo éxito o falló, sin tener que repetir `try/finally` a mano en cada lugar.

## 2. Decorators — modificar comportamiento sin tocar la función original

```python
import functools
import time

def reintentar(intentos: int = 3, espera_segundos: float = 2.0):
    def decorador(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for intento in range(1, intentos + 1):
                try:
                    return func(*args, **kwargs)
                except ConnectionError as e:
                    if intento == intentos:
                        raise
                    logger.warning(f"Intento {intento} falló: {e}. Reintentando...")
                    time.sleep(espera_segundos)
        return wrapper
    return decorador

@reintentar(intentos=3)
def conectar_a_qflow():
    return pyodbc.connect(connection_string)
```

Este patrón es exactamente lo que necesitas para conexiones a `qf.hrAgentForecastResult` que pueden fallar por timeouts de red intermitentes — sin este decorador, tendrías el mismo bloque `try/except` con reintentos copiado en cada función que toca la base de datos.

`functools.wraps` no es opcional: sin él, pierdes el `__name__` y `__doc__` originales de la función decorada, lo cual rompe introspección, documentación automática (ver [[33-Documentacion-Profesional]]) y hasta algunos frameworks de testing.

## 3. Iteradores y Generadores — procesar datos sin cargarlos todos en memoria

```python
def leer_lotes_grandes(query: str, conn, tamano_lote: int = 10_000):
    """Generador: procesa millones de filas sin cargar todo en RAM."""
    cursor = conn.cursor()
    cursor.execute(query)
    while True:
        lote = cursor.fetchmany(tamano_lote)
        if not lote:
            break
        yield pd.DataFrame.from_records(lote, columns=[c[0] for c in cursor.description])

for lote_df in leer_lotes_grandes("SELECT * FROM qf.hrCallData", conn):
    procesar_features(lote_df)
```

La diferencia con una función normal: `yield` pausa la ejecución y devuelve el control al llamador, retomando exactamente donde se quedó en la siguiente iteración. Si tu histórico de `qf.hrGetAbandonData` es de millones de filas (recordemos que no tiene filtro de fecha), cargarlo todo con `fetchall()` puede tronar la memoria del servidor; un generador procesa por lotes con memoria constante.

## 4. Comprensiones — cuándo sí y cuándo NO

```python
# Bien: legible, una transformación simple
columnas_lag = [f"demand_lag_{n}" for n in [1, 7, 14, 30]]

# Mal: comprensión anidada ilegible — mejor un loop explícito
resultado = [x for sublist in [[i*j for j in range(5)] for i in range(5)] for x in sublist]
```

Regla de un senior: si la comprensión no cabe cómodamente en una línea (~80-100 caracteres) o necesita más de un nivel de anidamiento, **escribe el loop explícito**. El código se lee muchas más veces de las que se escribe — la "elegancia" de una comprensión compleja no vale la carga cognitiva que impone a quien la lee seis meses después (probablemente tú mismo).

## 5. Closures y Lambdas — cuándo tienen sentido real

```python
def crear_validador_rango(minimo: float, maximo: float):
    """Closure: 'recuerda' minimo y maximo aunque la función externa ya terminó."""
    def validar(valor: float) -> bool:
        return minimo <= valor <= maximo
    return validar

validar_demanda = crear_validador_rango(0, 100_000)
validar_service_time = crear_validador_rango(0, 3600)
```

Las **lambdas** tienen sentido solo para funciones triviales de una línea, típicamente como argumento de otra función:

```python
df.sort_values(by="fecha", key=lambda x: pd.to_datetime(x))
mejores_modelos = sorted(resultados, key=lambda r: r.mae)
```

Si tu lambda necesita más de una expresión simple, o si le vas a poner nombre a la variable que la contiene, **ya debería ser una función `def` normal** — una lambda asignada a una variable (`calcular = lambda x: ...`) es un anti-patrón, PEP 8 lo señala explícitamente.

## 6. Properties — validación transparente sin romper la interfaz

```python
class ConfiguracionModelo:
    def __init__(self, dias_atras: int):
        self._dias_atras = dias_atras

    @property
    def dias_atras(self) -> int:
        return self._dias_atras

    @dias_atras.setter
    def dias_atras(self, valor: int) -> None:
        if valor <= 0:
            raise ValueError("dias_atras debe ser positivo")
        self._dias_atras = valor
```

Desde afuera, `config.dias_atras = -5` se ve como una simple asignación de atributo, pero internamente dispara la validación. Esto te permite empezar con atributos públicos simples y agregar validación después **sin romper el código que ya usa la clase** — no tienes que cambiar `config.dias_atras` por `config.set_dias_atras(-5)` en todo el proyecto.

## 7. Magic Methods — cuando tu objeto necesita comportarse como un tipo nativo

```python
class ResultadoEvaluacion:
    def __init__(self, mae: float, rmse: float):
        self.mae = mae
        self.rmse = rmse

    def __repr__(self) -> str:
        return f"ResultadoEvaluacion(mae={self.mae:.3f}, rmse={self.rmse:.3f})"

    def __lt__(self, otro: "ResultadoEvaluacion") -> bool:
        return self.mae < otro.mae  # permite sorted(lista_de_resultados)

    def __eq__(self, otro: object) -> bool:
        if not isinstance(otro, ResultadoEvaluacion):
            return NotImplemented
        return self.mae == otro.mae and self.rmse == otro.rmse
```

Con `__lt__` definido, `sorted(resultados_challengers)` funciona directamente para encontrar el mejor challenger sin escribir una función `key` cada vez. `__repr__` es el que verás en tus logs y en el debugger — sin él, tus objetos se imprimen como `<ResultadoEvaluacion object at 0x7f...>`, información inútil quiero decir "prácticamente cero valor" al debuggear.

## Ver también
- [[20-Python-Avanzado-Sistema-de-Tipos]]
- [[11-Logging-en-Python-para-ML]]
- [[22-Organizacion-del-Codigo-y-Principios-de-Diseno]]
