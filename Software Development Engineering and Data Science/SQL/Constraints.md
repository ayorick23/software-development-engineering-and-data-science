# Constraints

En SQL, las **restricciones** (_constraints_) son reglas que se aplican a los datos de una tabla para asegurar la integridad y la validez de la información almacenada. Estas reglas definen qué tipo de datos se pueden insertar, actualizar o eliminar en las columnas de una tabla. Las restricciones ayudan a prevenir errores y a mantener la consistencia de la base de datos.

Puedes definir restricciones al momento de crear una tabla ([[DDL (Data Definition Language)#`CREATE`|CREATE]] `TABLE`) o modificarlas en una tabla existente ([[DDL (Data Definition Language)#`ALTER`|ALTER]] `TABLE`).

## `NOT NULL` - No Nulo

La restricción `NOT NULL` asegura que una columna no puede contener valores nulos (vacíos). Cada fila debe tener un valor explícito para esa columna.

```sql
CREATE TABLE nombre_tabla (
    columna1 tipo_de_datos NOT NULL,
    columna2 tipo_de_datos,
    ...
);
```

Aplicar la restricción `NOT NULL` a una columna existente:

```sql
ALTER TABLE nombre_tabla
ALTER COLUMN nombre_columna tipo_de_datos NOT NULL;
```

## `UNIQUE` - Valores Únicos

La restricción `UNIQUE` garantiza que todos los valores en una columna (o un conjunto de columnas) sean diferentes. Aunque una tabla puede tener varias restricciones `UNIQUE`, solo puede tener una `PRIMARY KEY`. Una columna `UNIQUE` puede contener valores `NULL`, pero solo una vez (en la mayoría de los DBMS).

```sql
CREATE TABLE nombre_tabla (
    columna1 tipo_de_datos UNIQUE,
    columna2 tipo_de_datos,
    ...
);
```

Aplicar la restricción `UNIQUE` a una tabla existente:

```sql
ALTER TABLE nombre_tabla
ADD CONSTRAINT nombre_restriccion UNIQUE (columna1, columna2, ...);
```

## `PRIMARY KEY` - Clave Primaria

La restricción `PRIMARY KEY` (clave primaria) identifica de forma única cada fila en una tabla. Una tabla puede tener solo una clave primaria, y esta debe contener valores únicos y no nulos. Una clave primaria puede ser una sola columna o una combinación de varias columnas (clave primaria compuesta).

```sql
CREATE TABLE nombre_tabla (
    columna1 tipo_dato PRIMARY KEY,
    columna2 tipo_dato,
    ...
);
```

Para llaves primarias compuestas:

```sql
CREATE TABLE nombre_tabla (
    columna1 tipo_dato,
    columna2 tipo_dato,
    ...,
    PRIMARY KEY (columna1, columna2, ...)
);
```

Añadir una `PRIMARY KEY` a una tabla existente:

```sql
ALTER TABLE nombre_tabla
ADD CONSTRAINT nombre_restriccion PRIMARY KEY (columna1, columna2, ...);
```

## `FOREIGN KEY` - Clave Foránea

La restricción `FOREIGN KEY` (clave foránea) establece un vínculo entre dos tablas. Una clave foránea en una tabla (la tabla "hija") apunta a la clave primaria (o a una clave `UNIQUE`) en otra tabla (la tabla "padre"). Asegura la integridad referencial, es decir, que no puedas tener registros en la tabla hija que apunten a registros inexistentes en la tabla padre.

```sql
CREATE TABLE nombre_tabla (
    columna1 tipo_dato,
    columna2 tipo_dato,
    ...
    CONSTRAINT nombre_restriccion_fk FOREIGN KEY (columna_fk) REFERENCES tabla_referenciada(columna_referencia)
);
```

Añadir una `FOREIGN KEY` a una tabla existente:

```sql
ALTER TABLE nombre_tabla
ADD CONSTRAINT nombre_restriccion_fk FOREIGN KEY (columna_fk) REFERENCES tabla_referenciada(columna_referencia);
```

## `DEFAULT` - Valor por Defecto

La restricción `DEFAULT` proporciona un valor predeterminado para una columna cuando no se especifica explícitamente un valor durante una operación [[DML (Data Manipulation Language)#`INSERT`|INSERT]].

```sql
CREATE TABLE nombre_tabla (
    columna1 tipo_dato DEFAULT valor_por_defecto,
    columna2 tipo_dato,
    ...
);
```

## `CHECK` - Validación de Valores

La restricción `CHECK` se utiliza para limitar el rango de valores que se pueden colocar en una columna. Puedes definir una expresión booleana que debe ser verdadera para que la inserción o actualización de la fila sea exitosa.

```sql
CREATE TABLE nombre_tabla (
    columna1 tipo_datos,
    columna2 tipo_datos,
    ...
    CONSTRAINT nombre_restriccion CHECK (condición)
);
```

Agregar la restricción `CHECK` a una tabla existente:

```sql
ALTER TABLE nombre_tabla
ADD CONSTRAINT nombre_restriccion CHECK (condición);
```
