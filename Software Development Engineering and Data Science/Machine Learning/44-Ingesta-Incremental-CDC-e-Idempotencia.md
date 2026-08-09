---
tags: [ingenieria-de-datos, idempotencia, cdc, pipelines]
---

# 44 — Ingesta Incremental, CDC e Idempotencia en Pipelines de Datos

> Nota del mentor: tu pipeline corre cada 30 minutos, escribiendo en `qf.hrAgentForecastResult`. La pregunta que separa a un pipeline robusto de uno frágil es: **¿qué pasa si el pipeline se ejecuta dos veces seguidas por error, o si falla a mitad de camino y se reintenta?**. Si la respuesta es "se duplican datos" o "queda en un estado inconsistente", tienes un problema de idempotencia — uno de los conceptos más subestimados en ingeniería de datos.

## 1. Idempotencia — la propiedad que hace tu pipeline seguro de reintentar

**Una operación es idempotente si ejecutarla múltiples veces produce el mismo resultado que ejecutarla una sola vez.**

```python
# NO idempotente — cada ejecución agrega filas nuevas, duplicando datos si se corre dos veces
cursor.execute("INSERT INTO qf.hrAgentForecastResult (office_id, interval_start, agentes) VALUES (?, ?, ?)",
               office_id, interval_start, agentes_requeridos)

# Idempotente — MERGE garantiza el mismo resultado sin importar cuántas veces se ejecute
cursor.execute("""
    MERGE qf.hrAgentForecastResult AS destino
    USING (SELECT ? AS office_id, ? AS interval_start, ? AS agentes) AS origen
    ON destino.office_id = origen.office_id AND destino.interval_start = origen.interval_start
    WHEN MATCHED THEN UPDATE SET destino.agentes = origen.agentes
    WHEN NOT MATCHED THEN INSERT (office_id, interval_start, agentes) VALUES (origen.office_id, origen.interval_start, origen.agentes)
""", office_id, interval_start, agentes_requeridos)
```

Esto conecta directamente con el `MERGE` que ya viste en [[26-SQL-Profesional-Nivel-1-Consultas-Avanzadas]] y [[29-SQL-para-Machine-Learning]] — ahora entiendes **por qué** ese patrón de staging + MERGE es la elección correcta para un pipeline en producción, no solo una preferencia estilística: es lo que garantiza idempotencia real ante reintentos.

## 2. Por qué esto importa tanto en un pipeline programado (scheduled)

Un pipeline que corre cada 30 minutos, sin supervisión humana directa, **eventualmente** va a fallar a mitad de ejecución — un timeout de red, un reinicio del servidor, un error transitorio de SQL Server. Cuando eso pase, tienes dos opciones de diseño:

- **Pipeline no idempotente**: el reintento después de un fallo parcial puede duplicar datos, dejar la tabla en un estado inconsistente (algunas oficinas actualizadas, otras no), o requerir intervención manual para "limpiar" antes de poder reintentar con seguridad.
- **Pipeline idempotente**: puedes simplemente **volver a correr el pipeline completo** después de cualquier fallo, sin pensar, sin limpiar nada manualmente — el resultado final es el mismo sin importar cuántas veces se ejecutó parcialmente antes.

La idempotencia no es un "nice to have" — es lo que te permite dormir tranquilo sabiendo que un reintento automático (o manual) nunca corrompe silenciosamente los datos que consume el negocio.

## 3. Change Data Capture (CDC) — detectar qué cambió sin releer todo

El problema que resuelve: `qf.hrGetAbandonData` no tiene filtro de fecha y devuelve el histórico completo cada vez — esto funciona, pero es ineficiente si solo necesitas saber qué cambió desde la última ejecución. CDC es el patrón de **detectar y propagar solo los cambios incrementales** de una fuente de datos, en vez de reprocesar todo el histórico en cada corrida.

### Patrón 1 — columna de timestamp de actualización (el más simple de implementar)
```sql
SELECT * FROM qf.hrCallData
WHERE fecha_actualizacion > @ultima_fecha_procesada;
```
Requiere que la tabla fuente tenga una columna confiable que se actualice en cada `INSERT`/`UPDATE` — simple, pero depende completamente de que esa columna exista y sea confiable en el sistema origen (algo que no siempre controlas si la fuente es un sistema de terceros como Qflow).

### Patrón 2 — CDC nativo de SQL Server
```sql
EXEC sys.sp_cdc_enable_table
    @source_schema = 'qf', @source_name = 'hrCallData', @role_name = NULL;

SELECT * FROM cdc.fn_cdc_get_all_changes_qf_hrCallData(@from_lsn, @to_lsn, 'all');
```
SQL Server tiene una función de CDC integrada que rastrea automáticamente inserts/updates/deletes a nivel de log de transacciones, sin depender de que la aplicación mantenga una columna de timestamp manualmente — más robusto, pero requiere habilitarlo explícitamente y permisos administrativos sobre la base de datos, algo que en un contexto de cliente (Claro RD) probablemente requiere coordinación con su equipo de DBA.

### Patrón 3 — snapshot y comparación (cuando no hay CDC disponible)
```python
snapshot_actual = obtener_snapshot_completo()
cambios = comparar_con_snapshot_anterior(snapshot_actual, snapshot_guardado_previamente)
```
El patrón menos eficiente pero más universal — funciona sin ningún soporte especial de la fuente de datos, al costo de tener que leer y comparar todo el volumen en cada corrida, exactamente lo que ya haces hoy con `hrGetAbandonData`.

## 4. Watermarking — manejar datos que llegan tarde (late-arriving data)

Un problema real en sistemas de call centers: un registro de una llamada puede insertarse en la base de datos minutos después de que realmente ocurrió (latencia del sistema origen). Si tu pipeline procesa "todo lo nuevo desde la última ejecución" basado estrictamente en timestamp de evento, puede perderse registros que llegaron tarde.

```python
# Watermark: procesa con un margen de seguridad hacia atrás, no el límite exacto
margen_seguridad = timedelta(minutes=15)
fecha_desde = ultima_fecha_procesada - margen_seguridad
```

Este margen de seguridad, combinado con la idempotencia del `MERGE` (sección 1), te permite reprocesar de forma segura una ventana que se solapa parcialmente con la corrida anterior, capturando registros tardíos sin duplicar los que ya se procesaron correctamente.

## 5. Cómo se conecta todo — el diseño robusto completo

```
1. CDC / timestamp de actualización → identifica qué cambió desde la última corrida
2. Watermarking → agrega margen de seguridad para datos que llegaron tarde
3. Staging table → los datos nuevos/cambiados aterrizan primero en una tabla intermedia
4. MERGE idempotente → los cambios se aplican a la tabla final de forma segura ante reintentos
5. Logging estructurado (nota 11) → registra exactamente qué se procesó, cuándo y cuántas filas
```

Este es, en esencia, el diseño que separa un pipeline de nivel júnior ("lee todo, procesa todo, escribe todo, cada 30 minutos, sin pensar en fallos") de un pipeline de nivel senior — uno que asume que **va a fallar eventualmente** y está diseñado para tolerar ese fallo sin intervención manual ni corrupción de datos.

## Ver también
- [[26-SQL-Profesional-Nivel-1-Consultas-Avanzadas]]
- [[29-SQL-para-Machine-Learning]]
- [[23-Manejo-Profesional-de-Errores]]
- [[19-Mantenimiento-y-Ciclo-de-Vida-del-Modelo]]
