---
tags: [python, tipado, typing, avanzado]
---

# 20 — Python Avanzado: El Sistema de Tipos (`typing`, Generics, Protocols, Dataclasses, Enums, ABC)

> Nota del mentor: cuando empecé en esto, Python "sin tipos" era una ventaja de velocidad de desarrollo. Con 20 años de ver proyectos crecer de 200 a 20,000 líneas, te puedo asegurar que esa "ventaja" se convierte en la peor pesadilla de mantenimiento si no la controlas. El tipado estático de Python no es una moda de Java disfrazada — es lo que te permite que tu editor, tus tests y tus compañeros de equipo entiendan qué espera y qué devuelve cada función de tu pipeline **sin tener que leer la implementación completa**.

## 1. `typing` — lo esencial que debes dominar de memoria

```python
from typing import Optional, Union
import pandas as pd
import numpy as np

def calcular_erlang_c(
    demanda: float,
    tiempo_servicio: float,
    agentes: int,
    nivel_servicio_objetivo: float = 0.8,
) -> tuple[int, float]:
    """Retorna (agentes_requeridos, nivel_servicio_estimado)."""
    ...

def cargar_modelo(ruta: str) -> Optional[object]:
    """Retorna None si el modelo no existe en la ruta."""
    ...

# Python 3.10+: unión con el operador |, más legible que Union
def procesar(valor: int | float | None) -> str:
    ...
```

Reglas prácticas de un senior:

- **Tipa las fronteras de tu código, no cada línea interna.** Firmas de funciones públicas, parámetros de clases, retornos — ahí el tipado da el 90% del valor. Tipar cada variable local dentro de una función de 5 líneas es ruido.
- **`Optional[X]` significa "X o None", nunca "puede que no exista el parámetro".** Es uno de los errores más comunes de quien viene de otros lenguajes.
- **`mypy` o `pyright` validan esto en CI** (ver [[14-CICD-para-ML-con-GitLab]]) — el tipado sin un checker automático es solo documentación que nadie garantiza que siga siendo cierta.

## 2. Generics — cuando una función/clase debe funcionar con cualquier tipo, pero de forma consistente

```python
from typing import TypeVar, Generic

T = TypeVar("T")

class ModelRegistry(Generic[T]):
    def __init__(self) -> None:
        self._modelos: dict[str, T] = {}

    def registrar(self, nombre: str, modelo: T) -> None:
        self._modelos[nombre] = modelo

    def obtener(self, nombre: str) -> T:
        return self._modelos[nombre]
```

`ModelRegistry[XGBRegressor]` y `ModelRegistry[GradientBoostingRegressor]` son ambos válidos, y el checker de tipos sabe que `obtener()` devuelve exactamente el tipo con el que se instanció — no `object` genérico que obligaría a hacer casts manuales en cada uso.

## 3. Protocols — tipado estructural ("duck typing" con garantías)

Este es el concepto que más cuesta a quien viene de POO clásica. Un `Protocol` define **qué métodos debe tener** un objeto, sin exigir herencia explícita:

```python
from typing import Protocol

class Predictor(Protocol):
    def fit(self, X, y) -> "Predictor": ...
    def predict(self, X) -> np.ndarray: ...

def entrenar_y_evaluar(modelo: Predictor, X_train, y_train, X_val, y_val) -> float:
    modelo.fit(X_train, y_train)
    predicciones = modelo.predict(X_val)
    return mean_absolute_error(y_val, predicciones)
```

Esta función acepta **cualquier objeto** que tenga `.fit()` y `.predict()` — un `XGBRegressor`, un `GradientBoostingRegressor`, o un modelo propio que tú escribas — sin que ninguno tenga que heredar de una clase base común. Es exactamente el contrato implícito que ya usa `scikit-learn` internamente, pero ahora tú lo puedes declarar explícitamente y que tu checker de tipos lo valide.

## 4. Dataclasses — para dejar de escribir `__init__` a mano

```python
from dataclasses import dataclass, field

@dataclass
class ConfiguracionEntrenamiento:
    dias_atras: int = 90
    dias_validacion: int = 14
    margen_mejora_minima: float = 0.02
    hiperparametros: dict = field(default_factory=dict)

    def __post_init__(self):
        if self.dias_atras <= self.dias_validacion:
            raise ValueError("dias_atras debe ser mayor que dias_validacion")
```

Comparado con una clase manual, `@dataclass` te genera automáticamente `__init__`, `__repr__` y `__eq__`. En tu proyecto de Claro RD, esto es exactamente lo que debería reemplazar los diccionarios sueltos de configuración que probablemente ves en `config.py` — un diccionario no valida tipos ni valores; un dataclass sí (con `__post_init__`).

## 5. Enums — cuando un valor solo puede ser uno de un conjunto fijo

```python
from enum import Enum, auto

class EstadoModelo(Enum):
    CHAMPION = auto()
    CHALLENGER = auto()
    ARCHIVADO = auto()

class TipoMetrica(str, Enum):
    MAE = "mae"
    RMSE = "rmse"
    WAPE = "wape"
```

En vez de comparar strings mágicos (`if estado == "champion"`, propenso a typos silenciosos), usas `EstadoModelo.CHAMPION` — el checker de tipos detecta un error de escritura en tiempo de desarrollo, no en producción a las 2am.

## 6. Pathlib — nunca más concatenar rutas con strings

```python
from pathlib import Path

ruta_modelos = Path("models") / "claro_rd" / "demand"
ruta_modelos.mkdir(parents=True, exist_ok=True)
archivo_modelo = ruta_modelos / f"xgboost_v{version}.pkl"

if archivo_modelo.exists():
    logger.warning(f"Sobrescribiendo modelo existente: {archivo_modelo}")
```

`pathlib` es multiplataforma (Windows/Linux) por diseño — importante si tu equipo desarrolla en Windows pero el runner de GitLab CI corre en contenedores Linux, como suele pasar.

## 7. Abstract Base Classes (ABC) — el contrato formal, cuando `Protocol` no basta

Mientras `Protocol` es implícito (no requiere herencia), `ABC` es explícito y **obliga** a las subclases a implementar ciertos métodos, lanzando error si no lo hacen:

```python
from abc import ABC, abstractmethod

class BaseForecaster(ABC):
    @abstractmethod
    def entrenar(self, datos: pd.DataFrame) -> None: ...

    @abstractmethod
    def predecir(self, datos: pd.DataFrame) -> pd.DataFrame: ...

    def validar_entrada(self, datos: pd.DataFrame) -> None:
        """Método concreto compartido — no abstracto."""
        if datos.empty:
            raise ValueError("El DataFrame de entrada está vacío")

class DemandForecaster(BaseForecaster):
    def entrenar(self, datos: pd.DataFrame) -> None:
        ...
    def predecir(self, datos: pd.DataFrame) -> pd.DataFrame:
        ...
```

**¿Cuándo `ABC` y cuándo `Protocol`?** Regla práctica: usa `ABC` cuando controlas toda la jerarquía de clases y quieres compartir lógica concreta además del contrato (como `validar_entrada` arriba). Usa `Protocol` cuando quieres aceptar objetos de librerías externas (scikit-learn, XGBoost) que nunca van a heredar de tu clase.

## Ver también
- [[12-Gestion-Moderna-de-Proyectos-Python]]
- [[21-Python-Avanzado-Ejecucion-y-Metaprogramacion]]
- [[30-Principios-SOLID-y-Clean-Code-para-ML]]
