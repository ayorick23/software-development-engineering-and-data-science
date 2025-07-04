# Indexes

Los **índices** en SQL son objetos de base de datos especiales que mejoran significativamente la velocidad de recuperación de datos de una tabla. Piensa en un índice como el índice de un libro: en lugar de hojear cada página para encontrar un tema específico, usas el índice para ir directamente a la página correcta. De manera similar, los índices permiten que el motor de la base de datos encuentre filas de datos más rápidamente sin tener que escanear toda la tabla (lo que se conoce como "escaneo de tabla" o table scan).

Aunque los índices aceleran las consultas [[DML (Data Manipulation Language)#`SELECT`|SELECT]], tienen un costo:

- **Espacio de almacenamiento:** Los índices requieren espacio en disco para almacenarse.
- **Rendimiento en escritura:** Las operaciones [[DML (Data Manipulation Language)#`INSERT`|INSERT]], [[DML (Data Manipulation Language)#`UPDATE`|UPDATE]] y [[DML (Data Manipulation Language)#`DELETE`|DELETE]] pueden volverse más lentas porque el índice también debe ser actualizado cada vez que cambian los datos de la tabla.

	Por lo tanto, la clave está en usar índices de manera estratégica: indexar las columnas que se utilizan frecuentemente en cláusulas [[DQL (Data Query Language)#3. `WHERE` (_Filtrado de Filas_)|WHERE]], [[Joins|JOIN]], [[DQL (Data Query Language)#6. `ORDER BY` (_Ordenación de Resultados_)|ORDER BY]] y [[DQL (Data Query Language)#4. `GROUP BY` (_Agrupación de Filas_)|GROUP BY]].

## Tipos de Índices

Existen dos tipos principales de índices, y su implementación puede variar ligeramente entre los diferentes sistemas de gestión de bases de datos (DBMS) como MySQL, PostgreSQL o SQL Server.

### 1. Índices Clustered (Clúster)

- **Organización física:** Un índice `CLUSTERED` determina el orden físico en que se almacenan los datos en la tabla. Esto significa que los datos de la tabla se ordenan según el índice `CLUSTERED`.
- **Uno por tabla:** Una tabla solo puede tener un índice `CLUSTERED`. Si defines una [[Constraints#`PRIMARY KEY` - Clave Primaria|PRIMARY KEY]] en una tabla, muchos DBMS (como SQL Server) la crean automáticamente como un índice `CLUSTERED` por defecto.
- **Acceso rápido a rangos:** Son extremadamente eficientes para consultas de rango (ej., `WHERE fecha BETWEEN '2024-01-01' AND '2024-01-31'`) o para recuperar un conjunto ordenado de datos.
- **Lecturas de tabla completas:** Si una consulta necesita todas las columnas de una tabla y ya está ordenada por el índice `CLUSTERED`, la lectura es muy eficiente.

### 2. Índices Non-Clustered (No Clúster)

- **Organización lógica:** Un índice `NON-CLUSTERED` no afecta el orden físico de los datos. En su lugar, es una estructura separada que contiene la clave del índice y un puntero a la ubicación física de la fila de datos en la tabla (o al índice `CLUSTERED` si existe).
- **Múltiples por tabla:** Una tabla puede tener múltiples índices `NON-CLUSTERED`.
- **Mejor para búsquedas específicas:** Son ideales para consultas que buscan valores específicos (ej., `WHERE email = 'ejemplo@dominio.com'`).
- **Cobertura:** Pueden "cubrir" una consulta si todas las columnas que la consulta necesita están incluidas en el índice `NON-CLUSTERED`, lo que permite que el motor de la base de datos obtenga toda la información directamente del índice sin tener que ir a la tabla principal.

## Creación de Índices (`CREATE INDEX`)

La sentencia [[DDL (Data Definition Language)#`CREATE`|CREATE]] `INDEX` se utiliza para crear un índice en una tabla.

```sql
CREATE [UNIQUE] [CLUSTERED | NONCLUSTERED] INDEX nombre_indice
ON nombre_tabla (columna1 [ASC|DESC], columna2 [ASC|DESC], ...);
```

- `UNIQUE`: Opcional. Crea un índice que garantiza que todos los valores en la columna(s) indexada(s) sean únicos.
- `CLUSTERED | NONCLUSTERED`: Opcional. Especifica el tipo de índice. Si se omite, el comportamiento por defecto depende del DBMS (puede ser `NONCLUSTERED` o, si es una `PRIMARY KEY`, a menudo `CLUSTERED`).

## Eliminación de Índices (`DROP INDEX`)

La sentencia [[DDL (Data Definition Language)#`DROP`|DROP]] `INDEX` se utiliza para eliminar un índice de una tabla. La sintaxis puede variar ligeramente entre DBMS.

```sql
-- SQL Server y MySQL
DROP INDEX nombre_indice ON nombre_tabla;

-- PostgreSQL
DROP INDEX nombre_indice; -- El nombre del índice debe ser único en el esquema
-- O DROP INDEX nombre_tabla_nombre_columna_idx; -- Si es un índice creado automáticamente
```

## Cuándo y Dónde Usar Índices

Los índices son más beneficiosos en las siguientes situaciones:

- **Columnas utilizadas en cláusulas `WHERE`:** Esto permite que la base de datos encuentre rápidamente las filas que cumplen la condición.
- **Columnas utilizadas en cláusulas `JOIN`:** Las columnas que se usan para vincular tablas (generalmente claves primarias y foráneas) son excelentes candidatas para la indexación.
- **Columnas utilizadas en cláusulas `ORDER BY` y `GROUP BY`:** Los índices pueden ayudar a la base de datos a ordenar o agrupar los resultados de manera más eficiente, a veces incluso evitando un paso de ordenación.
- **Columnas con alta cardinalidad (muchos valores únicos):** Por ejemplo, un `email` o un `ID` son buenos candidatos. Una columna como `genero` (con solo dos valores) generalmente no se beneficia mucho de un índice.
- **Tablas grandes:** El impacto de los índices es mucho más notorio en tablas con un gran volumen de datos.
