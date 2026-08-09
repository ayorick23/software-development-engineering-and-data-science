---
tags: [sql, sql-server, optimizacion, performance]
---

# 27 — SQL Profesional Nivel 2: Optimización (Índices, Planes de Ejecución, Estadísticas, Particionamiento)

> Nota del mentor: escribir un query correcto es la mitad del trabajo. La otra mitad — la que distingue a un ingeniero senior — es saber por qué un query que "funciona" tarda 200ms en una tabla de prueba y 45 segundos contra `qf.hrCallData` con dos años de historial. Esta nota es sobre esa segunda mitad.

## 1. Índices — la estructura que decide si el motor "busca" o "escanea"

Sin índice, buscar una fila específica obliga al motor a un **Table Scan**: leer cada fila de la tabla completa. Con el índice correcto, el motor hace un **Index Seek**: salta directo a las filas relevantes, como buscar una palabra en el índice de un libro en vez de leer página por página.

### Clustered Index — define el orden físico de la tabla

```sql
-- Una tabla solo puede tener UN clustered index, porque define cómo se
-- almacenan físicamente los datos en disco
CREATE CLUSTERED INDEX IX_hrCallData_office_interval
ON qf.hrCallData (office_id, interval_start);
```

El clustered index no es "un índice más" — literalmente reordena las páginas físicas de la tabla según esa clave. En una tabla de forecasting como la tuya, ordenar físicamente por `(office_id, interval_start)` acelera dramáticamente las consultas que ya vimos con window functions (`PARTITION BY office_id ORDER BY interval_start`), porque los datos que necesitas ya están físicamente contiguos en disco.

### Nonclustered Index — un "índice de libro" adicional, apuntando a los datos

```sql
CREATE NONCLUSTERED INDEX IX_hrCallData_fecha
ON qf.hrCallData (interval_start)
INCLUDE (total_demand, avg_service_time);
```

Puedes tener múltiples nonclustered index por tabla. El `INCLUDE` es clave: agrega columnas al índice **sin que formen parte de la clave de búsqueda**, pero disponibles directamente en el índice — esto permite un **Covering Index**: si tu query solo necesita `interval_start`, `total_demand` y `avg_service_time`, el motor puede resolver todo desde el índice sin tocar la tabla base (`Key Lookup` evitado), lo cual es sustancialmente más rápido.

### El costo real de los índices — no son gratis

Cada índice acelera lecturas pero **ralentiza escrituras** (cada `INSERT`/`UPDATE`/`DELETE` debe actualizar también todos los índices) y consume espacio en disco. Regla práctica: indexa las columnas que usas en `WHERE`, `JOIN` y `ORDER BY` con frecuencia; no indexes "por si acaso" cada columna de una tabla que recibe escrituras constantes cada 30 minutos como la tuya.

## 2. Planes de ejecución — cómo leer lo que el motor realmente hace

```sql
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

SELECT office_id, SUM(total_demand)
FROM qf.hrCallData
WHERE interval_start >= '2026-01-01'
GROUP BY office_id;
```

Activa **"Include Actual Execution Plan"** en SSMS (Ctrl+M) antes de ejecutar. Lo que buscas en el plan resultante, de más a menos grave:

- **Table Scan / Clustered Index Scan** en una tabla grande: casi siempre señal de que falta un índice adecuado para ese filtro.
- **Key Lookup** con costo alto: el índice encontró las filas pero tuvo que volver a la tabla base a buscar columnas adicionales — candidato perfecto para agregar esas columnas con `INCLUDE`.
- **Grosor de las flechas** entre operadores: representan el número de filas que fluyen entre pasos — flechas gruesas inesperadas señalan que el optimizador está subestimando o sobreestimando el volumen de datos.
- **Costo relativo (%) de cada operador**: te dice dónde concentrar tu esfuerzo de optimización — no vale la pena optimizar un operador que representa el 2% del costo total del query.

## 3. Estadísticas — lo que el optimizador usa para "adivinar" el mejor plan

SQL Server no ejecuta un query a ciegas: usa **estadísticas** (histogramas de distribución de valores por columna) para estimar cuántas filas devolverá cada filtro, y con eso decide si usar un índice o un scan completo.

```sql
-- Ver cuándo se actualizaron por última vez las estadísticas de una tabla
DBCC SHOW_STATISTICS ('qf.hrCallData', IX_hrCallData_office_interval);

-- Forzar actualización manual si sospechas que están desactualizadas
UPDATE STATISTICS qf.hrCallData;
```

**Síntoma clásico de estadísticas desactualizadas**: un query que corría rápido durante meses de repente se vuelve lento después de una carga masiva de datos nuevos (como una recarga histórica completa), sin que nadie haya cambiado el código SQL. El optimizador sigue "creyendo" que la tabla tiene la distribución de datos de antes, y elige un plan de ejecución subóptimo para el volumen real actual.

## 4. Particionamiento — dividir una tabla enorme en piezas manejables

```sql
CREATE PARTITION FUNCTION PF_PorMes (DATE)
AS RANGE RIGHT FOR VALUES ('2026-01-01', '2026-02-01', '2026-03-01');

CREATE PARTITION SCHEME PS_PorMes
AS PARTITION PF_PorMes ALL TO ([PRIMARY]);

CREATE CLUSTERED INDEX IX_hrCallData_particionado
ON qf.hrCallData (interval_start)
ON PS_PorMes (interval_start);
```

Con esto, SQL Server puede hacer **Partition Elimination**: si tu query filtra por `interval_start >= '2026-02-01'`, el motor ni siquiera toca físicamente los datos de enero — solo lee las particiones relevantes. En una tabla con años de historial como `qf.hrCallData` (recuerda: `qf.hrGetAbandonData` no tiene filtro de fecha y devuelve todo el histórico), particionar por mes o trimestre puede ser la diferencia entre consultas de segundos y consultas de minutos, además de simplificar el mantenimiento (puedes purgar datos viejos eliminando particiones completas en vez de un `DELETE` masivo fila por fila, que además es mucho más costoso en el log de transacciones).

## 5. Errores de optimización comunes en pipelines de ML

- **`SELECT *` en un pipeline de producción**: trae columnas que nunca usas, impide covering indexes, y desperdicia ancho de banda de red entre SQL Server y Python. Siempre nombra las columnas exactas que necesitas.
- **Funciones sobre la columna filtrada** (`WHERE CAST(interval_start AS DATE) = '2026-01-01'`): esto hace el filtro **no-sargable** — el motor no puede usar un índice sobre `interval_start` directamente porque primero tiene que calcular la función en cada fila. Reescribe como rango: `WHERE interval_start >= '2026-01-01' AND interval_start < '2026-01-02'`.
- **Traer todo a pandas para filtrar/agregar en Python** lo que SQL Server podría filtrar/agregar mucho más eficientemente con índices — un antipatrón muy común en quien viene de un mundo "todo en pandas", y es exactamente lo que resuelven las window functions de [[26-SQL-Profesional-Nivel-1-Consultas-Avanzadas]] y las agregaciones de [[29-SQL-para-Machine-Learning]].

## Ver también
- [[26-SQL-Profesional-Nivel-1-Consultas-Avanzadas]]
- [[28-SQL-Profesional-Nivel-3-Modelado]]
- [[29-SQL-para-Machine-Learning]]
