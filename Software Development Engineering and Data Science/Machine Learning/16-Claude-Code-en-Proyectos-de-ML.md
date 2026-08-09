---
tags: [claude-code, herramientas, productividad, ia-asistida]
---

# 16 — Claude Code en Proyectos de Machine Learning

> Nota del mentor: en 20 años he visto pasar generaciones de herramientas — de IDEs sin autocompletado a Copilot, y ahora a agentes que pueden ejecutar comandos, leer un repo completo y editar múltiples archivos coordinadamente. Claude Code entra en esta última categoría. Úsalo bien y multiplicas tu capacidad; úsalo mal y terminas con código que no entiendes y no puedes defender frente a tu jefa.

## 1. Qué es realmente Claude Code (y en qué se diferencia de un chat normal)

Claude Code es un **agente de código de terminal**: no solo conversa, sino que puede leer archivos de tu repositorio, ejecutar comandos (`pytest`, `git`, `python script.py`), editar múltiples archivos y verificar el resultado ejecutando código real, todo dentro de un ciclo iterativo. La diferencia clave frente a copiar y pegar código de un chat: **tiene el contexto completo de tu proyecto** y puede validar sus propios cambios corriendo tus tests.

## 2. Casos de uso de alto valor en un proyecto de ML como el tuyo

- **Refactorización guiada de código heredado**: exactamente lo que hiciste al modernizar `Conexion.py` en siete módulos. Claude Code puede leer el archivo monolítico completo, proponer la separación en módulos, y ejecutar los tests después de cada cambio para verificar que no rompió nada.
- **Escritura de tests para código sin cobertura**: dale un módulo como `feature_engineering.py` y pídele que genere tests de `pytest` cubriendo casos borde — luego revisa tú si los casos borde propuestos son los correctos para tu dominio de negocio.
- **Debugging con contexto de logs**: pégale (o dale acceso a) el archivo de logs de una corrida fallida del pipeline, y pídele que correlacione el error con el código fuente — es mucho más rápido que hacer `grep` manual en un log de miles de líneas.
- **Generación de `.gitlab-ci.yml` y `pyproject.toml` iniciales**: para bootstrapping de configuración, dado que siguen convenciones bien documentadas que el modelo conoce bien.
- **Documentación de código legado sin comentarios**: pídele que lea un módulo y genere docstrings explicando qué hace cada función, como primer borrador para que tú lo valides.

## 3. Dónde debes tener más cuidado (y por qué)

- **Lógica de negocio específica del cliente.** Claude Code no sabe, sin que tú se lo digas, que `ModifiedDemand` sobrescribiendo `TotalDemandPredict` es intencional y no un bug — ese conocimiento vive en las decisiones de negocio que confirmó tu jefa, no en el código. Siempre dale ese contexto explícito antes de pedirle que "arregle" algo que parece raro.
- **Cambios en pipelines de producción sin supervisión.** Nunca dejes que un agente aplique cambios directo a un pipeline que corre en producción sin pasar por Merge Request y CI, exactamente igual que exigirías de cualquier desarrollador humano.
- **Números y métricas que van a un reporte para tu jefa.** Si le pides que calcule o resuma métricas de un experimento, siempre verifica tú los números contra la fuente — trátalo como a un colega junior muy rápido, no como una fuente de verdad infalible.
- **Credenciales y datos sensibles.** No le des connection strings reales de la base de datos de Claro RD ni datos de producción con información identificable si no es estrictamente necesario; usa variables de entorno y datos de muestra/anonimizados cuando sea posible.

## 4. Buenas prácticas de uso — cómo un senior lo integra al flujo diario

1. **Dale contexto explícito, no solo el código.** "Esto es un pipeline de forecasting de demanda para un call center, `office_id` identifica una sede, `TotalDemand` es la variable objetivo" vale más que mil líneas de código sin explicación.
2. **Pide explicación antes de implementación** en cambios no triviales — es exactamente tu forma de trabajar preferida, y es también la forma correcta de trabajar con cualquier herramienta de IA: entender el "por qué" antes de aceptar el "qué".
3. **Usa `CLAUDE.md` (o equivalente) en la raíz del repo** para documentar convenciones del proyecto (estilo de logging, estructura de módulos, decisiones de negocio confirmadas) — así el agente parte de ese contexto en cada sesión nueva sin que tengas que repetirlo.
4. **Revisa cada diff como revisarías un Merge Request de un compañero.** La velocidad de generación no debe traducirse en menor rigor de revisión — al contrario, mientras más rápido se genera el código, más disciplina de revisión necesitas.
5. **Pide que corra los tests después de cada cambio**, no solo que "escriba" el código — la validación en el loop es lo que distingue a un agente de código de un autocompletado glorificado.

## 5. Integración con el resto de tu stack

Claude Code se apoya bien sobre la infraestructura que ya estás construyendo:

- Con [[13-Testing-en-Machine-Learning]]: entre más sólida tu suite de tests, más seguro es dejar que un agente refactorice, porque los tests son la red de seguridad que detecta regresiones automáticamente.
- Con [[11-Logging-en-Python-para-ML]]: un buen logging estructurado le da al agente (y a ti) contexto de calidad para debuggear incidentes.
- Con [[14-CICD-para-ML-con-GitLab]]: cualquier cambio generado, humano o asistido, pasa por el mismo pipeline de CI antes de llegar a producción — sin excepciones.

## 6. La regla de oro

Un agente de código es un multiplicador de tu criterio, no un sustituto de él. Con 20 años en esto te puedo decir que la habilidad que más va a diferenciar a un ingeniero de ML competente de uno mediocre en los próximos años **no** es qué tan rápido genera código con IA — es qué tan bien sabe **qué pedir, qué revisar y qué rechazar**.

## Ver también

- [[13-Testing-en-Machine-Learning]]
- [[11-Logging-en-Python-para-ML]]
- [[14-CICD-para-ML-con-GitLab]]
