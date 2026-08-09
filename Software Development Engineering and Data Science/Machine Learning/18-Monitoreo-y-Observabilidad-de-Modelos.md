---
tags: [monitoreo, observabilidad, drift, mlops]
---

# 18 — Monitoreo y Observabilidad de Modelos en Producción

> Nota del mentor: la diferencia entre monitoreo y observabilidad es sutil pero importante. **Monitoreo** te dice *que algo está mal* (una alerta). **Observabilidad** te da las herramientas para entender *por qué* está mal sin tener que adivinar. En ML necesitas ambas, y necesitas monitorear cosas que un sistema de software tradicional ni siquiera contempla: la calidad estadística de las predicciones, no solo si el servicio está "arriba" o "abajo".

## 1. Las cuatro categorías de monitoreo en un sistema de ML

### Monitoreo de infraestructura (lo que ya conoces de software tradicional)
Uptime, latencia de respuesta, uso de CPU/memoria, tasa de errores HTTP. Herramientas: **Prometheus** (recolección de métricas) + **Grafana** (visualización), ya mencionadas en [[09-MLOps-en-Profundidad]].

### Monitoreo de datos de entrada (Data Drift)
¿Los datos que le llegan al modelo hoy se parecen estadísticamente a los datos con los que fue entrenado? Si la distribución de `TotalDemand` histórica cambia radicalmente (por una promoción del cliente, un evento estacional no visto en entrenamiento, un cambio en cómo se registra el dato en la fuente), el modelo empieza a operar fuera de su zona de confianza aunque el código funcione perfecto.

### Monitoreo de predicciones (Concept Drift)
¿La relación entre las variables de entrada y la variable objetivo sigue siendo la misma? Esto es más sutil que el data drift — los datos de entrada pueden verse normales, pero la relación subyacente cambió (ej. un nuevo canal de atención al cliente altera el patrón histórico entre volumen de llamadas y tiempo de servicio).

### Monitoreo de negocio (Impacto real)
¿El modelo sigue generando el valor que se esperaba? Para tu pipeline, esto sería: ¿la dotación de agentes calculada realmente resulta en niveles de servicio aceptables, o hay sobre/sub dotación sistemática? Esta es, al final del día, la métrica que le importa a tu jefa — más que el MAE en sí mismo.

---

## 2. Data Drift en detalle — cómo se detecta

Las dos pruebas estadísticas más usadas para detectar que la distribución de una variable cambió:

- **Kolmogorov-Smirnov (KS test)**: compara la distribución acumulada de una variable numérica entre el set de referencia (entrenamiento) y el set actual (producción); un p-valor bajo indica que las distribuciones son significativamente distintas.
- **Population Stability Index (PSI)**: mide qué tanto cambió la distribución de una variable dividida en buckets, muy usado en la industria financiera y fácilmente adoptable en forecasting. Un PSI > 0.2 suele considerarse una señal de alerta.

```python
from evidently.report import Report
from evidently.metric_preset import DataDriftPreset

report = Report(metrics=[DataDriftPreset()])
report.run(reference_data=train_df, current_data=production_df)
report.save_html("drift_report.html")
```

**Evidently** (visto en [[07-Librerias-de-Data-Science-y-ML]]) automatiza este cálculo columna por columna y genera reportes visuales — mucho más práctico que calcular KS/PSI a mano para cada feature.

---

## 3. Concept Drift — más difícil de detectar

A diferencia del data drift, el concept drift a veces solo se nota cuando ya tienes el **valor real** (ground truth) para comparar contra la predicción — lo cual, en forecasting, puede tardar días o semanas en estar disponible. Estrategias:

- **Monitoreo de métricas retrasadas**: calcular MAE/RMSE real en cuanto el dato real esté disponible, y graficar su tendencia en el tiempo — un MAE que sube sostenidamente sesión tras sesión es la señal más confiable de concept drift.
- **Proxies de corto plazo**: cuando el ground truth tarda mucho, algunos equipos monitorean señales indirectas correlacionadas (ej. quejas de clientes, tickets de sobrecarga) como alerta temprana mientras se espera el dato real.

---

## 4. Qué loguear y qué graficar — el puente con [[11-Logging-en-Python-para-ML]]

Un dashboard de monitoreo de ML típico (en Grafana, alimentado por Prometheus o por métricas exportadas desde tus logs estructurados) debería incluir:

- Distribución de predicciones por corrida (¿hay valores atípicos extremos?).
- Tiempo de ejecución del pipeline por corrida (¿se está degradando el rendimiento del job?).
- Tasa de filas rechazadas por validación de datos (conexión directa con [[13-Testing-en-Machine-Learning]]).
- PSI/KS por feature clave, actualizado en cada corrida.
- MAE/RMSE reales vs. predichos, en cuanto el dato real esté disponible.
- Frecuencia de activación del reentrenamiento automático y resultado del gate champion/challenger.

---

## 5. Alertas — de qué sirve monitorear si nadie se entera a tiempo

Monitorear sin alertar es tener un panel bonito que nadie mira hasta que ya es tarde. Reglas prácticas:

- Define **umbrales accionables**, no solo "bonitos de ver". Un PSI > 0.25 en una variable clave debería disparar una alerta a Slack/Teams, no quedarse solo en un dashboard.
- Evita la **fatiga de alertas**: si todo dispara una alerta, la gente empieza a ignorarlas todas. Reserva alertas críticas (Slack/PagerDuty) para lo que realmente requiere acción inmediata, y deja el resto como visualización pasiva en el dashboard.
- Cada alerta debería apuntar a una acción clara: "revisar manualmente", "el reentrenamiento automático ya se disparó", "requiere intervención humana urgente".

---

## 6. Observabilidad como cultura, no solo como herramienta

La diferencia real entre un equipo junior y uno senior no es cuántas herramientas de monitoreo tiene instaladas, sino si **realmente las revisa y actúa sobre ellas** de forma rutinaria. Un dashboard de drift que nadie mira en semanas es exactamente tan útil como no tenerlo.

## Ver también

- [[11-Logging-en-Python-para-ML]]
- [[13-Testing-en-Machine-Learning]]
- [[09-MLOps-en-Profundidad]]
- [[19-Mantenimiento-y-Ciclo-de-Vida-del-Modelo]]
