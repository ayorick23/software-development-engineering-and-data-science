---
Fecha de creación: 2025-08-15T17:58:00
Materia:
  - Base de Datos I (Relacionales)
Fecha de clase: 2025-08-15
---
[[Clase 04 - Introducción a Ambientes SQL|← Clase anterior]] | [[Clase 07 - Funciones en MySQL|Clase siguiente →]]

# MySQL Workbench

Como parte de la instalación de [[Clase 03 - Sentencias SQL en MySQL#¿Qué es MySQL?|MySQL]] encontrarás un entorno completo de diseño y manipulación de bases de datos que se llama **"MySQL Workbench"**, para acceder a él búscalo con ese nombre.

Para trabajar en el entorno nos conectaremos a la instancia local (servidor local en tu computadora) que dice "Local instance" en inglés. Si definiste una clave para la cuenta "root" tendrás que colocarla en este momento.

En el tablero de trabajo se encuentran en la parte media izquierda dos pestañas:
- Administration
- Schemas

"Schemas" muestra las bases de datos que trae MySQL una de sistema llamada ``sys`` y otras de ejemplo como son: ``sakila`` y ``world``.

## Creando bases de datos y tablas en MySQL Workbench

En el tablero de trabajo vamos a crear una nueva base de datos que se llame "Universidad"

1. Para ello colocaremos la sentencia [[Clase 04 - Introducción a Ambientes SQL#MySQL ``CREATE DATABASE``|CREATE DATABASE]] de la siguiente forma:

```sql
CREATE DATABASE Universidad;
```

Notarás que no se logra ver la nueva base de datos inmediatamente por lo que habrá que dar clic en el botón de refrescar a la derecha de "Schemas".

2. Una vez creada tendremos que ponerla en uso para poder crear tablas dentro de ella, eso se hace de la siguiente forma:

```sql
USE Universidad;
```

3. Crearemos en primera instancia la tabla "Carreras" de la siguiente forma:

```sql
CREATE TABLE Carreras (
	ID_carrera VARCHAR(5) PRIMARY KEY NOT NULL,
	nombre_carrera VARCHAR(40) NOT NULL
);
```

Una vez refrescamos nos aparecerá al lado izquierdo.

4. Ahora crearemos la tabla "Alumnos" de la siguiente forma:

```sql
CREATE TABLE Alumnos (
	ID_alumno INT PRIMARY KEY NOT NULL,
	nombre_alumno VARCHAR(40) NOT NULL,
	edad INT,
	ID_carrera VARCHAR(5),
	ciclo INT NOT NULL,
	FOREIGN KEY (ID_carrera) REFERENCES Carreras(ID_carrera)
);
```

Refrescamos nuevamente para comprobar la nueva tabla creada. Para la [[Clase 04 - Introducción a Ambientes SQL#MySQL ``FOREIGN KEY CONSTRAINT``|FOREIGN KEY]] creada notaremos la restricción siguiente que ayuda a cuidar la integridad referencial.

5. Después de esta introducción haremos varios ejercicios de inserción de datos como el siguiente:

```sql
INSERT INTO Carreras (ID_carrera, nombre_carrera)
VALUES
('11001', 'Ingeniería en Computación'),
('11002', 'Ingeniería en Robótica Aplicada');
```

6. Ejecutamos el [[Clase 02 - Estándar SQL#Comando ``SELECT``|comando SELECT]] para comprobar los datos insertados.

```sql
SELECT * FROM Carreras;
```

7. Podemos actualizar la información con [[DML (Data Manipulation Language)#`UPDATE`|UPDATE]]:

```sql
UPDATE Carreras
SET nombre_carrera = 'Ingeniería en Desarrollo de Software'
WHERE ID_carrera = '11001';
```