---
tags: [sql, machine-learning, feature-engineering]
---

# 29 — SQL para Machine Learning: Agregaciones Masivas, Feature Engineering, Joins Eficientes, Staging

> Nota del mentor: esta nota es donde todo lo anterior (nivel 1, 2 y 3) converge en tu trabajo real. La pregunta que un ingeniero de ML senior siempre se hace antes de escribir una línea de pandas es: **"¿esto lo puede hacer SQL Server de forma más eficiente que yo trayendo los datos a Python?"**. Casi siempre, la respuesta es sí.

## 1. El principio general: mueve el cómputo a donde vive el dato

```python
# Patrón ineficiente (común en código heredado)
df = pd.read_sql("SELECT * FROM qf.hrCallData", conn)  # trae MILLONES de filas
resultado = df.groupby("office_id")["total_demand"].sum()  # agrega en Python

# Patrón correcto: la agregación masiva ocurre en SQL, Python solo recibe el resultado
resultado = pd.read_sql("""
    SELECT office_id, SUM(total_demand) AS demanda_total
    FROM qf.hrCallData
    GROUP BY office_id
""", conn)
```

La diferencia de rendimiento no es marginal: SQL Server puede usar índices, paralelismo interno y operar sobre datos que nunca salen de memoria del servidor de base de datos — mientras que traer todo a pandas implica serializar cada fila, transportarla por red, y procesarla en un solo proceso Python. En un pipeline como el de Claro RD que corre cada 30 minutos, esta diferencia se traduce directamente en el tiempo total de ejecución del pipeline.

## 2. Agregaciones masivas — features estadísticos calculados en SQL

```sql
SELECT
    office_id,
    AVG(total_demand) AS demanda_promedio_90d,
    STDEV(total_demand) AS demanda_desviacion_90d,
    MIN(total_demand) AS demanda_minima_90d,
    MAX(total_demand) AS demanda_maxima_90d,
    -- percentiles con PERCENTILE_CONT (interpolado)
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY total_demand)
        OVER (PARTITION BY office_id) AS demanda_mediana_90d,
    COUNT(*) AS cantidad_intervalos
FROM qf.hrCallData
WHERE interval_start >= DATEADD(DAY, -90, GETDATE())
GROUP BY office_id;
```

Estos son exactamente los features estadísticos que en tu proyecto probablemente calculas en `feature_engineering.py` con pandas — calcularlos directamente en SQL, especialmente para históricos grandes, reduce la cantidad de datos que viajan de la base de datos a Python: en vez de traer millones de filas crudas para calcular un promedio en pandas, traes directamente el promedio ya calculado, una fila por oficina.

## 3. Feature engineering directo en SQL — lags, ventanas móviles, diferencias

```sql
SELECT
    office_id,
    interval_start,
    total_demand,
    LAG(total_demand, 1) OVER (PARTITION BY office_id ORDER BY interval_start) AS demand_lag_1,
    LAG(total_demand, 48) OVER (PARTITION BY office_id ORDER BY interval_start) AS demand_lag_1dia,
    AVG(total_demand) OVER (
        PARTITION BY office_id ORDER BY interval_start
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS promedio_movil_7,
    total_demand - LAG(total_demand, 1) OVER (PARTITION BY office_id ORDER BY interval_start) AS diferencia_intervalo_anterior
FROM qf.hrCallData;
```

Esto retoma directamente las window functions de [[26-SQL-Profesional-Nivel-1-Consultas-Avanzadas]], aplicadas explícitamente al caso de uso de features de series de tiempo. La decisión de **dónde** calcular cada feature (SQL vs. pandas) depende de:

- **Volumen**: features sobre todo el histórico → SQL. Features sobre un subconjunto pequeño ya en memoria → pandas está bien.
- **Complejidad de la lógica**: transformaciones muy específicas de dominio con múltiples condicionales anidados a veces son más legibles en Python que en SQL — no fuerces todo a SQL solo por principio.
- **Reutilización**: si el mismo feature se necesita en múltiples pipelines/proyectos, una vista SQL centralizada evita reimplementar la lógica en cada uno (y evita exactamente el tipo de duplicación cuádruple que corregiste al unificar `feature_engineering.py`).

