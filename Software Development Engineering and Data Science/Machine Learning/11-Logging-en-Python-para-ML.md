---
tags: [mlops, python, logging, buenas-practicas]
---

# 11 — Logging en Python para Proyectos de Machine Learning

> Nota del mentor: en tus primeros días viste `print()` regados por todo `Conexion.py`. Eso no es logging, es "ruido con esteroides". Un sistema de logs bien diseñado es lo que te permite, a las 2am, entender por qué el forecast de Claro RD salió con demanda negativa sin tener que reproducir el bug en tu máquina.

## 1. ¿Por qué logging y no `print()`?

`print()` tiene tres problemas fatales en producción:

1. **No tiene niveles.** No puedes decirle "muéstrame solo los errores en producción, pero todo en desarrollo".
2. **No tiene destino configurable.** Un `print()` se va a la consola y ya. No puedes mandarlo a un archivo, a un colector centralizado (ELK, Datadog, Azure Monitor) y a la consola al mismo tiempo.
3. **No tiene contexto estructurado.** No sabes en qué módulo, función, línea o con qué variables ocurrió el evento, a menos que lo escribas a mano cada vez.

El módulo estándar `logging` de Python resuelve los tres. Está relacionado directamente con [[04-Ingenieria-de-Datos]] (todo pipeline batch necesita trazabilidad) y con [[18-Monitoreo-y-Observabilidad-de-Modelos]] (los logs son una de las tres patas de la observabilidad, junto a métricas y trazas).

## 2. Los niveles de logging y cuándo usarlos

| Nivel | Valor numérico | Cuándo usarlo en un proyecto ML |
|---|---|---|
| `DEBUG` | 10 | Forma de las matrices, shape de un DataFrame, hiperparámetros probados, valores intermedios de un feature |
| `INFO` | 20 | "Inicio de entrenamiento", "Modelo X registrado en MLflow", "Pipeline completado en 4.2s" |
| `WARNING` | 30 | Datos faltantes que se imputaron, drift leve detectado, fallback a un modelo anterior |
| `ERROR` | 40 | Falló la conexión a la base de datos, el modelo no pudo cargar, una fila corrupta rompió el feature engineering |
| `CRITICAL` | 50 | El pipeline completo no puede continuar, corrupción de datos que exige intervención humana inmediata |

Regla de oro que te va a ahorrar años de dolores de cabeza: **en producción el nivel por defecto es `INFO`**, nunca `DEBUG`. `DEBUG` se activa temporalmente para diagnosticar un incidente puntual, porque genera demasiado volumen y puede exponer datos sensibles.

## 3. Anatomía de un logger bien configurado

```python
import logging
import sys
from logging.handlers import RotatingFileHandler

def get_logger(name: str, log_file: str = "pipeline.log") -> logging.Logger:
    logger = logging.getLogger(name)
    logger.setLevel(logging.INFO)

    if logger.handlers:  # evita duplicar handlers si se llama dos veces
        return logger

    formatter = logging.Formatter(
        "%(asctime)s | %(levelname)-8s | %(name)s | %(message)s"
    )

    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setFormatter(formatter)

    file_handler = RotatingFileHandler(
        log_file, maxBytes=10_000_000, backupCount=5
    )
    file_handler.setFormatter(formatter)

    logger.addHandler(console_handler)
    logger.addHandler(file_handler)
    return logger
```

Puntos que un junior casi siempre pasa por alto:

- **`getLogger(__name__)` en cada módulo**, no un logger global compartido. Esto te permite saber exactamente de qué módulo vino cada línea (`forecasting`, `database`, `staffing`, etc., como en tu proyecto de Claro RD) y activar/desactivar niveles por módulo.
- **`RotatingFileHandler`** en vez de un archivo plano que crece infinitamente. En un pipeline que corre cada 30 minutos, un archivo sin rotación puede llenar el disco del servidor en semanas.
- **`logger.propagate = False`** cuando tienes handlers duplicados por herencia jerárquica de loggers (un error clásico que produce logs duplicados).

