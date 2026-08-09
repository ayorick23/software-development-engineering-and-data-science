---
tags: [patrones-de-diseno, ingenieria-de-software, arquitectura]
---

# 31 — Patrones de Diseño Útiles para Machine Learning (Factory, Strategy, Repository, Dependency Injection)

> Nota del mentor: existen 23 patrones de diseño clásicos del libro de la Gang of Four. En 20 años de carrera, uso activamente cuatro o cinco en proyectos de ML de forma recurrente. Estos son esos — aprenderlos bien vale más que memorizar los 23 a medias.

## 1. Factory Pattern — centralizar la creación de objetos complejos

**Problema que resuelve**: cuando crear un objeto (un modelo, una conexión, un validador) requiere lógica condicional que, sin control, termina repetida en múltiples lugares del código.

```python
class ForecasterFactory:
    _registro: dict[str, type[BaseForecaster]] = {
        "xgboost": XGBoostForecaster,
        "gradient_boosting": GradientBoostingForecaster,
        "lightgbm": LightGBMForecaster,
    }

    @classmethod
    def crear(cls, tipo_modelo: str, **hiperparametros) -> BaseForecaster:
        if tipo_modelo not in cls._registro:
            raise ValueError(f"Tipo de modelo desconocido: {tipo_modelo}")
        return cls._registro[tipo_modelo](**hiperparametros)

# Uso — el resto del código nunca necesita conocer las clases concretas
modelo = ForecasterFactory.crear("xgboost", n_estimators=300, max_depth=6)
```

En tu sistema de autoaprendizaje, esto es exactamente lo que necesitarías si algún día quisieras que el challenger pruebe distintos tipos de algoritmo (no solo distintas ventanas de días) — el resto del pipeline (evaluación, gate de MAE/RMSE, registro en MLflow) no cambia una sola línea, sin importar qué tipo de modelo se esté probando.

## 2. Strategy Pattern — intercambiar algoritmos en tiempo de ejecución

**Problema que resuelve**: cuando necesitas que el **comportamiento** (no solo el objeto) sea intercambiable — por ejemplo, distintas formas de validar un modelo antes de promoverlo.

```python
class EstrategiaValidacion(Protocol):
    def es_valido(self, metricas_challenger: dict, metricas_champion: dict) -> bool: ...

class ValidacionPorMAE(EstrategiaValidacion):
    def __init__(self, margen_minimo: float = 0.02):
        self.margen_minimo = margen_minimo

    def es_valido(self, metricas_challenger, metricas_champion) -> bool:
        mejora = (metricas_champion["mae"] - metricas_challenger["mae"]) / metricas_champion["mae"]
        return mejora >= self.margen_minimo

class ValidacionPorMAEyRMSE(EstrategiaValidacion):
    def __init__(self, margen_mae: float = 0.02, tolerancia_rmse: float = 0.05):
        self.margen_mae = margen_mae
        self.tolerancia_rmse = tolerancia_rmse

    def es_valido(self, metricas_challenger, metricas_champion) -> bool:
        mejora_mae = (metricas_champion["mae"] - metricas_challenger["mae"]) / metricas_champion["mae"]
        empeora_rmse = (metricas_challenger["rmse"] - metricas_champion["rmse"]) / metricas_champion["rmse"]
        return mejora_mae >= self.margen_mae and empeora_rmse <= self.tolerancia_rmse

class SistemaAutoaprendizaje:
    def __init__(self, estrategia: EstrategiaValidacion):
        self.estrategia = estrategia  # intercambiable sin tocar el resto de la clase

    def evaluar_promocion(self, challenger_metrics, champion_metrics) -> bool:
        return self.estrategia.es_valido(challenger_metrics, champion_metrics)
```

Esto formaliza exactamente el gate de MAE/RMSE que ya tienes — la ventaja es que si mañana el negocio pide un criterio distinto para Demand vs. Service Time (una decisión que discutiste con tu jefa), simplemente instancias `SistemaAutoaprendizaje` con una estrategia distinta para cada modelo, sin duplicar ni modificar la lógica de orquestación.

