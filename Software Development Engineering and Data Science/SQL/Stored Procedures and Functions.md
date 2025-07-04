# Stored Procedures and Functions

Los **procedimientos almacenados** y las **funciones** son bloques de código SQL precompilados que se almacenan en la base de datos. Permiten encapsular lógica de negocio compleja, mejorar el rendimiento, aumentar la seguridad y facilitar la reutilización del código. Aunque ambos son programas almacenados, tienen diferencias clave en su propósito y comportamiento.

## Procedimientos Almacenados (_Stored Procedures_)

Un procedimiento almacenado es un conjunto de sentencias SQL que se compilan y almacenan en el servidor de la base de datos. Se ejecutan bajo demanda, lo que ofrece varias ventajas:

- **Rendimiento:** Al ser precompilados, se ejecutan más rápido que las consultas ad hoc. El plan de ejecución ya está creado.
- **Reusabilidad:** Una vez definido, un procedimiento puede ser llamado repetidamente desde aplicaciones, otras sentencias SQL o incluso otros procedimientos.
- **Seguridad:** Puedes conceder permisos para ejecutar un procedimiento sin dar acceso directo a las tablas subyacentes, lo que ayuda a restringir lo que los usuarios pueden ver o modificar.
- **Mantenimiento:** La lógica de negocio está centralizada en la base de datos, lo que facilita las actualizaciones y reduce la duplicación de código en la capa de la aplicación.
- **Parámetros:** Aceptan parámetros de entrada y pueden devolver parámetros de salida.
- **Control de Transacciones:** Pueden contener y gestionar transacciones ([[TCL (Transaction Control Language)#1. `BEGIN TRANSACTION` / `START TRANSACTION` - Iniciar una Transacción|BEGIN TRANSACTION]], [[TCL (Transaction Control Language)#2. `COMMIT` - Confirmar una Transacción|COMMIT]], [[TCL (Transaction Control Language)#3. `ROLLBACK` - Revertir una Transacción|ROLLBACK]]).
- **Sin valor de retorno obligatorio:** No tienen que devolver un valor, aunque pueden hacerlo mediante parámetros de salida o un código de estado.

```sql
CREATE PROCEDURE nombre_procedimiento
    (@parametro1 TIPO_DATO,
     @parametro2 TIPO_DATO OUTPUT -- Si es un parámetro de salida
    )
AS
BEGIN
    -- Cuerpo del procedimiento (sentencias SQL)
    SELECT * FROM Tabla WHERE Columna = @parametro1;
    -- Lógica de negocio
END;
```

**NOTA:** La sintaxis específica de los parámetros y la delimitación del cuerpo puede variar ligeramente entre DBMS (ej., MySQL usa `DELIMITER //`, `CREATE PROCEDURE ... BEGIN ... END //`).

## Funciones (_Functions_)

Las **funciones** en SQL son similares a los procedimientos almacenados en que son bloques de código precompilados. Sin embargo, tienen diferencias fundamentales:

- **Siempre devuelven un valor:** Una función siempre debe devolver un único valor escalar (o una tabla, en el caso de las funciones con valor de tabla).
- **Pueden ser utilizadas en expresiones SQL:** Puedes usar funciones directamente en sentencias [[DML (Data Manipulation Language)#`SELECT`|SELECT]], [[DQL (Data Query Language)#3. `WHERE` (_Filtrado de Filas_)|WHERE]], [[DQL (Data Query Language)#5. `HAVING` (_Filtrado de Grupos_)|HAVING]], [[DQL (Data Query Language)#6. `ORDER BY` (_Ordenación de Resultados_)|ORDER BY]], etc., al igual que las funciones de agregación ([[Aggregate Functions#`SUM()` - Sumar Valores|SUM()]], [[Aggregate Functions#`AVG()` - Calcular el Promedio|AVG()]]).
- **Restricciones:** Generalmente, las funciones no pueden realizar operaciones [[DML (Data Manipulation Language)|DML]] (`INSERT`, `UPDATE`, `DELETE`) directamente en las tablas (depende del DBMS y el tipo de función). Son principalmente para cálculos y consultas.
- **Parámetros:** Solo aceptan parámetros de entrada.

### Tipos de Funciones

- **Funciones Escalares (Scalar Functions):** Devuelven un único valor para cada fila de entrada.
- **Funciones con Valor de Tabla (Table-Valued Functions - TVFs):** Devuelven una tabla como resultado. (Más común en SQL Server, menos directa en otros DBMS).

```sql
CREATE FUNCTION nombre_funcion
    (@parametro1 TIPO_DATO)
RETURNS TIPO_DATO_RETORNO
AS
BEGIN
    DECLARE @resultado TIPO_DATO_RETORNO;
    -- Lógica de cálculo
    SET @resultado = ...;
    RETURN @resultado;
END;
```

**NOTA:** Al igual que los procedimientos, la sintaxis puede variar entre DBMS.

## Diferencias Clave entre Procedimientos Almacenados y Funciones

| Característica       | Procedimientos Almacenados                                                                                                  | Funciones                                                              |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Valor de Retorno** | No tienen un valor de retorno obligatorio. Pueden devolver múltiples resultados (conjuntos de filas, parámetros de salida). | Siempre devuelven un único valor escalar o una tabla.                  |
| **Uso en SQL**       | Se ejecutan con `EXEC` o `CALL`. No pueden usarse directamente en `SELECT`, `WHERE`, etc.                                   | Pueden usarse en sentencias `SELECT`, `WHERE`, `HAVING`, etc.          |
| **Operaciones DML**  | Pueden ejecutar `INSERT`, `UPDATE`, `DELETE`.                                                                               | Generalmente no pueden ejecutar DML directo (depende del tipo y DBMS). |
| **Transacciones**    | Pueden gestionar transacciones (`BEGIN`, `COMMIT`, `ROLLBACK`).                                                             | No pueden gestionar sus propias transacciones explícitamente.          |
| **Parámetros**       | Aceptan parámetros de entrada (`IN`) y salida (`OUT` / `OUTPUT`).                                                           | Solo aceptan parámetros de entrada (`IN`).                             |

## Cuándo Usar Cada Uno

**Usa Procedimientos Almacenados cuando:**

- Necesites realizar una serie de operaciones DML que deben ser atómicas (dentro de una transacción).
- Necesites encapsular lógica de negocio compleja que involucre múltiples pasos y manipulación de datos.
- Quieras mejorar la seguridad, concediendo solo el permiso de ejecución sin acceso directo a las tablas.
- Necesites retornar múltiples conjuntos de resultados o valores a través de parámetros de salida.
- Busques una mejora en el rendimiento debido a la precompilación.

**Usa Funciones cuando:**

- Necesites realizar cálculos y devolver un único valor escalar.
- Quieras reutilizar expresiones complejas dentro de consultas `SELECT`, `WHERE`, `HAVING`, etc.
- Necesites generar un conjunto de datos tabular basado en ciertos parámetros (funciones con valor de tabla).
- Tu lógica no requiere modificar los datos de la base de datos (no DML).
