---
tags: [diseno-de-software, arquitectura, herencia, composicion]
---

# 32 — Diseño de Software: Composición, Herencia, Interfaces y Abstracciones

> Nota del mentor: la pregunta "¿herencia o composición?" es una de las decisiones de diseño que más impacto tiene a largo plazo, y la respuesta correcta casi siempre sorprende a quien viene de una formación clásica de POO: **prefiere composición sobre herencia**, casi siempre. Esta nota te explica por qué, con casos reales de ML.

## 1. Herencia — "es un tipo de"

```python
class BaseForecaster(ABC):
    @abstractmethod
    def fit(self, X, y): ...
    @abstractmethod
    def predict(self, X): ...

class XGBoostForecaster(BaseForecaster):  # "un XGBoostForecaster ES UN BaseForecaster"
    def fit(self, X, y):
        self.modelo = XGBRegressor()
        self.modelo.fit(X, y)
    def predict(self, X):
        return self.modelo.predict(X)
```

La herencia es correcta cuando la relación **"es un tipo de"** es genuinamente verdadera y estable en el tiempo. `XGBoostForecaster` es, sin ambigüedad, un tipo de `BaseForecaster` — esa relación no va a cambiar.

### El problema real de la herencia — jerarquías rígidas y frágiles

```python
class BaseForecaster(ABC):
    def entrenar_y_registrar(self, X, y):
        self.fit(X, y)
        self.registrar_en_mlflow()  # ¿todo forecaster DEBE usar MLflow?
        self.enviar_notificacion()   # ¿todo forecaster DEBE notificar?
```

Cuando metes demasiado comportamiento en la clase base, cada subclase hereda **todo**, incluso lo que no necesita — y cambiar la clase base afecta a todas las subclases simultáneamente, incluso a las que nunca debieron verse afectadas. Esto es "el problema del yo-yo": para entender qué hace una subclase, tienes que saltar constantemente entre ella y varios niveles de clases padre.

## 2. Composición — "tiene un"

```python
class ServicioForecast:  # "un ServicioForecast TIENE UN modelo, TIENE UN registrador"
    def __init__(self, modelo: BaseForecaster, registrador: "RegistradorMLflow"):
        self.modelo = modelo
        self.registrador = registrador

    def entrenar_y_registrar(self, X, y):
        self.modelo.fit(X, y)
        self.registrador.registrar(self.modelo)
```

Con composición, `ServicioForecast` **usa** un modelo y un registrador, en vez de **ser** un tipo especial de ellos. Puedes combinar cualquier modelo con cualquier registrador sin crear una explosión de subclases (`XGBoostForecasterConMLflow`, `XGBoostForecasterSinMLflow`, `LightGBMForecasterConMLflow`...) — el problema clásico de "explosión combinatoria de herencia" que la composición evita por completo.

### Regla práctica: "has-a" vs "is-a"

- ¿Es genuinamente cierto que X **es un tipo de** Y, y esa relación nunca cambiará? → herencia (`XGBoostForecaster` es un `BaseForecaster`).
- ¿X **usa o contiene** funcionalidad de Y, pero podría combinarse con otras variantes? → composición (`ServicioForecast` usa un `modelo` y un `registrador`, intercambiables independientemente).

## 3. Interfaces — el contrato sin la implementación

En Python, las interfaces se expresan con `Protocol` (tipado estructural, visto en [[20-Python-Avanzado-Sistema-de-Tipos]]) o con `ABC` (herencia explícita). La diferencia de diseño clave: una interfaz define **qué** debe poder hacer un objeto, nunca **cómo** lo hace.

```python
class Predictor(Protocol):
    def predict(self, X) -> np.ndarray: ...

def evaluar(modelo: Predictor, X_val, y_val) -> float:
    predicciones = modelo.predict(X_val)
    return mean_absolute_error(y_val, predicciones)
```

`evaluar()` no sabe ni le importa si `modelo` es XGBoost, un modelo lineal, o una red neuronal — solo le importa que tenga `.predict()`. Esto es lo que te permite escribir código de evaluación **una sola vez**, reutilizable para cualquier algoritmo presente o futuro.

## 4. Abstracciones — el nivel correcto de generalidad

Una abstracción mal calibrada es tan dañina como no tener ninguna. Dos errores opuestos y ambos comunes:

- **Abstracción prematura**: crear una jerarquía de clases elaborada para "cualquier tipo de modelo futuro posible" cuando en realidad solo usas XGBoost y probablemente lo seguirás usando por años. Esto agrega complejidad sin beneficio real — código más difícil de leer para resolver un problema que no existe todavía.
- **Ausencia de abstracción**: código duplicado por todos lados porque nunca se identificó el patrón común — exactamente el problema que resolviste al notar que dos notebooks de entrenamiento reimplementaban la misma lógica de feature engineering cuatro veces.

**Regla práctica (la "regla de tres")**: la primera vez que escribes algo, escríbelo directo, sin abstraer. La segunda vez que necesitas algo similar, tolera la duplicación un poco más y presta atención al patrón que se repite. La tercera vez que ves la misma necesidad, ahí sí abstrae — para ese punto ya tienes evidencia real de qué varía y qué es constante, en vez de estar adivinando.

## 5. Ejemplo integrador — cómo se ven estos conceptos juntos en tu pipeline

```python
# Interfaces (Protocols) — el contrato
class RepositorioDatos(Protocol):
    def obtener_historico(self, office_id: int) -> pd.DataFrame: ...

class Predictor(Protocol):
    def fit(self, X, y) -> None: ...
    def predict(self, X) -> np.ndarray: ...

# Herencia — familia real de modelos (relación "es un")
class BaseForecaster(ABC):
    @abstractmethod
    def fit(self, X, y) -> None: ...
    @abstractmethod
    def predict(self, X) -> np.ndarray: ...

class XGBoostForecaster(BaseForecaster): ...
class LightGBMForecaster(BaseForecaster): ...

# Composición — orquestación real del pipeline (relación "tiene un")
class PipelineForecast:
    def __init__(
        self,
        repositorio: RepositorioDatos,
        modelo: Predictor,
        estrategia_validacion: EstrategiaValidacion,
    ):
        self.repositorio = repositorio
        self.modelo = modelo
        self.estrategia_validacion = estrategia_validacion

    def ejecutar(self, office_id: int):
        datos = self.repositorio.obtener_historico(office_id)
        self.modelo.fit(datos[features], datos[target])
        ...
```

`PipelineForecast` no hereda de nada — está **compuesto** de piezas intercambiables, cada una definida por una interfaz clara. Esta es la arquitectura que hace posible testear cada pieza por separado ([[25-Testing-Avanzado]]), intercambiar implementaciones sin reescribir el orquestador ([[31-Patrones-de-Diseno-Utiles-para-ML]]), y entender el sistema completo leyendo el constructor de una sola clase.

## Ver también
- [[20-Python-Avanzado-Sistema-de-Tipos]]
- [[30-Principios-SOLID-y-Clean-Code-para-ML]]
- [[31-Patrones-de-Diseno-Utiles-para-ML]]
