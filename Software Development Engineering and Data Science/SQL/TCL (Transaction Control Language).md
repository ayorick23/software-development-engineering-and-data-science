# Transaction Control Language (TCL)

El **TCL** (_Transaction Control Language o Lenguaje de Control de Transacciones_) es una parte fundamental de SQL que te permite gestionar y controlar las transacciones dentro de una base de datos. Una transacción es una secuencia lógica de una o más operaciones SQL (como [[DML (Data Manipulation Language)#`INSERT`|INSERT]], [[DML (Data Manipulation Language)#`UPDATE`|UPDATE]], [[DML (Data Manipulation Language)#`DELETE`|DELETE]]) que se ejecutan como una única unidad de trabajo. Esto significa que o bien todas las operaciones dentro de la transacción se completan con éxito (se "confirman"), o bien ninguna de ellas lo hace (se "revierte" o deshace).

El objetivo principal de TCL es garantizar la integridad y consistencia de los datos, especialmente en entornos multiusuario o en situaciones donde múltiples operaciones deben ocurrir atómicamente.

Los comandos clave de TCL son:

## 1. `BEGIN TRANSACTION` / `START TRANSACTION` - Iniciar una Transacción

Este comando marca el punto de inicio de una transacción. Todas las sentencias DML que se ejecuten después de este comando y antes de un `COMMIT` o `ROLLBACK` se considerarán parte de la misma transacción.

```sql
BEGIN TRANSACTION; -- Común en SQL Server
-- O
START TRANSACTION; -- Común en MySQL, PostgreSQL, Oracle
```

## 2. `COMMIT` - Confirmar una Transacción

El comando `COMMIT` hace que todos los cambios realizados durante la transacción sean permanentes en la base de datos. Una vez que una transacción se confirma, sus cambios son visibles para otros usuarios y se guardan de forma duradera.

```sql
COMMIT;
```

## 3. `ROLLBACK` - Revertir una Transacción

El comando `ROLLBACK` deshace todos los cambios realizados durante la transacción desde el último `BEGIN TRANSACTION` o `SAVEPOINT`. Esto es crucial si ocurre un error, si una de las operaciones falla o si decides que no quieres que los cambios sean permanentes. La base de datos vuelve a su estado anterior al inicio de la transacción.

```sql
ROLLBACK;
```

## 4. `SAVEPOINT` - Puntos de Guardado

Un `SAVEPOINT` (punto de guardado) es un marcador dentro de una transacción al que puedes revertir (con `ROLLBACK TO SAVEPOINT`) sin deshacer toda la transacción. Esto es útil para transacciones largas o complejas donde quieres poder deshacer solo una parte de las operaciones.

```sql
SAVEPOINT nombre_savepoint;
ROLLBACK TO SAVEPOINT nombre_savepoint;
```

## 5. `SET TRANSACTION` (Nivel de Aislamiento, Solo Lectura)

Este comando se utiliza para establecer características de la transacción actual, como el nivel de aislamiento o si es una transacción de solo lectura.

- **Niveles de Aislamiento (ACID - Aislamiento):** Controlan cómo una transacción puede afectar a otras transacciones simultáneas y viceversa. Los niveles comunes son:
  - `READ UNCOMMITTED`: Permite leer cambios no confirmados por otras transacciones (lecturas sucias).
  - `READ COMMITTED`: Solo permite leer cambios confirmados por otras transacciones.
  - `REPEATABLE READ`: Garantiza que si lees una fila, no cambiará dentro de la misma transacción (pero nuevas filas pueden aparecer - "phantom reads").
  - `SERIALIZABLE`: El nivel más alto de aislamiento. Garantiza que las transacciones concurrentes produzcan el mismo resultado que si se ejecutaran secuencialmente.

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- Luego, BEGIN TRANSACTION
```

## Propiedades ACID de las Transacciones

El concepto de transacción es fundamental para garantizar la confiabilidad en los sistemas de bases de datos. Esto se logra adhiriéndose a las propiedades **ACID**:

- **A - Atomicidad (Atomicity):** Una transacción es una unidad de trabajo indivisible. O todas las operaciones dentro de ella se completan con éxito, o ninguna lo hace. Si falla alguna parte, toda la transacción se revierte al estado original (como si nunca hubiera comenzado). El `ROLLBACK` asegura la atomicidad.

- **C - Consistencia (Consistency):** Una transacción lleva la base de datos de un estado válido a otro estado válido. Esto significa que la base de datos debe cumplir con todas las reglas y restricciones (como [[Constraints#`PRIMARY KEY` - Clave Primaria|PRIMARY KEY's]], [[Constraints#`FOREIGN KEY` - Clave Foránea|FOREIGN KEY's]], [[Constraints#`CHECK` - Validación de Valores|CHECK]]) tanto antes como después de la transacción. Si una transacción intenta violar estas reglas, se revierte.

- **I - Aislamiento (Isolation):** Las transacciones concurrentes (que se ejecutan al mismo tiempo) no deben interferir entre sí. Cada transacción debe parecer que se está ejecutando de forma independiente y serial. Los niveles de aislamiento (`SET TRANSACTION ISOLATION LEVEL`) controlan el grado de aislamiento.

- **D - Durabilidad (Durability):** Una vez que una transacción se ha confirmado (`COMMIT`), sus cambios son permanentes y persisten incluso en caso de fallos del sistema (como un corte de energía). Los cambios se escriben de forma segura en el almacenamiento persistente.

### Modo de Auto-Commit (Autocommit)

La mayoría de los DBMS operan en modo auto-commit por defecto. Esto significa que cada sentencia [[DML (Data Manipulation Language)|DML]] (`INSERT`, `UPDATE`, `DELETE`) se trata automáticamente como su propia transacción y se confirma inmediatamente después de su ejecución.

Para trabajar explícitamente con transacciones, generalmente necesitas deshabilitar el auto-commit (o el `BEGIN TRANSACTION` lo deshabilitará implícitamente hasta que se realice un `COMMIT` o `ROLLBACK`).

**Cómo verificar/cambiar auto-commit (ejemplos, varía según DBMS):**

- **SQL Server:** Generalmente, está activado por defecto. `BEGIN TRANSACTION` es la forma explícita de iniciar una.
- **MySQL:** `SET autocommit = 0;` (para deshabilitar) o `SET autocommit = 1;` (para habilitar).
- **PostgreSQL:** Por defecto, cada sentencia DML es una transacción. Puedes usar `BEGIN;` o `START TRANSACTION;` para agrupar sentencias.