## 4. Logging estructurado (JSON) — el siguiente nivel

Cuando tu log va a un sistema centralizado (ELK, Loki, Azure Log Analytics), el texto plano es difícil de indexar y consultar. El logging estructurado emite cada línea como JSON:

```python
import logging
import json
from datetime import datetime, timezone

class JSONFormatter(logging.Formatter):
    def format(self, record):
        payload = {
            "timestamp": datetime.now(timezone.utc).isoformat(),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
            "module": record.module,
            "func": record.funcName,
        }
        if hasattr(record, "extra_fields"):
            payload.update(record.extra_fields)
        return json.dumps(payload)
```

Y lo invocas así, agregando contexto de negocio (algo clave en ML — quieres saber para qué `office_id` o qué `model_version` ocurrió el evento):

```python
logger.info(
    "Forecast generado",
    extra={"extra_fields": {"office_id": 145, "model_version": "1.4.2", "rows": 2880}}
)
```

Librerías como `structlog` o `python-json-logger` te dan esto sin reinventar la rueda; para proyectos serios de MLOps son la opción recomendada sobre construir tu propio `JSONFormatter`.

## 5. Logging específico de Machine Learning

Aquí es donde el logging deja de ser "genérico de software" y se vuelve "de ML". Cosas que un logger de un pipeline de ML debería capturar que un CRUD normal no necesita:

- **Metadatos de datos de entrada**: shape, rango de fechas, cantidad de nulos por columna, distribución básica (media, std) antes y después de imputación.
- **Metadatos de entrenamiento**: hiperparámetros usados, tiempo de entrenamiento, tamaño de train/test, semilla aleatoria (`random_state`) para reproducibilidad.
- **Metadatos de inferencia**: versión del modelo cargado, tiempo de inferencia, número de predicciones generadas, si hubo fallback a un modelo anterior.
- **Alertas de drift o de calidad de datos**: cuando un feature sale del rango esperado histórico (esto conecta directo con [[07-Librerias-de-Data-Science-y-ML#Evidently|Evidently]] y [[07-Librerias-de-Data-Science-y-ML#Great Expectations|Great Expectations]]).

Este tipo de logging es lo que después alimenta tus dashboards de [[18-Monitoreo-y-Observabilidad-de-Modelos]] — no pienses el logging como algo aislado del monitoreo, es su materia prima.

## 6. Anti-patrones que vas a ver (y que debes evitar)

- **Loggear datos sensibles**: PII, credenciales, tokens. Nunca. Ni siquiera en `DEBUG`.
- **Loggear dentro de loops sin control**: si tu pipeline procesa 500,000 filas, no hagas `logger.info()` por cada fila. Loguea agregados (inicio, fin, cada N filas, o solo anomalías).
- **Usar `except: pass` sin loguear el error**. Es el equivalente a esconder la basura debajo de la alfombra — el bug sigue ahí, solo que ahora es invisible hasta que explota en producción sin ningún rastro.
- **Mezclar `print()` y `logging` en el mismo módulo**. Sé consistente: si el proyecto usa `logging`, todo el proyecto usa `logging`.

## 7. Conexión con el resto del stack

El logging no vive solo — se conecta con:

- [[14-CICD-para-ML-con-GitLab]]: los logs de un pipeline fallido en CI son tu primera fuente de diagnóstico.
- [[19-Mantenimiento-y-Ciclo-de-Vida-del-Modelo]]: los logs históricos te dicen cuándo y por qué se disparó un reentrenamiento.
- [[15-MLflow-en-Profundidad]]: MLflow tiene su propio sistema de tracking de parámetros y métricas, que es complementario (no sustituto) al logging de aplicación.

## Ver también
- [[04-Ingenieria-de-Datos]]
- [[09-MLOps-en-Profundidad]]
- [[18-Monitoreo-y-Observabilidad-de-Modelos]]
- [[19-Mantenimiento-y-Ciclo-de-Vida-del-Modelo]]
