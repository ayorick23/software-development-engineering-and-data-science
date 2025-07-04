# Views

Las **vistas** en SQL son tablas virtuales basadas en el conjunto de resultados de una consulta SQL. Una vista contiene filas y columnas al igual que una tabla real, pero los datos de una vista no se almacenan en la base de datos de forma independiente. En su lugar, los datos se obtienen de las tablas base subyacentes cada vez que se consulta la vista. Piensa en una vista como una ventana a tus datos, que presenta una versión personalizada y simplificada de una o más tablas.

Las vistas son una herramienta poderosa para:

1. **Simplificar consultas complejas:** Puedes encapsular [[Joins]], [[DQL (Data Query Language)#3. `WHERE` (_Filtrado de Filas_)|WHERE's]], [[DQL (Data Query Language)#4. `GROUP BY` (_Agrupación de Filas_)|GROUP BY's]] y funciones complejas en una vista, permitiendo a los usuarios consultar la vista como si fuera una tabla simple.
2. **Mejorar la seguridad:** Puedes restringir el acceso de los usuarios a solo ciertas columnas o filas de una tabla, o a solo las vistas necesarias, sin darles acceso directo a las tablas base.
3. **Proporcionar abstracción de datos:** Si la estructura de la tabla base cambia (ej., se añade o elimina una columna), solo necesitas modificar la vista, no las aplicaciones que la usan, siempre y cuando la vista siga exponiendo las columnas esperadas.
4. **Reutilización de consultas:** Una vez definida, una vista puede ser consultada por múltiples aplicaciones o usuarios, garantizando consistencia en los datos recuperados.

## Creación de una Vista (`CREATE VIEW`)

La sentencia [[DDL (Data Definition Language)#`CREATE`|CREATE]] `VIEW` se utiliza para definir una nueva vista en la base de datos.

```sql
CREATE VIEW nombre_vista AS
SELECT columna1, columna2, ...
FROM nombre_tabla
WHERE condicion;
```

## Modificación de una Vista (`ALTER VIEW` o `DROP VIEW` y `CREATE VIEW`)

Si necesitas cambiar la definición de una vista existente, la mayoría de los DBMS soportan [[DDL (Data Definition Language)#`ALTER`|ALTER]]  `VIEW`. Si tu DBMS no lo soporta directamente o prefieres un enfoque más seguro, puedes eliminarla y recrearla.

```sql
-- Para modificar una vista existente
ALTER VIEW nombre_vista AS
SELECT nueva_columna1, nueva_columna2, ...
FROM nueva_tabla
WHERE nueva_condicion;

-- Para eliminar y luego recrear (en caso de no soportar ALTER VIEW o preferir este método)
DROP VIEW nombre_vista;
CREATE VIEW nombre_vista AS
SELECT ...;
```

## Eliminación de una Vista (`DROP VIEW`)

La sentencia [[DDL (Data Definition Language)#`DROP`|DROP]] `VIEW` se utiliza para eliminar una vista de la base de datos.

```sql
DROP VIEW [IF EXISTS] nombre_vista;
```

El `IF EXISTS` es útil para evitar un error si la vista que intentas eliminar ya no existe.

## Vistas Actualizables (Updatable Views)

Aunque las vistas son "virtuales", algunas vistas pueden ser utilizadas para operaciones [[DML (Data Manipulation Language)#`INSERT`|INSERT]], [[DML (Data Manipulation Language)#`UPDATE`|UPDATE]] y [[DML (Data Manipulation Language)#`DELETE`|DELETE]] en las tablas base. Esto se conoce como vistas actualizables. Sin embargo, hay restricciones estrictas para que una vista sea actualizable:

- Debe basarse en una sola tabla (generalmente).
- No debe contener [[DQL (Data Query Language)#4. `GROUP BY` (_Agrupación de Filas_)|GROUP BY]], [[DQL (Data Query Language)#5. `HAVING` (_Filtrado de Grupos_)|HAVING]], `UNION` o funciones de agregación.
- Todas las columnas [[Constraints#`NOT NULL` - No Nulo|NOT NULL]] de la tabla base deben estar presentes en la vista.
- No debe contener `DISTINCT`.
- Las columnas calculadas o las uniones de múltiples tablas generalmente no son actualizables directamente (aunque algunos DBMS tienen excepciones o extensiones).

## Cuándo Usar Vistas

- **Simplificación de consultas:** Para usuarios que necesitan ver datos combinados o complejos sin escribir [[Joins]] o lógica complicada.
- **Seguridad:** Controlar qué datos (filas y columnas) puede ver cada usuario sin dar acceso completo a las tablas.
- **Consistencia:** Asegurar que todos los reportes o aplicaciones usen la misma definición de datos subyacentes.
- **Abstracción de esquema:** Ocultar la complejidad o los cambios en el esquema de la tabla subyacente.
