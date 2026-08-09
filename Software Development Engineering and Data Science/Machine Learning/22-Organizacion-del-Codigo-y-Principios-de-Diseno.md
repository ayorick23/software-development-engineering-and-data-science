---
tags: [arquitectura, diseno-de-software, buenas-practicas]
---

# 22 — Organización del Código: Separación de Responsabilidades, Modularidad, Acoplamiento y Cohesión

> Nota del mentor: cuando abriste `Conexion.py` por primera vez, seguramente sentiste ese peso de "todo está conectado con todo". Eso tiene nombre técnico: **alto acoplamiento** y **baja cohesión**. Entender estos dos conceptos de memoria es lo que te permite, la próxima vez que heredes un proyecto así, saber exactamente por dónde cortar.

## 1. Cohesión — ¿qué tanto pertenece junto lo que está junto?

Un módulo tiene **alta cohesión** cuando todo lo que contiene está relacionado con un mismo propósito. Tu refactor de `Conexion.py` en `forecasting.py`, `database.py`, `staffing.py`, `feature_engineering.py` es exactamente un ejercicio de aumentar cohesión: antes, un mismo archivo mezclaba conexión a base de datos, cálculo de Erlang C, y entrenamiento de modelos — tres responsabilidades distintas sin relación funcional directa entre sí.

**Señal de baja cohesión**: cuando describes qué hace un módulo y necesitas usar la palabra "y" repetidamente sin conexión lógica ("este archivo conecta a la base de datos **y** calcula Erlang C **y** también tiene la lógica de reintentos de red **y** valida el schema de entrada").

## 2. Acoplamiento — ¿qué tanto depende un módulo de los detalles internos de otro?

**Bajo acoplamiento** significa que puedes cambiar la implementación interna de un módulo sin romper a los que lo usan, siempre que mantengas su interfaz (contrato) estable.

```python
# Alto acoplamiento: forecasting.py conoce detalles internos de database.py
def entrenar_modelo():
    conn = pyodbc.connect("DRIVER={SQL Server};SERVER=...;DATABASE=...")
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM qf.hrCallData")
    ...

# Bajo acoplamiento: forecasting.py solo conoce la interfaz de database.py
from .database import obtener_datos_historicos

def entrenar_modelo():
    datos = obtener_datos_historicos()
    ...
```

En la segunda versión, si mañana migras de `pyodbc` a `sqlalchemy`, o cambias el connection string, **solo tocas `database.py`** — `forecasting.py` no se entera del cambio porque nunca conoció esos detalles.

## 3. La relación entre cohesión, acoplamiento y tus siete módulos

La meta de un buen diseño es siempre: **alta cohesión dentro de cada módulo, bajo acoplamiento entre módulos**. Es el mismo principio detrás de `config.py`, `database.py`, `forecasting.py`, `feature_engineering.py`, `staffing.py`, `adjustments.py`, `autoaprendizaje.py`:

```
config.py            → solo configuración, nadie más decide parámetros
database.py          → único responsable de hablar con SQL Server
feature_engineering.py → única fuente de verdad de transformaciones de features
forecasting.py        → orquesta entrenamiento/predicción, delega en los anteriores
staffing.py           → cálculo de Erlang C, no sabe nada de SQL
autoaprendizaje.py    → orquesta el ciclo champion/challenger
```

Cada módulo tiene una sola razón para cambiar — esto es literalmente el **Single Responsibility Principle** de SOLID (ver [[30-Principios-SOLID-y-Clean-Code-para-ML]]), pero vale la pena entenderlo aquí como principio de organización antes de formalizarlo como "letra de SOLID".

## 4. Cómo dividir un proyecto grande — el criterio práctico

No hay una fórmula matemática, pero estas preguntas te guían al dividir:

- **¿Este código cambia por razones distintas a ese otro código?** Si sí, sepáralos. La lógica de conexión a base de datos cambia por razones de infraestructura; la lógica de Erlang C cambia por razones de negocio (fórmulas de dotación). Son razones de cambio distintas → módulos distintos.
- **¿Puedo describir este módulo en una frase sin usar "y"?** Si necesitas "y", probablemente hace más de una cosa.
- **¿Este código se reutiliza en más de un lugar?** Si sí, sácalo a su propio módulo (como hiciste al unificar `feature_engineering.py` entre tus dos notebooks, eliminando la duplicación cuádruple).
- **¿Cuánto tarda un compañero nuevo en encontrar dónde vive una funcionalidad?** Si la respuesta es "tiene que preguntar", la organización necesita mejorar, sin importar qué tan "limpio" esté el código dentro de cada archivo.

## 5. Capas típicas de un proyecto de ML bien organizado

```
src/forecasting_pipeline/
├── config.py              # Configuración (capa de configuración)
├── database.py             # Acceso a datos (capa de infraestructura)
├── feature_engineering.py  # Transformación de datos (capa de dominio)
├── forecasting.py           # Modelos y predicción (capa de dominio)
├── staffing.py              # Reglas de negocio (Erlang C) (capa de dominio)
├── autoaprendizaje.py       # Orquestación del ciclo de reentrenamiento (capa de aplicación)
└── main.py                  # Punto de entrada, conecta todas las capas (capa de aplicación)
```

Esta separación entre **infraestructura** (cómo hablamos con el mundo exterior: base de datos, APIs), **dominio** (la lógica de negocio y ML en sí) y **aplicación** (orquestación de todo lo anterior) es una versión simplificada y aplicada de Clean Architecture — profundizamos en eso formalmente en [[32-Diseno-Composicion-Herencia-Interfaces]].

## 6. El costo real de ignorar esto

Un proyecto con baja cohesión y alto acoplamiento no "se rompe" de inmediato — se rompe lentamente, en forma de: cada cambio pequeño tarda más de lo esperado, los bugs aparecen en lugares que nadie tocó directamente, y el miedo a tocar código legado ("mejor no lo toco, no sé qué más depende de esto") paraliza al equipo. Exactamente el estado en el que probablemente encontraste `Conexion.py` — y exactamente lo que resolviste al modularizarlo.

## Ver también
- [[21-Python-Avanzado-Ejecucion-y-Metaprogramacion]]
- [[30-Principios-SOLID-y-Clean-Code-para-ML]]
- [[32-Diseno-Composicion-Herencia-Interfaces]]
