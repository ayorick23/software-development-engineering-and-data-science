---
tags: [sql, modelado-de-datos, data-warehouse]
---

# 28 — SQL Profesional Nivel 3: Modelado de Datos (Normalización, Star Schema, Snowflake, SCD)

> Nota del mentor: hasta ahora hablamos de cómo consultar y optimizar datos que ya existen. Esta nota es sobre cómo **diseñar** cómo se guardan desde el principio — una decisión que, si se hace mal, ningún índice ni query bien escrito puede compensar completamente después. Conecta directo con [[03-Arquitectura-Empresarial-de-Datos-y-ML]], donde ya viste el flujo completo ETL → Warehouse.

## 1. Normalización — eliminar redundancia en bases transaccionales (OLTP)

La normalización organiza datos para que **cada hecho se almacene en un solo lugar**, evitando inconsistencias por actualización.

- **1FN (Primera Forma Normal)**: cada columna tiene un solo valor atómico, no listas ni valores repetidos en una celda.
- **2FN**: además de 1FN, cada columna no clave depende de **toda** la clave primaria, no de una parte de ella (relevante en tablas con clave compuesta).
- **3FN**: además de 2FN, ninguna columna no clave depende de otra columna no clave (elimina dependencias transitivas).

```sql
-- Sin normalizar: el nombre de la oficina se repite en cada fila de demanda
-- office_id | office_name        | interval_start | total_demand
-- 145       | Santo Domingo Este | ...             | 12.5
-- 145       | Santo Domingo Este | ...             | 15.2

-- Normalizado: el nombre vive en una sola tabla, referenciado por ID
CREATE TABLE qf.hrOffices (office_id INT PRIMARY KEY, office_name VARCHAR(100));
CREATE TABLE qf.hrCallData (
    office_id INT REFERENCES qf.hrOffices(office_id),
    interval_start DATETIME,
    total_demand FLOAT
);
```

**¿Por qué normalizar?** Si `office_name` cambia (una oficina se renombra), lo actualizas en un solo lugar, no en millones de filas de `hrCallData`. Esto es correcto para sistemas **transaccionales** (OLTP) donde el objetivo es integridad y consistencia de escritura — como probablemente es la estructura fuente de Qflow/StaffManager.

## 2. Star Schema — el modelo que sí quieres para analítica y ML

Un data warehouse (visto en [[03-Arquitectura-Empresarial-de-Datos-y-ML]] y [[04-Ingenieria-de-Datos]]) prioriza **velocidad de lectura para análisis**, no integridad transaccional — y aquí la normalización estricta se vuelve un obstáculo, no una ayuda.

```
                    ┌──────────────────┐
                    │  DimOficina        │
                    │  office_id (PK)     │
                    │  office_name         │
                    │  region               │
                    └─────────┬──────────┘
                              │
┌──────────────────┐    ┌────┴─────────────────┐   ┌──────────────────┐
│  DimTiempo         │────│  FactDemanda          │───│  DimTurno          │
│  fecha_id (PK)      │    │  office_id (FK)       │   │  turno_id (PK)      │
│  dia_semana          │    │  fecha_id (FK)         │   │  nombre_turno        │
│  mes, trimestre       │    │  turno_id (FK)         │   └──────────────────┘
└──────────────────┘    │  total_demand           │
                          │  avg_service_time        │
                          │  agentes_requeridos        │
                          └──────────────────────┘
```

- **Tabla de hechos (Fact)**: contiene las **métricas numéricas** que quieres analizar (`total_demand`, `agentes_requeridos`) y claves foráneas hacia las dimensiones. Crece rápido — millones de filas.
- **Tablas de dimensión (Dim)**: contienen los **atributos descriptivos** (nombre de oficina, región, día de la semana) — crecen lento, se consultan constantemente para filtrar y agrupar.