## 3. Repository Pattern — separar el "qué datos necesito" del "cómo se obtienen"

**Problema que resuelve**: acoplar tu lógica de negocio directamente a SQL Server/pyodbc hace imposible testear sin una base de datos real, y dificulta migrar de tecnología después.

```python
class RepositorioForecast(Protocol):
    def obtener_historico(self, office_id: int, dias_atras: int) -> pd.DataFrame: ...
    def guardar_resultados(self, resultados: pd.DataFrame) -> None: ...

class RepositorioForecastSQLServer:
    def __init__(self, connection_string: str):
        self.connection_string = connection_string

    def obtener_historico(self, office_id, dias_atras) -> pd.DataFrame:
        query = "SELECT * FROM qf.hrCallData WHERE office_id = ? AND interval_start >= ?"
        with pyodbc.connect(self.connection_string) as conn:
            return pd.read_sql(query, conn, params=[office_id, fecha_limite(dias_atras)])

    def guardar_resultados(self, resultados: pd.DataFrame) -> None:
        # lógica de staging + MERGE, ver nota 29
        ...

class RepositorioForecastEnMemoria:
    """Para tests — sin tocar SQL Server real."""
    def __init__(self, datos_de_prueba: pd.DataFrame):
        self.datos = datos_de_prueba

    def obtener_historico(self, office_id, dias_atras) -> pd.DataFrame:
        return self.datos[self.datos["office_id"] == office_id]

    def guardar_resultados(self, resultados: pd.DataFrame) -> None:
        self.resultados_guardados = resultados  # solo lo guarda en memoria para verificar en el test
```

El Repository Pattern es la aplicación directa del Dependency Inversion Principle de [[30-Principios-SOLID-y-Clean-Code-para-ML]] — tu módulo `database.py` ya cumple parcialmente este rol; formalizarlo con una interfaz explícita (`Protocol`) es lo que te permite testear `forecasting.py` sin conexión real a Claro RD.

## 4. Dependency Injection — no construyas tus dependencias, recíbelas

```python
# Sin DI: la clase construye sus propias dependencias — difícil de testear y cambiar
class ServicioForecast:
    def __init__(self):
        self.repositorio = RepositorioForecastSQLServer("connection_string_hardcodeada")
        self.estrategia = ValidacionPorMAEyRMSE()

# Con DI: las dependencias se reciben desde afuera
class ServicioForecast:
    def __init__(self, repositorio: RepositorioForecast, estrategia: EstrategiaValidacion):
        self.repositorio = repositorio
        self.estrategia = estrategia

# En producción:
servicio = ServicioForecast(
    repositorio=RepositorioForecastSQLServer(config.db_connection_string),
    estrategia=ValidacionPorMAEyRMSE(margen_mae=config.margen_mejora_minima),
)

# En un test:
servicio_de_prueba = ServicioForecast(
    repositorio=RepositorioForecastEnMemoria(datos_de_prueba),
    estrategia=ValidacionPorMAEyRMSE(margen_mae=0.01),
)
```

DI no requiere ningún framework especial en Python — es simplemente **pasar las dependencias como parámetros del constructor** en vez de crearlas dentro de la clase. Este cambio, aparentemente simple, es lo que hace posible que [[25-Testing-Avanzado]] (mocks y fixtures) funcione limpiamente: inyectas un repositorio falso sin tener que "engañar" al código con parches de mock complejos.

## 5. Cuándo NO usar estos patrones

Aplicar Factory, Strategy, Repository y DI a un script de 40 líneas de análisis exploratorio es sobre-ingeniería — el costo de la abstracción supera el beneficio. Estos patrones ganan su valor en código que:

- Se ejecuta en producción de forma recurrente (tu pipeline de Claro RD, exactamente).
- Necesita tests confiables sin dependencias externas reales.
- Tiene probabilidad real de necesitar variantes (distintos modelos, distintas fuentes de datos, distintos criterios de negocio).

## Ver también
- [[30-Principios-SOLID-y-Clean-Code-para-ML]]
- [[25-Testing-Avanzado]]
- [[32-Diseno-Composicion-Herencia-Interfaces]]
