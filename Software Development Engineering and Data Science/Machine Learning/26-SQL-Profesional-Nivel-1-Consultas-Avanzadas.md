---
tags: [sql, sql-server, avanzado]
---

# 26 — SQL Profesional Nivel 1: Consultas Avanzadas (CTE, Window Functions, CASE, APPLY, MERGE)

> Nota del mentor: llevas meses escribiendo `SELECT * FROM qf.hrCallData WHERE ...` contra SQL Server. Este es el punto donde dejas de escribir SQL "que funciona" y empiezas a escribir SQL que **un motor de bases de datos puede optimizar bien** y que **otro humano puede leer sin sufrir**. Todo lo de esta nota es SQL Server / T-SQL, el dialecto que usas en Claro RD.

## 1. CTE (Common Table Expressions) — nombrar tus pasos intermedios

```sql
WITH DemandaDiaria AS (
    SELECT
        office_id,
        CAST(interval_start AS DATE) AS fecha,
        SUM(total_demand) AS demanda_total_dia
    FROM qf.hrCallData
    WHERE interval_start >= DATEADD(DAY, -90, GETDATE())
    GROUP BY office_id, CAST(interval_start AS DATE)
),
DemandaConPromedio AS (
    SELECT
        office_id,
        fecha,
        demanda_total_dia,
        AVG(demanda_total_dia) OVER (PARTITION BY office_id) AS promedio_90d
    FROM DemandaDiaria
)
SELECT *
FROM DemandaConPromedio
WHERE demanda_total_dia > promedio_90d * 1.5  -- días de demanda anómala
ORDER BY office_id, fecha;
```

Un CTE es, en esencia, darle nombre a una subconsulta para poder referenciarla después — como asignar una variable intermedia en Python en vez de anidar todo en una sola expresión ilegible. La diferencia frente a una subquery anidada tradicional no es de rendimiento (el optimizador de SQL Server suele generar el mismo plan de ejecución), es de **legibilidad y mantenibilidad**: puedes leer el query de arriba hacia abajo como una secuencia de pasos con nombre, en vez de descifrar paréntesis anidados cinco niveles de profundidad.

**CTE recursivo** — para jerarquías (menos común en tu dominio de forecasting, pero vale conocerlo):

```sql
WITH JerarquiaOficinas AS (
    SELECT office_id, office_padre_id, 0 AS nivel
    FROM qf.hrOffices WHERE office_padre_id IS NULL
    UNION ALL
    SELECT o.office_id, o.office_padre_id, j.nivel + 1
    FROM qf.hrOffices o
    INNER JOIN JerarquiaOficinas j ON o.office_padre_id = j.office_id
)
SELECT * FROM JerarquiaOficinas;
```

## 2. Window Functions — el cambio de mentalidad más importante de esta nota

Una window function calcula un valor **para cada fila**, mirando un conjunto ("ventana") de filas relacionadas, **sin colapsar el resultado** como haría un `GROUP BY`.

```sql
SELECT
    office_id,
    interval_start,
    total_demand,
    -- lag: el valor de la fila anterior — exactamente lo que necesitas para features de lag
    LAG(total_demand, 1) OVER (PARTITION BY office_id ORDER BY interval_start) AS demand_lag_1,
    LAG(total_demand, 48) OVER (PARTITION BY office_id ORDER BY interval_start) AS demand_lag_1dia,

    -- promedio móvil de las últimas 7 filas (ventana deslizante)
    AVG(total_demand) OVER (
        PARTITION BY office_id ORDER BY interval_start
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS promedio_movil_7,

    -- ranking dentro de cada oficina
    RANK() OVER (PARTITION BY office_id ORDER BY total_demand DESC) AS ranking_demanda,

    -- porcentaje acumulado
    SUM(total_demand) OVER (
        PARTITION BY office_id ORDER BY interval_start
        ROWS UNBOUNDED PRECEDING
    ) AS demanda_acumulada
FROM qf.hrCallData;
```

Esto es, literalmente, cómo puedes calcular tus **features de lag directamente en SQL** en vez de en pandas después de traer todos los datos — algo que en un pipeline con millones de filas cambia radicalmente el tiempo de ejecución, porque el trabajo se hace donde vive el dato, con índices, en vez de traer todo a Python y procesarlo en memoria.

**Diferencia clave entre `ROW_NUMBER`, `RANK` y `DENSE_RANK`** — un error de junior muy común:

```sql
-- Si hay empates en total_demand:
-- ROW_NUMBER(): 1, 2, 3, 4  (siempre consecutivo, rompe empates arbitrariamente)
-- RANK():       1, 2, 2, 4  (empates comparten posición, salta el siguiente número)
-- DENSE_RANK(): 1, 2, 2, 3  (empates comparten posición, sin saltos)
```

