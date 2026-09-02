---
tags: [mlflow, mlops, buenas-practicas, seguridad, cheat-sheet]
---

# 15 — Buenas Prácticas, Seguridad y Comparativa

> Cierre del cheat-sheet. Se apoya en todos los archivos anteriores, especialmente [[03 - Tracking - Servidor, Backend Store y Artifact Store]], [[04 - Tracking - Búsqueda, Comparación y Organización]] y [[07 - Model Registry]].

## Organización de experiments a escala

Cuando un equipo crece más allá de una persona, la falta de convenciones convierte el Tracking en un desorden inconsultable. Reglas que escalan bien:

- **Un experiment por caso de uso de negocio**, no por persona ni por fecha. `"claro-rd-demand-forecast"` sí; `"pruebas_dereck_agosto"` no.
- **Nombres de run descriptivos del enfoque probado**, no genéricos (`"run_1"`, `"test"`). Prefiere `"xgboost-90d-window-v2"`.
- **Tags consistentes** entre todo el equipo para las mismas dimensiones (`team`, `data_version`, `git_commit`) — si cada quien inventa su propio nombre de tag, la búsqueda con `filter_string` deja de servir.
- **Un Registered Model por modelo de producción real**, no uno por cada prueba — el Registry es para modelos candidatos a producción, no para historial de experimentación exploratoria (eso ya lo cubre Tracking).
- **Limpieza periódica**: usar `mlflow gc` (ver [[13 - CLI Reference]]) para purgar runs de prueba marcados como soft-deleted; si no, el backend store y el artifact store crecen indefinidamente con ruido.

## Seguridad del servidor de Tracking

- **Nunca exponer el servidor de Tracking directamente a internet sin autenticación.** El Tracking Server, por defecto, no tiene auth activado — cualquiera con la URL puede leer y escribir experimentos.
- Usar `--serve-artifacts` (ver [[03 - Tracking - Servidor, Backend Store y Artifact Store]]) para que las credenciales de la nube (S3/Azure Blob) vivan solo en el servidor, nunca en cada cliente.
- Para autenticación real de equipo, preferir un **reverse proxy con SSO/OAuth corporativo** delante del servidor MLflow, en vez de depender únicamente del módulo `mlflow[auth]` nativo (pensado para casos simples, no para integración con IdP corporativo).
- Las credenciales de base de datos del backend store (Postgres) no deberían estar en texto plano en scripts — usar variables de entorno o un secret manager (Azure Key Vault, AWS Secrets Manager).
- Restringir permisos de escritura en el Model Registry: no todos los que pueden loguear experimentos deberían poder mover un modelo a `Production`/asignar el alias `champion` — esto normalmente se resuelve con el gate automatizado de CI/CD (ver [[14 - Integraciones con el Ecosistema]]), no con permisos manuales dispersos.

## Trazabilidad completa — el objetivo final

El valor real de adoptar MLflow con disciplina es poder responder, para cualquier modelo en producción, sin reconstruir nada manualmente:

1. ¿Con qué código se entrenó? → `mlflow.source.git.commit` (tag automático)
2. ¿Con qué datos? → `mlflow.log_input` o tag de versión de DVC/feature store
3. ¿Con qué hiperparámetros? → `params` del run
4. ¿Con qué resultado en validación? → `metrics` del run
5. ¿Quién lo promovió y cuándo? → historial de versiones/aliases del Registry
6. ¿Cómo hacer rollback si falla? → reasignar el alias `champion` a la versión anterior, una sola llamada

## MLflow vs. otras herramientas del mismo espacio

| Herramienta | Fortaleza relativa | Cuándo preferirla sobre MLflow |
|---|---|---|
| **Weights & Biases (W&B)** | UI más pulida, mejor para deep learning con muchísimas métricas por época, colaboración en equipo con comentarios | Equipos de investigación con foco en deep learning, presupuesto para SaaS |
| **Neptune.ai** | Fuerte en organización de metadata compleja y comparación de runs a gran escala | Equipos con cientos de experimentos simultáneos que necesitan tablas altamente personalizables |
| **Kubeflow (Katib/Pipelines)** | Nativo de Kubernetes, mejor para orquestación de pipelines completos en K8s | Organizaciones ya 100% en Kubernetes con necesidad de orquestación, no solo tracking |
| **DVC** | Versionado de datos y pipelines como código (Git-like), sin servidor central | Cuando el foco es reproducibilidad de *datos y pipeline*, no tanto UI de comparación de experimentos |
| **SageMaker Experiments** | Integración nativa profunda con el resto de AWS SageMaker | Si el stack completo ya vive en SageMaker (training jobs, endpoints, pipelines) |

**Por qué MLflow suele ganar como punto de partida**: es open-source, agnóstico de librería y de nube, no obliga a un vendor lock-in, y cubre las cuatro necesidades (tracking, empaquetado, formato de modelo, registro) en una sola herramienta sin costo de licencia. La desventaja relativa frente a W&B/Neptune es una UI algo menos pulida para deep learning intensivo en métricas por step.

## Errores comunes de juniors (y cómo evitarlos)

- **Confundir MLflow con un orquestador**: MLflow registra qué pasó, no decide cuándo correr algo. Ver [[01 - Introducción y Arquitectura General]].
- **Loguear el modelo pero no el signature/input_example**: sin eso, `mlflow models serve` no puede validar requests entrantes y los errores de tipo de dato aparecen recién en producción. Ver [[06 - Model Format y Flavors]].
- **Usar `Stages` en proyectos nuevos**: está deprecado; usar `aliases` desde el día uno evita una migración futura. Ver [[07 - Model Registry]].
- **No versionar el entorno junto al modelo**: confiar en que "el entorno de mi máquina ya tiene lo necesario" en vez de dejar que MLflow capture `requirements.txt`/`conda.yaml` automáticamente — rompe reproducibilidad al mover el modelo a otro entorno.
- **Registrar cada trial de una búsqueda de hiperparámetros en el Model Registry**: satura el Registry con ruido; solo el modelo ganador final debería registrarse (ver [[10 - Hyperparameter Tuning con MLflow]]).
- **Exponer `mlflow models serve` directamente a tráfico productivo de alto volumen** sin un balanceador ni autoscaling delante. Ver [[09 - Model Serving y Despliegue]].

## Ver también

- [[01 - Introducción y Arquitectura General]]
- [[07 - Model Registry]]
- [[14 - Integraciones con el Ecosistema]]
- [[15-MLflow-en-Profundidad]] (en `Machine Learning/`)
