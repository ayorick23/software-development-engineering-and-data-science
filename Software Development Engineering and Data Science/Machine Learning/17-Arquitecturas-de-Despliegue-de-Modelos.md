---
tags: [deployment, arquitectura, mlops, produccion]
---

# 17 — Arquitecturas de Despliegue de Modelos de ML

> Nota del mentor: "desplegar un modelo" no significa una sola cosa. Significa cosas radicalmente distintas según la latencia que el negocio necesita, el volumen de datos, y si hay un humano esperando la respuesta o no. Tu pipeline de Claro RD es batch; un sistema de recomendaciones de un e-commerce es online. Confundir estos patrones es uno de los errores de arquitectura más caros que un equipo puede cometer.

## 1. Los tres grandes patrones de inferencia

### Batch (por lotes)

El modelo procesa un volumen grande de datos de una sola vez, con una cadencia programada (cada 30 minutos, diario, semanal). No hay un usuario esperando una respuesta inmediata.

- **Ejemplo real**: exactamente tu pipeline de Claro RD — corre cada 30 minutos, procesa todas las oficinas, escribe resultados en `qf.hrAgentForecastResult`.
- **Ventajas**: más simple de construir y operar, puede usar recursos de cómputo más baratos (no necesita estar "siempre encendido"), fácil de reintentar si falla.
- **Desventajas**: latencia inherente — la predicción más reciente tiene, en el peor caso, la antigüedad de todo el intervalo entre corridas.

### Online / Real-time (síncrono)

El modelo responde a una solicitud individual en milisegundos, típicamente detrás de una API REST.

- **Ejemplo real**: un sistema de scoring de fraude en el momento de una transacción con tarjeta, un motor de recomendaciones al cargar una página.
- **Ventajas**: latencia mínima, respuesta personalizada al momento exacto de la solicitud.
- **Desventajas**: exige infraestructura siempre disponible (alta disponibilidad), más compleja de escalar, cada milisegundo de latencia del modelo cuenta.

### Streaming (asíncrono, casi tiempo real)

El modelo procesa eventos a medida que llegan por un bus de eventos (Kafka, Event Hubs), sin esperar una solicitud puntual ni acumular un batch completo.

- **Ejemplo real**: detección de anomalías sobre logs de sensores IoT en tiempo real, actualización continua de un feature de "actividad reciente del usuario".
- **Ventajas**: balance entre latencia y throughput, natural para datos que llegan de forma continua.
- **Desventajas**: la pieza más compleja de operar de las tres — requiere infraestructura de streaming (visto en [[04-Ingenieria-de-Datos]]) y manejo cuidadoso de estado y orden de eventos.

---

## 2. Cómo elegir el patrón correcto — la pregunta que debes hacerte primero

No es "¿qué tecnología es más moderna?", es: **¿cuánto tiempo puede esperar el negocio entre que ocurre un evento y que el modelo responde?**

- Si la respuesta es "hasta la próxima hora está bien" → batch.
- Si la respuesta es "milisegundos, hay un usuario esperando" → online.
- Si la respuesta es "segundos, y los eventos llegan continuamente sin una solicitud puntual" → streaming.

Tu proyecto de Claro RD es batch porque la dotación de agentes se planifica con intervalos de 30 minutos — no tendría sentido de negocio (ni de costo) hacerlo online.

---

## 3. Patrones de despliegue online en detalle

### Modelo embebido en la aplicación
El modelo se carga directamente dentro del proceso de la aplicación (ej. un `.pkl` cargado en un servicio Flask/FastAPI).

- Simple, baja latencia (no hay llamada de red extra), pero acopla el ciclo de vida del modelo al de la aplicación — actualizar el modelo requiere redesplegar la app entera.

### Modelo como microservicio independiente (Model-as-a-Service)
El modelo vive detrás de su propia API REST/gRPC, separada de la aplicación que lo consume.

```
App de negocio  →  HTTP/gRPC  →  Servicio de Modelo (FastAPI + modelo cargado)
                                          ↓
                                   MLflow Model Registry
```

- Desacopla el ciclo de vida del modelo del de la aplicación — puedes actualizar el modelo sin tocar la app. Es el patrón recomendado por defecto para la mayoría de casos online serios.

### Serverless (Functions-as-a-Service)
El modelo corre en una función que se activa por invocación (AWS Lambda, Azure Functions), sin servidor permanentemente encendido.

- Excelente para tráfico esporádico o impredecible (pagas solo por invocación), pero sufre de **cold start** (latencia extra en la primera invocación tras inactividad) — mal ajuste para modelos grandes de Deep Learning con esta restricción.

---

## 4. El rol de contenedores en el despliegue

Independientemente del patrón, casi todo despliegue moderno de un modelo pasa por un contenedor Docker:

```dockerfile
FROM python:3.11-slim
COPY pyproject.toml .
RUN pip install .
COPY src/ src/
CMD ["uvicorn", "forecasting_pipeline.api:app", "--host", "0.0.0.0", "--port", "8000"]
```

Esto conecta directo con [[09-MLOps-en-Profundidad]] (Docker, Kubernetes) y con [[14-CICD-para-ML-con-GitLab]] — el pipeline de CI/CD normalmente termina construyendo esta imagen y publicándola en un registro de contenedores antes del despliegue.

## 5. Estrategias de despliegue seguro (progressive delivery)

Desplegar un modelo nuevo directo al 100% del tráfico es arriesgado. Patrones que reducen ese riesgo:

- **Shadow deployment**: el modelo nuevo recibe el mismo tráfico que el actual, pero sus predicciones **no** se usan para decisiones reales — solo se registran para comparar contra el modelo en producción antes de confiar en él.
- **Canary release**: el modelo nuevo recibe un pequeño porcentaje del tráfico real (ej. 5%), y se aumenta gradualmente si las métricas se mantienen sanas.
- **A/B testing**: dos modelos corren simultáneamente sobre segmentos distintos de usuarios/oficinas, comparando el impacto en una métrica de negocio real, no solo en métricas técnicas offline.
- **Blue-Green deployment**: dos entornos idénticos (uno activo, uno en espera con la nueva versión); el cambio de tráfico es instantáneo y el rollback también, revirtiendo el enrutador al entorno anterior.

En tu caso de Claro RD, el equivalente de bajo riesgo que ya practicas — comparar challenger contra champion en una ventana de validación antes de reemplazar — es conceptualmente un *shadow deployment* aplicado a un contexto batch.

## Ver también

- [[09-MLOps-en-Profundidad]]
- [[04-Ingenieria-de-Datos]]
- [[18-Monitoreo-y-Observabilidad-de-Modelos]]
- [[19-Mantenimiento-y-Ciclo-de-Vida-del-Modelo]]