## 3. `CASE` — más allá del `if/else` básico

```sql
SELECT
    office_id,
    total_demand,
    CASE
        WHEN total_demand = 0 THEN 'sin_actividad'
        WHEN total_demand < 10 THEN 'baja'
        WHEN total_demand BETWEEN 10 AND 50 THEN 'media'
        ELSE 'alta'
    END AS categoria_demanda,

    -- CASE dentro de una agregación: patrón de "pivoteo condicional"
    SUM(CASE WHEN DATEPART(WEEKDAY, interval_start) IN (1, 7) THEN total_demand ELSE 0 END) AS demanda_fin_semana,
    SUM(CASE WHEN DATEPART(WEEKDAY, interval_start) NOT IN (1, 7) THEN total_demand ELSE 0 END) AS demanda_entre_semana
FROM qf.hrCallData
GROUP BY office_id;
```

El patrón `SUM(CASE WHEN ... THEN valor ELSE 0 END)` es la forma estándar en T-SQL de hacer un pivot condicional sin usar `PIVOT` explícito — muy usado para construir features segmentadas (demanda de fin de semana vs. entre semana) directamente en la consulta.

## 4. `CROSS APPLY` y `OUTER APPLY` — joins con lógica, no solo con columnas

Esto es lo que más diferencia a alguien que domina T-SQL de alguien que solo conoce SQL estándar. `APPLY` te permite unir una tabla con el resultado de una **subconsulta que depende de cada fila** de la tabla externa — algo que un `JOIN` tradicional no puede expresar directamente.

```sql
-- Para cada oficina, obtener sus 3 intervalos de mayor demanda histórica
SELECT o.office_id, o.office_name, top3.interval_start, top3.total_demand
FROM qf.hrOffices o
CROSS APPLY (
    SELECT TOP 3 interval_start, total_demand
    FROM qf.hrCallData c
    WHERE c.office_id = o.office_id
    ORDER BY total_demand DESC
) AS top3;
```

`CROSS APPLY` se comporta como un `INNER JOIN` (si la subconsulta no devuelve filas para una oficina, esa oficina desaparece del resultado). `OUTER APPLY` se comporta como `LEFT JOIN` (conserva la oficina aunque la subconsulta no devuelva nada, con `NULL` en las columnas de la subconsulta):

```sql
SELECT o.office_id, o.office_name, ultimo.interval_start, ultimo.total_demand
FROM qf.hrOffices o
OUTER APPLY (
    SELECT TOP 1 interval_start, total_demand
    FROM qf.hrCallData c
    WHERE c.office_id = o.office_id
    ORDER BY interval_start DESC
) AS ultimo;
-- oficinas sin ningún registro en hrCallData igual aparecen, con NULL
```

## 5. `MERGE` — insertar, actualizar o eliminar en una sola sentencia

```sql
MERGE qf.hrAgentForecastResult AS destino
USING #ResultadosNuevos AS origen
ON destino.office_id = origen.office_id
   AND destino.interval_start = origen.interval_start
WHEN MATCHED THEN
    UPDATE SET
        destino.total_demand_predict = origen.total_demand_predict,
        destino.agentes_requeridos = origen.agentes_requeridos,
        destino.fecha_actualizacion = GETDATE()
WHEN NOT MATCHED BY TARGET THEN
    INSERT (office_id, interval_start, total_demand_predict, agentes_requeridos, fecha_actualizacion)
    VALUES (origen.office_id, origen.interval_start, origen.total_demand_predict, origen.agentes_requeridos, GETDATE())
WHEN NOT MATCHED BY SOURCE AND destino.interval_start < DATEADD(DAY, -1, GETDATE()) THEN
    DELETE;
```

Esto reemplaza el patrón manual de "primero hago `DELETE` de lo viejo, luego hago `INSERT` de lo nuevo" (que puede dejar la tabla en un estado inconsistente si el proceso falla a mitad de camino) por una sola operación atómica: inserta lo nuevo, actualiza lo existente, y opcionalmente limpia lo que ya no debería estar — exactamente el patrón que necesitas cada vez que tu pipeline reescribe resultados de forecast en `qf.hrAgentForecastResult` cada 30 minutos.

**Precaución real de producción**: `MERGE` en SQL Server ha tenido bugs documentados de Microsoft en condiciones de alta concurrencia (varias sesiones ejecutándolo simultáneamente sobre la misma tabla) — si tu pipeline corre con múltiples workers en paralelo sobre la misma tabla destino, vale la pena usar bloqueos explícitos (`WITH (HOLDLOCK)`) o validar con tu DBA antes de confiar en `MERGE` bajo concurrencia alta.

## Ver también
- [[04-Ingenieria-de-Datos]]
- [[27-SQL-Profesional-Nivel-2-Optimizacion]]
- [[29-SQL-para-Machine-Learning]]