## 4. Joins eficientes — el orden y el tipo importan

```sql
-- Trae solo lo necesario, filtra ANTES de unir cuando es posible
SELECT c.office_id, c.interval_start, c.total_demand, o.region
FROM (
    SELECT * FROM qf.hrCallData
    WHERE interval_start >= DATEADD(DAY, -90, GETDATE())
) AS c
INNER JOIN qf.hrOffices o ON c.office_id = o.office_id
WHERE o.activa = 1;
```

Filtrar `hrCallData` a los últimos 90 días **antes** del `JOIN` (subconsulta) reduce drásticamente el número de filas que el motor tiene que emparejar, comparado con unir las tablas completas y filtrar después. El optimizador de SQL Server a veces reordena esto automáticamente (revisa el plan de ejecución de [[27-SQL-Profesional-Nivel-2-Optimizacion]] para confirmarlo), pero escribirlo explícitamente ayuda tanto al optimizador como a cualquier humano que lea el query después.

**`INNER JOIN` vs `LEFT JOIN` — una decisión de negocio, no solo técnica**: si usas `INNER JOIN` entre `hrCallData` y `hrOffices` y una oficina fue eliminada de `hrOffices` pero sigue teniendo datos históricos, esa oficina **desaparece silenciosamente** de tus resultados. En un pipeline de forecasting, esto puede significar perder predicciones de una oficina real sin ningún error visible — usa `LEFT JOIN` cuando quieras conservar todos los registros del lado "principal" aunque no tengan coincidencia, y sé explícito sobre esta decisión en el código.

## 5. Tablas temporales y staging — el patrón correcto para pipelines de escritura

```sql
-- Tabla temporal: vive solo durante la sesión/procedimiento actual
CREATE TABLE #ResultadosNuevos (
    office_id INT,
    interval_start DATETIME,
    total_demand_predict FLOAT,
    agentes_requeridos INT
);

INSERT INTO #ResultadosNuevos
SELECT office_id, interval_start, total_demand_predict, agentes_requeridos
FROM staging.hrForecastResultsStaging;  -- datos que Python ya insertó ahí

-- Ahora el MERGE de la nota 26 usa esta tabla temporal como origen
MERGE qf.hrAgentForecastResult AS destino
USING #ResultadosNuevos AS origen
ON destino.office_id = origen.office_id AND destino.interval_start = origen.interval_start
WHEN MATCHED THEN UPDATE SET ...
WHEN NOT MATCHED THEN INSERT ...;
```

**¿Por qué no escribir directo en la tabla final?** El patrón de **staging** (tabla intermedia, luego `MERGE` hacia el destino) te da tres cosas que escribir directo no te da:

1. **Atomicidad**: si el proceso de Python falla a mitad de la carga, la tabla final nunca queda en un estado parcial/corrupto — el staging absorbe el fallo.
2. **Validación previa al impacto**: puedes correr validaciones de calidad de datos (ver [[13-Testing-en-Machine-Learning]], Pandera/Great Expectations) sobre el staging antes de que esos datos lleguen a la tabla que consume el negocio.
3. **Trazabilidad de auditoría**: si algo sale mal en producción, puedes inspeccionar exactamente qué llegó al staging en esa corrida, sin depender de logs de aplicación que puedan haberse rotado o perdido.

Una **tabla temporal** (`#nombre`) vive solo dentro de la sesión de SQL Server que la creó — ideal dentro de un procedimiento almacenado o script de una sola ejecución. Una **tabla de staging permanente** (`staging.hrForecastResultsStaging`) persiste entre ejecuciones y es la opción correcta cuando Python necesita escribir los datos primero y SQL Server los procesa después en un paso separado — exactamente el flujo típico entre tu script de Python y la base de datos de Claro RD.

## Ver también
- [[26-SQL-Profesional-Nivel-1-Consultas-Avanzadas]]
- [[27-SQL-Profesional-Nivel-2-Optimizacion]]
- [[11-Logging-en-Python-para-ML]]
- [[13-Testing-en-Machine-Learning]]