La ventaja frente a un modelo normalizado tradicional: consultas analíticas típicas ("demanda total por región y trimestre") requieren pocos `JOIN` simples (fact → dims directamente), en vez de atravesar múltiples niveles de tablas normalizadas — esto es lo que hace que un Star Schema sea dramáticamente más rápido para las agregaciones masivas que necesita [[29-SQL-para-Machine-Learning]].

## 3. Snowflake Schema — cuando las dimensiones también se normalizan

```
FactDemanda → DimOficina → DimRegion → DimPais
```

Es un Star Schema donde las dimensiones **se normalizan un nivel más**, en vez de tener toda la jerarquía (oficina, región, país) aplanada en una sola tabla `DimOficina`. Ahorra espacio y evita redundancia en dimensiones muy grandes, pero exige más `JOIN` por consulta.

**Criterio práctico de elección**: Star Schema por defecto — es más simple de consultar y de explicar a analistas de negocio. Solo migra a Snowflake si una dimensión es realmente enorme (millones de filas) y la redundancia del Star Schema pesa más que el costo de los `JOIN` extra — en la mayoría de proyectos de tamaño mediano como el tuyo, el espacio ahorrado no compensa la complejidad añadida.

## 4. Slowly Changing Dimensions (SCD) — cómo manejar cambios en las dimensiones

El problema: una oficina cambia de región, un empleado cambia de puesto — ¿sobrescribes el dato viejo, o conservas el historial?

### SCD Tipo 1 — sobrescribir (sin historial)
```sql
UPDATE DimOficina SET region = 'Santo Domingo Norte' WHERE office_id = 145;
```
Simple, pero **pierdes el historial**: si analizas datos de hace un año, la oficina aparece con la región actual, no con la que tenía entonces. Correcto solo cuando el valor histórico realmente no importa (ej. corrección de un typo en el nombre).

### SCD Tipo 2 — nueva fila con vigencia (con historial completo)
```sql
CREATE TABLE DimOficina (
    office_key INT IDENTITY PRIMARY KEY,  -- clave subrogada, no el office_id de negocio
    office_id INT,                         -- clave de negocio, se repite entre versiones
    region VARCHAR(50),
    fecha_inicio DATE,
    fecha_fin DATE NULL,  -- NULL = versión vigente actual
    es_actual BIT
);

-- Al detectar el cambio: cierro la versión anterior, abro una nueva
UPDATE DimOficina SET fecha_fin = '2026-06-30', es_actual = 0
WHERE office_id = 145 AND es_actual = 1;

INSERT INTO DimOficina (office_id, region, fecha_inicio, fecha_fin, es_actual)
VALUES (145, 'Santo Domingo Norte', '2026-07-01', NULL, 1);
```
Esto conserva historial completo y permite hacer análisis correctos "point-in-time" (cómo estaban las cosas exactamente en la fecha de cada hecho histórico) — el patrón más usado en warehouses maduros, y el correcto por defecto cuando el historial de cambios importa para el análisis (que es casi siempre en contextos de forecasting de largo plazo).

### SCD Tipo 3 — columna adicional (historial limitado)
```sql
ALTER TABLE DimOficina ADD region_anterior VARCHAR(50);
```
Guarda solo el valor anterior inmediato, no toda la secuencia histórica — un compromiso poco usado en la práctica salvo casos muy específicos, porque pierde información si hay más de un cambio.

## 5. Conexión directa con tu trabajo diario

Aunque `qf.hrAgentForecastResult` probablemente no es un warehouse formal con Star Schema, entender este modelo te ayuda a razonar sobre **cómo estructurarías** un modelo analítico si algún día propones consolidar el histórico de forecasting de todos los clientes de ACF Technologies (no solo Claro RD) en un solo lugar consultable — la pregunta de "¿esto es un hecho medible, o un atributo descriptivo?" es la pregunta central de todo diseño de modelado dimensional.

## Ver también
- [[03-Arquitectura-Empresarial-de-Datos-y-ML]]
- [[04-Ingenieria-de-Datos]]
- [[29-SQL-para-Machine-Learning]]
