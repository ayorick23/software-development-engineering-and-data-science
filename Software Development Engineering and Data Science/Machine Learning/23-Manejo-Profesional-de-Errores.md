---
tags: [python, excepciones, logging, buenas-practicas]
---

# 23 — Manejo Profesional de Errores: Excepciones Propias, Encadenamiento y Logging Correcto

> Nota del mentor: el manejo de errores es donde más se nota la diferencia entre código de notebook y código de producción. En un notebook, un error simplemente detiene la celda y tú lees el traceback. En un pipeline que corre solo cada 30 minutos sin supervisión humana, un manejo de errores pobre significa que el sistema falla en silencio, o peor, sigue corriendo con datos corruptos sin que nadie se entere hasta que el negocio lo nota.

## 1. Excepciones propias — dar nombre a los errores de tu dominio

```python
class ForecastingPipelineError(Exception):
    """Excepción base para todos los errores del pipeline."""

class DatosInsuficientesError(ForecastingPipelineError):
    """Se lanza cuando no hay suficiente historial para entrenar."""
    def __init__(self, office_id: int, filas_disponibles: int, filas_minimas: int):
        self.office_id = office_id
        self.filas_disponibles = filas_disponibles
        self.filas_minimas = filas_minimas
        super().__init__(
            f"Oficina {office_id}: solo {filas_disponibles} filas disponibles, "
            f"se requieren mínimo {filas_minimas}"
        )

class ModeloNoSuperaGateError(ForecastingPipelineError):
    """Se lanza cuando el challenger no supera al champion."""
```

¿Por qué no usar simplemente `raise ValueError("faltan datos")`? Porque una excepción propia te permite:

- **Capturar selectivamente**: `except DatosInsuficientesError` sin atrapar accidentalmente otros `ValueError` no relacionados que podrían ocultar bugs reales.
- **Adjuntar contexto estructurado** (`office_id`, `filas_disponibles`) que tu logging puede capturar y que tu sistema de alertas puede usar para decidir la severidad.
- **Documentar el dominio de errores posible de tu pipeline** — cualquier compañero nuevo puede leer las clases de excepción y entender de inmediato qué puede salir mal.

## 2. Jerarquía de excepciones — no todo error es igual de grave

```python
class ForecastingPipelineError(Exception): ...
class ErrorRecuperable(ForecastingPipelineError): ...       # reintentar tiene sentido
class ErrorNoRecuperable(ForecastingPipelineError): ...      # detener el pipeline

class ConexionBaseDatosError(ErrorRecuperable): ...
class SchemaInvalidoError(ErrorNoRecuperable): ...
```

Esta jerarquía te permite escribir lógica de manejo genérica pero correcta:

```python
try:
    ejecutar_pipeline()
except ErrorRecuperable as e:
    logger.warning(f"Error recuperable, reintentando: {e}")
    reintentar_con_backoff()
except ErrorNoRecuperable as e:
    logger.critical(f"Error no recuperable, deteniendo pipeline: {e}")
    notificar_a_equipo(e)
    raise
```

## 3. Encadenamiento de excepciones (`raise ... from ...`) — no pierdas la causa raíz

```python
def cargar_configuracion(ruta: str) -> dict:
    try:
        with open(ruta) as f:
            return yaml.safe_load(f)
    except FileNotFoundError as e:
        raise ConfiguracionNoEncontradaError(
            f"No se encontró el archivo de configuración en {ruta}"
        ) from e
```

Sin el `from e`, cuando este error llegue a tus logs solo verás el traceback de `ConfiguracionNoEncontradaError`, perdiendo el rastro de **qué causó originalmente el problema** (`FileNotFoundError`). Con `from e`, Python conserva ambos tracebacks encadenados — la excepción original aparece como "The above exception was the direct cause of the following exception", información que puede ahorrarte media hora de investigación en un incidente real.

## 4. Qué NO hacer — anti-patrones que vas a encontrar en código legado

```python
# ANTI-PATRÓN 1: silenciar el error sin dejar rastro
try:
    resultado = calcular_forecast()
except Exception:
    pass  # el bug sigue ahí, ahora invisible

# ANTI-PATRÓN 2: capturar Exception genérico sin necesidad
try:
    conn = conectar_bd()
except Exception as e:  # ¿qué error específico esperas? ¿KeyError? ¿ConnectionError?
    logger.error(str(e))

# ANTI-PATRÓN 3: perder el traceback original
try:
    entrenar_modelo()
except Exception as e:
    raise RuntimeError("Falló el entrenamiento")  # sin "from e" — perdiste el rastro
```

La regla de oro: **captura la excepción más específica posible**, y si de verdad necesitas capturar `Exception` genérico (por ejemplo, en el punto de entrada más externo del pipeline, como red de seguridad final), siempre loguea el traceback completo con `logger.exception()` (no `logger.error()`), que incluye automáticamente el stack trace.

## 5. Logging correcto de excepciones

```python
import logging
logger = logging.getLogger(__name__)

def procesar_oficina(office_id: int):
    try:
        datos = obtener_datos_historicos(office_id)
        validar_schema(datos)
        entrenar_y_predecir(datos)
    except DatosInsuficientesError as e:
        logger.warning(
            "Saltando oficina por datos insuficientes",
            extra={"extra_fields": {"office_id": office_id, "error": str(e)}}
        )
        return  # continúa con la siguiente oficina, no detiene todo el batch
    except Exception:
        logger.exception(f"Error inesperado procesando oficina {office_id}")
        raise  # error no anticipado: sí detener y escalar
```

Nota el patrón: un error **esperado y de negocio conocido** (`DatosInsuficientesError`) se loguea como `WARNING` y el pipeline **continúa** con la siguiente oficina — no tiene sentido detener el forecast de las 200 oficinas de Claro RD porque una tiene datos insuficientes. Un error **inesperado** se loguea con `logger.exception()` (que captura el traceback completo automáticamente) y se re-lanza, porque no sabes con certeza si es seguro continuar.

## 6. Conexión con el resto del stack

Este manejo de errores es lo que le da sustancia real a tu logging de [[11-Logging-en-Python-para-ML]] — un logging bien configurado sin excepciones bien diseñadas termina registrando mensajes genéricos e inútiles ("Error: 'NoneType' object is not subscriptable") en vez de errores de negocio claros y accionables. Y es también la base de por qué tus jobs de [[14-CICD-para-ML-con-GitLab]] pueden diferenciar entre "falló y hay que reintentar" y "falló y hay que alertar a un humano ya".

## Ver también
- [[11-Logging-en-Python-para-ML]]
- [[13-Testing-en-Machine-Learning]]
- [[22-Organizacion-del-Codigo-y-Principios-de-Diseno]]
