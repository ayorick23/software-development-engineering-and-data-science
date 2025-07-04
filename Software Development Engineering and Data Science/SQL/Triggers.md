# Triggers

Los **triggers** (o disparadores) en SQL son un tipo especial de procedimiento almacenado que se ejecuta automáticamente (se "dispara") en respuesta a eventos específicos de modificación de datos en una tabla. Estos eventos suelen ser operaciones [[DML (Data Manipulation Language)|DML]] como `INSERT`, `UPDATE` o `DELETE`. Los triggers son una herramienta poderosa para hacer cumplir reglas de negocio complejas, mantener la integridad de los datos, auditar cambios o automatizar acciones en la base de datos.

## ¿Cómo Funcionan los Triggers?

Un trigger se asocia a una tabla y a un tipo de evento. Cuando el evento ocurre en esa tabla, el trigger se activa. El código dentro del trigger se ejecuta automáticamente y puede interactuar con los datos de la tabla que disparó el evento, o incluso con otras tablas.

## Tipos de Triggers (según el momento de activación)

La mayoría de los sistemas de bases de datos soportan triggers que se disparan en diferentes momentos con respecto a la operación DML:

1. `BEFORE` Triggers (Antes de la operación):

   - Se ejecutan antes de que la operación DML ([[DML (Data Manipulation Language)#`INSERT`|INSERT]], [[DML (Data Manipulation Language)#`UPDATE`|UPDATE]], [[DML (Data Manipulation Language)#`DELETE`|DELETE]]) se realice en la tabla.
   - Son útiles para validar datos, modificar valores antes de la inserción/actualización, o prevenir la operación si no cumple ciertas condiciones.
   - Puedes modificar las filas que están a punto de ser insertadas o actualizadas.

2. `AFTER` Triggers (Después de la operación):
   - Se ejecutan después de que la operación DML se ha completado en la tabla.
   - Son ideales para auditar cambios, mantener la consistencia en otras tablas, o realizar acciones en cadena.
   - No puedes modificar las filas que ya han sido insertadas/actualizadas/eliminadas directamente dentro del trigger para la misma operación, pero sí puedes interactuar con otras tablas o las "versiones" de las filas que se modificaron.

## Cláusulas y Conceptos Importantes

### `FOR EACH ROW` vs. `FOR EACH STATEMENT` (o sus equivalentes)

- **`FOR EACH ROW` (Row-Level Trigger):** El trigger se dispara una vez por cada fila afectada por la sentencia DML. Es el tipo más común.
- **`FOR EACH STATEMENT` (Statement-Level Trigger):** El trigger se dispara solo una vez por cada sentencia DML, independientemente de cuántas filas sean afectadas. (Común en Oracle; en SQL Server, los triggers son por defecto a nivel de sentencia).

### Tablas/Pseudotablas Mágicas (para acceder a datos modificados)

- Los DBMS proporcionan mecanismos para acceder a los datos que están siendo insertados, actualizados o eliminados dentro del trigger. Los nombres varían:
  - **SQL Server:** `INSERTED` (para `INSERT` y `UPDATE`) y `DELETED` (para `UPDATE` y `DELETE`). Estas son tablas temporales con la estructura de la tabla base.
  - **MySQL:** `NEW` (para `INSERT` y `UPDATE`) y `OLD` (para `UPDATE` y `DELETE`). Son variables que representan la fila afectada. NEW.columna es el valor después del cambio; OLD.columna es el valor antes del cambio.
  - **PostgreSQL/Oracle:** `NEW` y `OLD` también son usados, a menudo precedidos por `:` (ej., `:NEW.columna`).

```sql
CREATE TRIGGER nombre_trigger
[BEFORE | AFTER] [INSERT | UPDATE | DELETE] -- Momento y evento
ON nombre_tabla
[FOR EACH ROW] -- Opciones de nivel (si aplica)
[WHEN (condicion)] -- Condición opcional para disparar el trigger (MySQL, PostgreSQL, Oracle)
EXECUTE PROCEDURE nombre_funcion_o_logica(); -- Sintaxis varía mucho
-- O en otros DBMS, el cuerpo del trigger va directamente:
AS
BEGIN
    -- Lógica del trigger usando NEW/OLD o INSERTED/DELETED
END;
```

## Eliminación de Triggers (`DROP TRIGGER`)

Puedes eliminar un trigger si ya no es necesario con la siguiente sintaxis:

```sql
DROP TRIGGER nombre_trigger ON nombre_tabla; -- SQL Server
-- O
DROP TRIGGER [IF EXISTS] nombre_trigger; -- MySQL, PostgreSQL
```

## Consideraciones al Usar Triggers

- **Complejidad:** Un abuso de triggers puede hacer que el sistema sea difícil de entender y depurar, ya que la lógica se vuelve implícita en lugar de explícita en las aplicaciones.
- **Rendimiento:** Demasiados triggers o triggers con lógica compleja pueden afectar negativamente el rendimiento de las operaciones DML.
- **Debugging:** Depurar triggers puede ser más desafiante que depurar código de aplicación estándar.
- **Orden de Ejecución:** Si tienes múltiples triggers del mismo tipo (ej., varios `AFTER INSERT`), el orden de ejecución no siempre está garantizado (o puede ser DBMS-específico).
- **Bucle Infinito:** Es posible crear triggers que se disparen mutuamente en un bucle infinito, lo que puede bloquear el sistema.
