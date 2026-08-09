---
Fecha de creación: 2025-10-24T15:50:00
Materia:
  - Base de Datos I (Relacionales)
Fecha de clase: 2025-10-24
---
[[Clase 14 - Subconsultas en MySQL|← Clase anterior]]

# Procedimientos Almacenados en MySQL
(ver [[Stored Procedures and Functions#Procedimientos Almacenados (_Stored Procedures_)|Procedimientos Almacenados]])

Un **procedimiento almacenado** (*Stored Procedure*) es un conjunto de instrucciones SQL que se **almacenan en el servidor** y pueden ser **ejecutadas** posteriormente por los usuarios o por otros programas.

Se utilizan para:

- Automatizar tareas repetitivas.
- Reducir la carga en el cliente.
- Mejorar el rendimiento en operaciones complejas.
- Reforzar la seguridad (control de acceso a datos).
- Estandarizar la lógica del negocio dentro de la base de datos.

## Características Principales

- Se crean **una sola vez** y se ejecutan muchas veces.
- Aceptan **parámetros de entrada y salida**.
- Pueden incluir **estructuras de control** como `IF`, `CASE`, `WHILE`, `LOOP`, etc.
- Se ejecutan con el comando `CALL`.
- Pueden devolver resultados mediante `SELECT` o parámetros `OUT`.

## Sintaxis Básica

```sql
DELIMITER $$

CREATE PROCEDURE nombre_procedimiento(
    [IN|OUT|INOUT] parametro1 tipo,
    [IN|OUT|INOUT] parametro2 tipo
)
BEGIN
    -- Instrucciones SQL
END$$

DELIMITER ;
```

### Explicación:

- `DELIMITER` cambia el delimitador temporalmente para que MySQL no confunda el `;` dentro del bloque `BEGIN ... END`.
- `IN` → parámetro de entrada (valor que recibe el procedimiento).
- `OUT` → parámetro de salida (valor que devuelve el procedimiento).
- `INOUT` → parámetro de entrada y salida.
- `BEGIN ... END` delimita el bloque de código.

**Ejemplo:** Procedimiento sencillo sin parámetros

```sql
DELIMITER $$

CREATE PROCEDURE mostrar_clientes()
BEGIN
    SELECT * FROM clientes;
END$$

DELIMITER ;

-- Ejecución:
CALL mostrar_clientes();

```

**Ejemplo:** Procedimiento con parámetro de entrada (``IN``)

```sql
DELIMITER $$

CREATE PROCEDURE buscar_cliente(IN id INT)
BEGIN
    SELECT nombre, email, telefono
    FROM clientes
    WHERE id_cliente = id;
END$$

DELIMITER ;

-- Ejecución:
CALL buscar_cliente(3);
```

**Ejemplo:** Procedimiento con parámetro de salida (``OUT``)

```sql
DELIMITER $$

CREATE PROCEDURE contar_clientes(OUT total INT)
BEGIN
    SELECT COUNT(*) INTO total FROM clientes;
END$$

DELIMITER ;

-- Ejecución:
CALL contar_clientes(@resultado);
SELECT @resultado AS 'Total de clientes';
```

>**Nota:**  
>`INTO` se usa para guardar el resultado de una consulta en una variable.

## Estructuras de Control dentro de un Procedimiento

### IF-THEN-ELSE

```sql
IF condicion THEN
    -- instrucciones
ELSE
    -- instrucciones alternativas
END IF;

```

**Ejemplo:**

```sql
DELIMITER $$

CREATE PROCEDURE verificar_stock(IN producto_id INT)
BEGIN
    DECLARE cantidad INT;

    SELECT stock INTO cantidad FROM productos WHERE id_producto = producto_id;

    IF cantidad > 0 THEN
        SELECT 'Disponible' AS estado;
    ELSE
        SELECT 'Agotado' AS estado;
    END IF;
END$$

DELIMITER ;
```

### WHILE-DO

```sql
WHILE condicion DO
    -- instrucciones
END WHILE;
```

**Ejemplo:**

```sql
DELIMITER $$

CREATE PROCEDURE contar_hasta(IN numero INT)
BEGIN
    DECLARE contador INT DEFAULT 1;
    WHILE contador <= numero DO
        SELECT CONCAT('Contador: ', contador) AS valor;
        SET contador = contador + 1;
    END WHILE;
END$$

DELIMITER ;
```

## Modificación y Eliminación de Procedimientos

No se puede modificar directamente; se debe eliminar y crear de nuevo.

```sql
DROP PROCEDURE IF EXISTS nombre_procedimiento;
```

Luego crear el nuevo procedimiento con los cambios.

## Manejo de Errores con ``HANDLER``

Puedes capturar errores usando **HANDLERs**.

```sql
DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
BEGIN
    SELECT 'Ha ocurrido un error.' AS mensaje;
END;
```

**Ejemplo:**

```sql
DELIMITER $$

CREATE PROCEDURE insertar_cliente(
    IN nombre_cliente VARCHAR(100),
    IN correo VARCHAR(100)
)
BEGIN
    DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
    BEGIN
        SELECT 'Error al insertar el cliente.' AS mensaje;
    END;

    INSERT INTO clientes (nombre, email) VALUES (nombre_cliente, correo);
    SELECT 'Cliente insertado correctamente.' AS mensaje;
END$$

DELIMITER ;
```

## Consulta de Procedimientos Existentes

```sql
SHOW PROCEDURE STATUS;
```

Para ver el código fuente de un procedimiento:

```sql
SHOW CREATE PROCEDURE nombre_procedimiento;
```

## Buenas Prácticas

- Usa nombres descriptivos para los procedimientos y parámetros.  
- Siempre incluye `DROP PROCEDURE IF EXISTS` antes de crear uno nuevo.  
- Agrega comentarios en el código para mantener claridad.  
- Evita lógica de negocio muy compleja dentro del procedimiento (divide en varios).  
- Controla los errores y validaciones antes de insertar o actualizar datos.
