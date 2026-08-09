---
tags: [testing, pytest, mocks, avanzado]
---

# 25 — Testing Avanzado: Fixtures, Mocks, Coverage y Parametrize

> Nota del mentor: en [[13-Testing-en-Machine-Learning]] vimos el "qué" y el "por qué" del testing en ML. Aquí vamos al "cómo" con las herramientas concretas de `pytest` que vas a usar todos los días. El objetivo no es que te conviertas en QA — es que tus pipelines fallen rápido y de forma clara cuando algo se rompe, en vez de fallar en silencio en producción tres semanas después.

## 1. Fixtures — preparar el mismo contexto sin repetir código

```python
import pytest
import pandas as pd

@pytest.fixture
def datos_demanda_ejemplo() -> pd.DataFrame:
    """Datos de ejemplo reutilizables en múltiples tests."""
    return pd.DataFrame({
        "office_id": [145, 145, 145],
        "interval_start": pd.date_range("2026-01-01", periods=3, freq="30min"),
        "total_demand": [12.0, 15.0, 8.0],
        "avg_service_time": [180.0, 195.0, 170.0],
    })

def test_calcular_lag_features(datos_demanda_ejemplo):
    resultado = build_lag_features(datos_demanda_ejemplo, lags=[1])
    assert resultado.shape[0] == 3

def test_validar_schema_datos_correctos(datos_demanda_ejemplo):
    # no debe lanzar excepción
    validate_input(datos_demanda_ejemplo)
```

Sin fixtures, tendrías que reconstruir el mismo DataFrame de ejemplo en cada función de test — repetición que, cuando cambias la estructura de datos, obliga a actualizar N lugares en vez de uno.

### Fixtures con scope — controlar cuándo se recrean

```python
@pytest.fixture(scope="module")
def conexion_bd_pruebas():
    """Se crea una sola vez por archivo de test, no por cada test individual."""
    conn = crear_conexion_sqlite_en_memoria()
    yield conn
    conn.close()
```

`scope="function"` (default) recrea el fixture en cada test — más aislado, más lento. `scope="module"` o `scope="session"` lo comparten entre tests — más rápido, pero cuidado con el estado compartido entre tests que puede causar dependencias ocultas entre ellos.

## 2. Mocks — aislar tu código de dependencias externas lentas o no disponibles

```python
from unittest.mock import Mock, patch

def test_entrenar_modelo_registra_en_mlflow():
    with patch("forecasting_pipeline.forecasting.mlflow") as mlflow_mock:
        entrenar_modelo(datos_ejemplo, config_ejemplo)
        mlflow_mock.log_metric.assert_called_with("mae", pytest.approx(12.4, abs=0.1))
        mlflow_mock.xgboost.log_model.assert_called_once()

def test_pipeline_reintenta_en_fallo_de_conexion():
    conexion_mock = Mock(side_effect=[ConnectionError(), ConnectionError(), "conexion_exitosa"])
    resultado = conectar_con_reintentos(conexion_mock, intentos=3)
    assert resultado == "conexion_exitosa"
    assert conexion_mock.call_count == 3
```

¿Por qué mockear? Porque un test unitario **no debería** necesitar una conexión real a la base de datos de Claro RD, ni un servidor de MLflow corriendo, ni esperar 4 minutos de entrenamiento real de XGBoost — eso son pruebas de integración, una categoría distinta y más lenta. Un mock reemplaza la dependencia externa por un objeto controlado que simula el comportamiento que necesitas probar (incluyendo fallos, como el `ConnectionError` simulado arriba).

**Regla práctica**: mockea las fronteras de tu sistema (base de datos, APIs externas, MLflow, sistema de archivos), nunca la lógica de negocio que estás probando — si mockeas demasiado, terminas probando que tus mocks funcionan, no que tu código funciona.

## 3. `parametrize` — un test, múltiples casos, sin duplicar código

```python
@pytest.mark.parametrize("dias_atras,dias_validacion,valido", [
    (90, 14, True),
    (14, 90, False),   # validación mayor que entrenamiento — inválido
    (0, 14, False),     # dias_atras en cero — inválido
    (365, 30, True),
])
def test_validacion_configuracion(dias_atras, dias_validacion, valido):
    if valido:
        config = ConfiguracionEntrenamiento(dias_atras, dias_validacion)
        assert config.dias_atras == dias_atras
    else:
        with pytest.raises(ValueError):
            ConfiguracionEntrenamiento(dias_atras, dias_validacion)
```

Sin `parametrize`, necesitarías cuatro funciones de test casi idénticas. Con `parametrize`, `pytest` genera cuatro tests independientes automáticamente, cada uno reportado por separado si falla — sabes exactamente cuál combinación de parámetros rompió algo, sin tener que adivinar dentro de un solo test gigante con múltiples asserts.

## 4. Coverage — usarlo con criterio, no como vanidad

```bash
pytest tests/ --cov=src/forecasting_pipeline --cov-report=html --cov-report=term-missing
```

El reporte `term-missing` te dice exactamente qué **líneas** no están cubiertas — úsalo para identificar ramas de código sin probar (ej. el `except DatosInsuficientesError` nunca se ejecutó en tus tests), no como una carrera hacia el 100%. Como ya se mencionó en [[13-Testing-en-Machine-Learning]]: cobertura alta con asserts débiles (`assert resultado is not None` sin verificar el contenido real) da una falsa sensación de seguridad — es peor que saber honestamente que algo no está cubierto.

## 5. Estructura recomendada de la suite de tests

```
tests/
├── conftest.py              # fixtures compartidas entre todos los tests
├── unit/
│   ├── test_feature_engineering.py
│   ├── test_staffing.py
│   └── test_config.py
├── integration/
│   ├── test_database_connection.py   # requiere BD real, más lento
│   └── test_mlflow_logging.py         # requiere servidor MLflow real
└── data_quality/
    └── test_input_schema.py           # validaciones tipo Pandera/Great Expectations
```

`conftest.py` es un archivo especial de `pytest` — las fixtures definidas ahí están disponibles automáticamente en todos los tests del directorio, sin necesidad de importarlas explícitamente. Separar `unit/` de `integration/` te permite correr solo los tests rápidos en cada commit (`pytest tests/unit`) y reservar los lentos para el pipeline de CI completo o ejecuciones nocturnas — exactamente la separación de stages rápidos/lentos que vimos en [[14-CICD-para-ML-con-GitLab]].

## 6. El test que más vale la pena escribir primero

Si tuvieras que priorizar con tiempo limitado, prioriza siempre un **test de regresión** para cada bug real que ya encontraste — como el bug de `warm_start` en tu sistema de autoaprendizaje. Ese test específico (`test_warmstart_reutiliza_arboles_existentes`) vale más que diez tests genéricos de casos hipotéticos, porque garantiza que un error que **ya te costó tiempo real** nunca vuelva a colarse sin que nadie se entere.

## Ver también
- [[13-Testing-en-Machine-Learning]]
- [[23-Manejo-Profesional-de-Errores]]
- [[14-CICD-para-ML-con-GitLab]]
