---
tags: [ingenieria-de-software, solid, clean-code, arquitectura]
---

# 30 — Principios SOLID y Clean Code Aplicados a Machine Learning

> Nota del mentor: SOLID nació en el mundo del software empresarial de los 2000, y muchos ingenieros de ML lo descartan pensando que "es para backend, no para modelos". Con 20 años viendo pipelines de ML pudrirse por falta de estos principios, te puedo asegurar que es exactamente al revés: el código de ML tiende a crecer más rápido y más desordenado que el backend tradicional, precisamente porque nadie le aplica estas reglas.

## 1. S — Single Responsibility Principle (SRP)

**"Una clase o función debe tener una sola razón para cambiar."**

```python
# Viola SRP: esta clase cambia por razones de negocio (fórmula de Erlang C),
# de infraestructura (conexión a BD) Y de ML (entrenamiento) — tres razones distintas
class PipelineForecast:
    def conectar_bd(self): ...
    def entrenar_modelo(self): ...
    def calcular_erlang_c(self): ...
    def enviar_resultado_a_bd(self): ...

# Respeta SRP: cada clase cambia por una sola razón
class RepositorioDatos:
    def conectar(self): ...
    def obtener_historico(self): ...
    def guardar_resultados(self): ...

class EntrenadorModelo:
    def entrenar(self, datos): ...

class CalculadoraStaffing:
    def calcular_erlang_c(self, demanda, tiempo_servicio): ...
```

Ya viviste esto directamente: tu refactor de `Conexion.py` en `database.py`, `forecasting.py` y `staffing.py` es SRP aplicado, aunque no lo llamaras así en su momento.

## 2. O — Open/Closed Principle (OCP)

**"Abierto a extensión, cerrado a modificación."** Debes poder agregar comportamiento nuevo sin editar código que ya funciona y ya está probado.

```python
# Viola OCP: cada nuevo algoritmo obliga a modificar esta función existente
def entrenar(tipo_modelo: str, X, y):
    if tipo_modelo == "xgboost":
        modelo = XGBRegressor()
    elif tipo_modelo == "gradient_boosting":
        modelo = GradientBoostingRegressor()
    # agregar un tercer modelo = editar esta función, riesgo de romper los dos anteriores
    modelo.fit(X, y)
    return modelo

# Respeta OCP: agregar un modelo nuevo no toca código existente
class BaseForecaster(ABC):
    @abstractmethod
    def fit(self, X, y): ...

class XGBoostForecaster(BaseForecaster):
    def fit(self, X, y): ...

class LightGBMForecaster(BaseForecaster):  # NUEVO — cero cambios en código existente
    def fit(self, X, y): ...
```

Esto conecta directo con Protocols y ABC de [[20-Python-Avanzado-Sistema-de-Tipos]]: el contrato abstracto es lo que permite extender sin modificar.

## 3. L — Liskov Substitution Principle (LSP)

**"Una subclase debe poder reemplazar a su clase base sin romper el comportamiento esperado."**

```python
class BaseForecaster(ABC):
    @abstractmethod
    def predecir(self, X) -> np.ndarray:
        """Debe retornar un array de floats, uno por fila de X."""

# Viola LSP: retorna algo distinto a lo que el contrato promete
class ForecasterRoto(BaseForecaster):
    def predecir(self, X):
        return "no hay predicción disponible"  # rompe el contrato — un string, no un array
```

Cualquier código que use `BaseForecaster` (por ejemplo, tu función `entrenar_y_evaluar` de la nota 20) confía en que **cualquier** subclase se comporta según el contrato declarado. Si una subclase viola esa expectativa, el error aparece lejos de donde realmente está el problema — un bug muy difícil de rastrear.

## 4. I — Interface Segregation Principle (ISP)

**"Ninguna clase debe verse forzada a implementar métodos que no usa."**

