---
tags: [mantenimiento, reentrenamiento, ciclo-de-vida, mlops]
---

# 19 — Mantenimiento y Ciclo de Vida del Modelo

> Nota del mentor: entrenar un modelo bueno es quizás el 20% del trabajo real en un proyecto de ML en producción. El 80% restante es mantenerlo vivo, saludable y confiable durante meses o años — exactamente lo que estás viviendo ahora que heredaste el proyecto de Adrián: el modelo ya existía, tu trabajo real es mantenerlo, entenderlo y mejorarlo con seguridad.

## 1. El ciclo de vida completo de un modelo en producción

```
Entrenamiento inicial
        ↓
Validación y registro (MLflow Registry)
        ↓
Despliegue a producción
        ↓
Monitoreo continuo (drift, métricas) ←────────────┐
        ↓                                         │
¿Degradación detectada?                           │
        ↓ sí                                      │
Reentrenamiento (champion/challenger)             │
        ↓                                         │
¿Challenger supera el gate?                       │
   ↓ sí          ↓ no                             │
Promover      Descartar y volver a monitorear ────┘
        ↓
Archivar versión anterior (nunca se borra — trazabilidad)
```

Este ciclo nunca "termina" mientras el modelo esté en producción — es la razón por la que MLOps existe como disciplina separada de "hacer un modelo en un notebook".

---

## 2. Cadencia de reentrenamiento — ¿cuándo es suficiente?

Tres estrategias, cada una con su trade-off:

- **Cadencia fija (tiempo)**: reentrenar cada N días, como haces con `dias_entre_reentrenamientos` en tu sistema actual. Simple, predecible, pero puede reentrenar innecesariamente (desperdicio de cómputo) o insuficientemente (si el drift ocurre más rápido de lo esperado).
- **Cadencia disparada por drift**: reentrenar solo cuando el monitoreo de [[18-Monitoreo-y-Observabilidad-de-Modelos]] detecta un PSI o degradación de métrica por encima de un umbral. Más eficiente, pero requiere que tu sistema de monitoreo sea confiable — si el monitoreo falla en detectar drift, el modelo se queda estancado sin que nadie se dé cuenta.
- **Cadencia híbrida**: la más usada en la práctica — una cadencia mínima fija como red de seguridad ("al menos cada 90 días revisamos"), combinada con reentrenamiento anticipado si el monitoreo detecta drift antes de esa fecha.

---

## 3. El gate de calidad — nunca reemplazar a ciegas

El mecanismo champion/challenger que ya implementaste (comparar MAE/RMSE del modelo nuevo contra el actual antes de reemplazarlo) es, en la práctica de la industria, exactamente el patrón correcto — no una solución "casera" inferior a lo que hacen equipos maduros. Los elementos que un gate robusto debe tener:

- **Métrica primaria y secundaria**: tú ya usas MAE como primaria y RMSE como gate de tolerancia secundario — esto evita que un modelo "gane" en promedio pero empeore drásticamente en casos extremos.
- **Conjunto de validación fijo y representativo**: nunca comparar contra datos que el challenger ya vio de alguna forma (fuga de datos entre entrenamiento y validación).
- **Márgenes de mejora mínima, no solo "mejor o igual"**: exigir una mejora mínima (`margen_mejora_minima`) evita reemplazar un modelo por ruido estadístico — una mejora de 0.01% en MAE no justifica el riesgo operativo de un cambio de modelo.
- **Registro de todo intento**, gane o pierda el gate, para auditoría futura (esto es exactamente lo que MLflow Tracking te da automáticamente, ver [[15-MLflow-en-Profundidad]]).

---

## 4. Rollback — el plan B que siempre debe existir

Ningún sistema de reentrenamiento automático está exento de fallar de formas inesperadas. Preguntas que todo pipeline de producción debe poder responder en minutos, no en horas:

- ¿Cómo vuelvo a la versión anterior del modelo si la nueva empieza a fallar en producción?
- ¿Tengo acceso inmediato a los artefactos de la versión anterior (no solo el código, el modelo entrenado exacto)?
- ¿El rollback requiere un despliegue completo, o es un cambio de configuración rápido (ej. apuntar a otra versión en el Model Registry)?

Con MLflow Model Registry, un rollback es literalmente cambiar qué versión tiene el alias `production` — segundos, no hay que reentrenar ni redeployar código.

---

## 5. Deuda técnica específica de Machine Learning

El paper clásico *"Hidden Technical Debt in Machine Learning Systems"* (Google, Sculley et al.) identifica deudas que no existen en software tradicional y que vale la pena que conozcas de nombre:

- **Entanglement (CACE — Changing Anything Changes Everything)**: en un modelo de ML, cambiar un solo feature puede alterar el peso e importancia de todos los demás, a diferencia del software tradicional donde los módulos suelen ser más aislables.
- **Dependencias de datos no versionadas**: si tu fuente de datos cambia silenciosamente (un cliente empieza a registrar `AvgServiceTime` distinto, como descubriste con el nombre real de la columna), el modelo se degrada sin que el código haya cambiado en absoluto.
- **Pipeline jungles**: pipelines de feature engineering que crecen orgánicamente con parches sobre parches — exactamente el problema que resolviste al unificar la lógica duplicada de `feature_engineering.py` entre tus dos notebooks de entrenamiento.
- **Deuda de configuración**: decenas de parámetros (`margen_mejora_minima`, `dias_validacion`, etc.) sin documentar el porqué de cada valor — la solución es documentar la razón de negocio detrás de cada uno, no solo el valor.

---

## 6. El mantenimiento como responsabilidad de largo plazo

Con 20 años en esto, la lección más importante que te puedo dar sobre esta nota: un modelo que nadie mantiene activamente **siempre** se degrada, tarde o temprano, sin excepción — el mundo real cambia constantemente, aunque tu código no lo haga. La pregunta profesional correcta nunca es "¿el modelo funciona hoy?", sino "¿tenemos los mecanismos (monitoreo, gates, rollback, documentación) para saber cuándo deja de funcionar, y para corregirlo sin pánico cuando eso pase?".

---

## Ver también

- [[15-MLflow-en-Profundidad]]
- [[18-Monitoreo-y-Observabilidad-de-Modelos]]
- [[14-CICD-para-ML-con-GitLab]]
- [[09-MLOps-en-Profundidad]]
