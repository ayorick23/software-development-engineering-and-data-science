# Data Definition Language (DDL)

En [[Clase 02 - Estándar SQL#Definición de Estándar SQL|SQL]], DDL (_Data Definition Language o Lenguaje de Definición de Datos_) se refiere a un conjunto de comandos que se utilizan para definir la estructura de la base de datos. Estos comandos permiten **crear**, **modificar** y **eliminar** objetos de la base de datos, como tablas, vistas, índices, etc.

## `CREATE`

`CREATE`: Este comando se utiliza para crear nuevos objetos de base de datos, como tablas, vistas, índices, etc.

```sql
CREATE DATABASE nombre_base_datos;
```

```sql
CREATE TABLE nombre_tabla (
    columna1 tipo_datos [restricciones],
    columna2 tipo_datos [restricciones],
    ...
    PRIMARY KEY (columna_clave_primaria),
    FOREIGN KEY (columna_clave_foranea) REFERENCES tabla_referenciada(columna_referenciada)
);
```

## `ALTER`

`ALTER`: Este comando se utiliza para modificar la estructura de objetos de base de datos existentes. Se puede utilizar para añadir, eliminar o modificar columnas de una tabla, o para cambiar el nombre de una tabla o vista.

- Añadir una columna

  ```sql
  ALTER TABLE nombre_tabla
  ADD nombre_columna tipo_datos [restricciones];
  ```

- Eliminar una columna

  ```sql
  ALTER TABLE nombre_tabla
  DROP COLUMN nombre_columna;
  ```

- Modificar una columna

  ```sql
  ALTER TABLE nombre_tabla
  ALTER COLUMN nombre_columna TIPO_DATO_NUEVO; -- Sintaxis puede variar por DBMS
  ```

- Añadir una restricción

  ```sql
  ALTER TABLE nombre_tabla
  ADD CONSTRAINT nombre_restriccion TIPO_RESTRICCION (columna);
  ```

## `TRUNCATE`

`TRUNCATE`: Este comando se utiliza para eliminar todos los datos de una tabla, pero manteniendo la estructura de la tabla, mientras que `DROP` elimina toda la tabla, incluida su estructura.

```sql
TRUNCATE TABLE nombre_tabla;
```

## `DROP`

`DROP`: Este comando se utiliza para eliminar objetos de la base de datos, como tablas, vistas, índices, etc. Esto borrará la tabla y todos los datos que contiene de forma permanente.

```sql
DROP TABLE nombre_tabla;
```