```python
# Viola ISP: obliga a todo forecaster a implementar explain(), aunque no todos lo soporten
class BaseForecaster(ABC):
    @abstractmethod
    def fit(self, X, y): ...
    @abstractmethod
    def predict(self, X): ...
    @abstractmethod
    def explain(self, X): ...  # SHAP no está disponible para todos los modelos igual

# Respeta ISP: interfaces separadas, opt-in
class BaseForecaster(ABC):
    @abstractmethod
    def fit(self, X, y): ...
    @abstractmethod
    def predict(self, X): ...

class Interpretable(Protocol):
    def explain(self, X) -> dict: ...

class XGBoostForecaster(BaseForecaster, Interpretable):
    def explain(self, X): ...  # solo implementa esto quien realmente lo soporta
```

## 5. D — Dependency Inversion Principle (DIP)

**"Depende de abstracciones, no de implementaciones concretas."** Este es el principio detrás del patrón de Dependency Injection que profundizamos en [[31-Patrones-de-Diseno-Utiles-para-ML]].

```python
# Viola DIP: forecasting.py depende directamente de pyodbc y SQL Server
class ServicioForecast:
    def __init__(self):
        self.conn = pyodbc.connect("...")  # acoplado a una tecnología concreta

# Respeta DIP: depende de una abstracción (Protocol), no de pyodbc directamente
class RepositorioDatos(Protocol):
    def obtener_historico(self, office_id: int) -> pd.DataFrame: ...

class ServicioForecast:
    def __init__(self, repositorio: RepositorioDatos):
        self.repositorio = repositorio  # no le importa si es SQL Server, Postgres o un mock
```

La ganancia práctica: en tus tests (ver [[25-Testing-Avanzado]]) puedes inyectar un repositorio falso sin tocar SQL Server real, y si algún día migran de SQL Server a otra base de datos, `ServicioForecast` no cambia una sola línea.

## 6. Clean Code — principios prácticos más allá de SOLID

- **Nombres que revelan intención**: `dias_atras` es mejor que `d`; `calcular_agentes_requeridos()` es mejor que `calc()`. El costo de escribir un nombre largo es mínimo comparado con el costo de que alguien (tú mismo en 6 meses) no entienda qué hace algo.
- **Funciones pequeñas, un solo nivel de abstracción**: si una función mezcla "leer datos de SQL" con "calcular una fórmula matemática compleja" con "loguear resultados", está mezclando niveles de abstracción distintos — sepárala.
- **Evitar comentarios que explican "qué" hace el código**: si necesitas un comentario para explicar qué hace una línea, casi siempre es mejor reescribir la línea con nombres más claros. Los comentarios buenos explican **por qué** ("no usamos fast_executemany aquí por el error 22003 de overflow"), no **qué**.
- **Evitar números y strings mágicos**: `if dias > 90:` es peor que `if dias > VENTANA_MAXIMA_ENTRENAMIENTO:` — el segundo se explica solo, y si el valor cambia, se cambia en un solo lugar.

## 7. El balance — SOLID no es dogma, es criterio

Un error común de quien recién aprende SOLID es aplicarlo religiosamente hasta crear abstracciones prematuras sobre código que probablemente nunca necesitará esa flexibilidad. La pregunta correcta no es "¿esto cumple SOLID?", es "¿este nivel de abstracción resuelve un problema real que tengo hoy, o estoy anticipando un problema hipotético que quizás nunca llegue?". Aplica SOLID donde el cambio es probable (algoritmos de ML que cambiarán, fuentes de datos que podrían migrar); no lo apliques donde el código es genuinamente estable y simple.

## Ver también
- [[20-Python-Avanzado-Sistema-de-Tipos]]
- [[22-Organizacion-del-Codigo-y-Principios-de-Diseno]]
- [[31-Patrones-de-Diseno-Utiles-para-ML]]
- [[32-Diseno-Composicion-Herencia-Interfaces]]
