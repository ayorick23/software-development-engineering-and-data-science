---
tags: [documentacion, mkdocs, buenas-practicas]
---

# 33 — Documentación Profesional: Google Docstrings, MkDocs, README

> Nota del mentor: el código más elegante del mundo no sirve de nada si el siguiente ingeniero que lo hereda (como tú heredaste el proyecto de Adrián) no puede entender rápidamente qué hace y por qué. La documentación no es un extra opcional al final del proyecto — es parte de la entrega, tanto como los tests.

## 1. Google Style Docstrings — el estándar más usado en proyectos de ML/Python

```python
def calcular_erlang_c(
    demanda: float,
    tiempo_servicio: float,
    nivel_servicio_objetivo: float = 0.8,
) -> tuple[int, float]:
    """Calcula el número mínimo de agentes requeridos usando la fórmula Erlang C.

    Determina cuántos agentes se necesitan para atender una demanda dada
    manteniendo un nivel de servicio objetivo, iterando sobre el número
    de agentes hasta encontrar el mínimo que cumple el umbral.

    Args:
        demanda: Número esperado de llamadas en el intervalo (llamadas/hora).
        tiempo_servicio: Tiempo promedio de atención por llamada, en segundos.
        nivel_servicio_objetivo: Proporción mínima de llamadas atendidas
            dentro del umbral de espera aceptable. Por defecto 0.8 (80%).

    Returns:
        Una tupla (agentes_requeridos, nivel_servicio_estimado) donde:
            - agentes_requeridos: número mínimo de agentes que cumple el objetivo.
            - nivel_servicio_estimado: nivel de servicio real alcanzado con ese número.

    Raises:
        ValueError: Si demanda o tiempo_servicio son negativos.

    Example:
        >>> agentes, nivel_servicio = calcular_erlang_c(demanda=45.0, tiempo_servicio=180.0)
        >>> agentes
        12
    """
```

Las secciones `Args`, `Returns`, `Raises` y `Example` son las que herramientas como MkDocs (con la extensión `mkdocstrings`) leen automáticamente para generar documentación navegable — escribir el docstring bien **una vez** te da tanto ayuda en el editor (hover de VS Code/PyCharm) como sitio de documentación generado, sin trabajo adicional.

### Qué documentar y qué no

- **Documenta el "por qué" de decisiones no obvias**, no solo el "qué": "usamos `copy.deepcopy()` en vez de reasignar porque `warm_start=True` no tiene efecto sobre un objeto recién instanciado" vale mucho más que "esta función copia el modelo".
- **No documentes lo obvio**: `def obtener_nombre(self) -> str: """Retorna el nombre."""` es ruido, no documentación.
- **Actualiza el docstring cuando cambies el comportamiento**. Un docstring desactualizado es peor que ninguno — miente activamente a quien confía en él.

## 2. MkDocs — documentación navegable generada desde el código

```yaml
# mkdocs.yml
site_name: Claro RD Forecasting Pipeline
theme:
  name: material
nav:
  - Inicio: index.md
  - Arquitectura: arquitectura.md
  - Módulos:
      - forecasting: modulos/forecasting.md
      - feature_engineering: modulos/feature_engineering.md
plugins:
  - mkdocstrings:
      handlers:
        python:
          options:
            docstring_style: google
```

```markdown
<!-- docs/modulos/forecasting.md -->
# Módulo forecasting

::: forecasting_pipeline.forecasting
```

Esa última línea (`::: forecasting_pipeline.forecasting`) es la magia de `mkdocstrings`: lee automáticamente todos los docstrings del módulo y genera una página HTML navegable, con firma de funciones, tipos, y ejemplos — sin que tengas que escribir la documentación dos veces (una en el código, otra en un Word aparte que se desactualiza en la primera semana).

```bash
pip install mkdocs mkdocs-material mkdocstrings[python]
mkdocs serve   # previsualización local en http://localhost:8000
mkdocs build   # genera el sitio estático para publicar
```

Para un equipo como el tuyo en ACF Technologies, publicar esto como GitLab Pages (integrado directamente con tu `.gitlab-ci.yml` de [[14-CICD-para-ML-con-GitLab]]) da a cualquier compañero nuevo un punto de entrada real para entender el proyecto, en vez de tener que leer siete módulos de código desde cero.

## 3. README profesional — la primera impresión de tu repositorio

Un buen README de un proyecto de ML en producción responde, en este orden, a las preguntas que cualquiera se hace en los primeros 30 segundos:

```markdown
# Claro RD — Forecasting Pipeline

Pipeline de forecasting de demanda de llamadas y tiempo de servicio
para el cliente Claro RD, con cálculo de dotación de agentes vía Erlang C
y reentrenamiento automático (champion/challenger).

## ¿Qué hace?
[2-3 líneas de contexto de negocio — qué problema resuelve, para quién]

## Arquitectura
[Diagrama simple o referencia a docs/arquitectura.md]

## Requisitos
- Python 3.11+
- SQL Server con acceso a la base de datos qf.*
- Ver `pyproject.toml` para dependencias completas

## Instalación
\`\`\`bash
uv sync
cp .env.example .env  # completar variables de entorno
\`\`\`

## Uso
\`\`\`bash
uv run python -m forecasting_pipeline.main
\`\`\`

## Tests
\`\`\`bash
pytest tests/ --cov=src
\`\`\`

## Estructura del proyecto
[Árbol de directorios breve]

## Decisiones de negocio confirmadas
- ModifiedDemand sobrescribe TotalDemandPredict de forma intencional (no es un bug)
- fast_executemany deshabilitado por error 22003 de overflow numérico
[etc. — decisiones que de otra forma solo viven en la memoria de una persona]
```

Esa última sección — **decisiones de negocio confirmadas** — es la que más valor real le habría dado a quien heredó tu proyecto (tú) si hubiera existido desde el principio. Documentar explícitamente las decisiones no obvias evita que un ingeniero nuevo "corrija" por error algo que era intencional, exactamente el tipo de malentendido que tuviste que aclarar con tu jefa sobre `ModifiedDemand`.

## 4. Documentación como parte del flujo, no como tarea aparte

La disciplina real de un equipo maduro: **ningún Merge Request se aprueba si cambia comportamiento público sin actualizar el docstring o el README correspondiente** — de la misma forma que no se aprueba sin tests, como vimos en [[14-CICD-para-ML-con-GitLab]]. La documentación que vive separada del código (un Word compartido, un Confluence desconectado del repo) tiene una vida útil de semanas antes de desactualizarse; la documentación que vive **dentro** del repositorio, generada desde el código mismo, tiene muchas más probabilidades de mantenerse viva.

## Ver también
- [[14-CICD-para-ML-con-GitLab]]
- [[12-Gestion-Moderna-de-Proyectos-Python]]
- [[30-Principios-SOLID-y-Clean-Code-para-ML]]
