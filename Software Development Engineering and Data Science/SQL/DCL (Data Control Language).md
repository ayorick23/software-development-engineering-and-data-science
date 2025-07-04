# Data Control Language

El **DCL** (_Data Control Language o Lenguaje de Control de Datos_) en SQL se ocupa de los permisos y derechos de los usuarios en la base de datos. Permite a los administradores de la base de datos controlar quién puede acceder a qué objetos de la base de datos (tablas, [[Views|vistas]], [[Stored Procedures and Functions#Funciones (_Functions_)|funciones]], etc.) y qué acciones pueden realizar sobre ellos (leer, escribir, modificar, etc.). El DCL es fundamental para la seguridad y la gestión de la privacidad de los datos.

Los dos comandos principales de DCL son `GRANT` y `REVOKE`.

## 1. `GRANT` - Conceder Permisos

El comando `GRANT` se utiliza para otorgar privilegios (permisos) a usuarios o roles de la base de datos. Estos privilegios definen qué operaciones SQL específicas pueden realizar en objetos de la base de datos.

```sql
GRANT privilegio1, privilegio2, ...
ON objeto_base_de_datos -- (tabla, vista, función, procedimiento, etc.)
TO usuario1, usuario2, ... -- O TO PUBLIC (todos los usuarios)
[WITH GRANT OPTION];
```

### Privilegios Comunes

- [[DML (Data Manipulation Language)#`SELECT`|SELECT]]: Permiso para leer datos de una tabla o vista.
- [[DML (Data Manipulation Language)#`INSERT`|INSERT]]: Permiso para añadir nuevas filas a una tabla.
- [[DML (Data Manipulation Language)#`UPDATE`|UPDATE]]: Permiso para modificar filas existentes en una tabla. Se puede especificar a qué columnas.
- [[DML (Data Manipulation Language)#`DELETE`|DELETE]]: Permiso para eliminar filas de una tabla.
- `ALL PRIVILEGES`: Otorga todos los privilegios disponibles en un objeto.
- [[Constraints#`FOREIGN KEY` - Clave Foránea|REFERENCES]]: Permiso para crear una clave foránea que referencie la tabla (esencial para la integridad referencial).
- [[DDL (Data Definition Language)#`CREATE`|CREATE]] (en bases de datos/esquemas): Permiso para crear objetos dentro de la base de datos o esquema.
- [[DDL (Data Definition Language)#`ALTER`|ALTER]]: Permiso para modificar la estructura de un objeto (ej., añadir/eliminar columnas).
- [[DDL (Data Definition Language)#`DROP`|DROP]]: Permiso para eliminar un objeto.

### Cláusula Opcional `WITH GRANT OPTION`

Si se incluye `WITH GRANT OPTION`, el usuario al que se le concede el permiso puede, a su vez, otorgar ese mismo permiso a otros usuarios. Es una opción que debe usarse con precaución, ya que delega la capacidad de gestionar permisos.

## 2. `REVOKE` - Revocar Permisos

El comando `REVOKE` se utiliza para eliminar privilegios que previamente fueron concedidos a usuarios o roles.

```sql
REVOKE privilegio1, privilegio2, ...
ON objeto_base_de_datos
FROM usuario1, usuario2, ... -- O FROM PUBLIC
[CASCADE | RESTRICT]; -- Comportamiento si los privilegios fueron otorgados a otros
```

### Cláusulas Opcionales `CASCADE` y `RESTRICT`

Estas cláusulas determinan el comportamiento si el privilegio que se revoca fue concedido a otros usuarios por el usuario al que se le revoca el permiso (`WITH GRANT OPTION).

- `CASCADE`: Revoca el privilegio del usuario especificado y también de cualquier otro usuario al que el privilegio haya sido concedido por este usuario. Es una revocación en cascada.
- `RESTRICT` (Por defecto en muchos sistemas): Impide la revocación del privilegio si el usuario ha utilizado ese privilegio para concederlo a otros. La operación `REVOKE` fallará.

## Roles

En sistemas de bases de datos más grandes, en lugar de conceder permisos directamente a usuarios individuales, es común usar roles. Un rol es un conjunto de privilegios nombrados. Los privilegios se conceden a un rol, y luego los roles se conceden a los usuarios. Esto simplifica enormemente la gestión de permisos, ya que puedes asignar o revocar un rol a un usuario, en lugar de gestionar múltiples permisos individuales.

```sql
-- Crear un rol
CREATE ROLE nombre_rol;

-- Conceder privilegios al rol
GRANT SELECT ON Productos TO nombre_rol;
GRANT INSERT, UPDATE ON Pedidos TO nombre_rol;

-- Conceder el rol a un usuario
GRANT nombre_rol TO usuario1, usuario2;

-- Revocar el rol de un usuario
REVOKE nombre_rol FROM usuario1;
```

## Consideraciones de Seguridad

- **Principio de Privilegio Mínimo:** Concede solo los permisos estrictamente necesarios para que un usuario o aplicación realice su tarea. No otorgues `ALL PRIVILEGES` o permisos excesivos a menos que sea absolutamente requerido.
- **Roles vs. Usuarios:** Prefiere el uso de roles para una gestión de permisos más escalable y sencilla.
- **Auditoría:** En entornos de producción, es crucial auditar los permisos de los usuarios y las actividades que realizan para asegurar el cumplimiento y detectar posibles brechas de seguridad.
- **Contraseñas Robustas:** Asegúrate de que los usuarios utilicen contraseñas fuertes y que se cambien regularmente.
