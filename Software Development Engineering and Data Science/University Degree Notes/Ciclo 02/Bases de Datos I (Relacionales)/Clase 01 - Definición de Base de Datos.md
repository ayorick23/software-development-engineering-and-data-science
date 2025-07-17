---
Fecha de creación: 2025-07-11T18:09:00
Materia:
  - Base de Datos I (Relacionales)
Fecha de clase: 2025-07-11
---
# Definición de Base de Datos

Una base de datos es una colección organizada de datos o información que se almacenara y se gestiona de manera electrónica. Las bases de datos se utilizan para almacenar grandes cantidades de información de manera eficiente y permiten a los usuarios acceder, modificar y consultar los datos de manera rápida y sencilla.

## Ejemplos

- Una base de datos de una tienda en línea que almacena información sobre productos, clientes y pedidos.
- Una base de datos de una red social que almacena publicaciones y perfiles de usuarios.
- Redis, que se utiliza para almacenar datos temporales en aplicaciones web para mejorar el rendimiento.

## ¿Qué beneficios nos dan las bases de datos?

- **Organización:** Facilitan la organización y gestión de grandes volúmenes de datos.
- **Acceso Rápido:** Permiten consultas rápidas y eficientes.
- **Integridad:** Ayudan a mantener la integridad de los datos mediante reglas y restricciones.
- **Seguridad:** Proporcionan mecanismos para proteger la información sensible.

## Evolución Histórica y Tipos de Bases de Datos

### Archivos planos (1960's)

Los primeros sistemas de almacenamiento de datos se basaban en archivos planos, donde los datos se almacenaban en archivos de texto simples.

```bash
Regular,Storage,Bierarchy Type,Attributes Reader
**Lorem Ipsum** is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industrys standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book.
```

### Sistemas de gestión de bases de datos jerárquicas (1960's - 1970's)

Los datos se organizaban en una estructura de árbol jerárquico, similar a un organigrama.

### Sistemas de gestión de bases de datos de red (1970's - 1980's)

Los datos se organizaban en una estructura de red, permitiendo relaciones más complejas. Los registros podían tener múltiples padres y múltiples hijos.

### Sistemas de gestión de bases de datos relacionales (1980's - 1990's)

Introducidos por Edgar F. Codd en 1970, los sistemas relacionales organizan los datos en tablas bidimensionales y utilizan SQL (Structured Query Language) para la gestión de datos.

### Bases de datos multidimensionales (1990's - a la fecha)

Concepto introducido por Ted Codd en 1993, se desarrollan como una respuesta a la creciente demanda de análisis de datos más complejos y detallados en el entorno empresarial.

### Sistemas de gestión de bases de datos orientadas a objetos (1990's - 2000's)

Combina las características de las bases de datos relacionales y la programación orientada a objetos.

### Sistemas de gestión de bases de datos NoSQL (2000's - presente)

Diseñados para manejar grandes volúmenes de datos no estructurados y datos distribuidos.

## Elementos Fundamentales de una Base de Datos Relacional

### Datos

Son hechos conocidos que pueden registrarse y que tienen un significado implícito. **Ejemplos:** nombres, números telefónicos y direcciones de personas que conocemos.

### Entidad

Es todo aquello de lo que interesa guardar datos. **Ejemplos:** Clientes, facturas, productos.

### Claves Primarias y Claves Foráneas

Cada entidad tiene una clave primaria o campo clave o llave que identifica de forma única al conjunto de datos. Cuando en una entidad figura la clave primaria de otra entidad, ésta se denomina clave foránea o clave ajena. Las entidades se **relacionan** entre sí a través de las claves foráneas.

### Metadatos

Son datos acerca de los datos presentes en la base de datos. **Ejemplo:** nombre, tipo de datos, longitud por mencionar algunos.

### Tabla

Es un conjunto de filas y columnas bajo un mismo nombre que representa el conjunto de valores almacenados para una serie de datos. Por ejemplo, la información de todos los clientes de una BD se almacenará en una tabla llamada CLIENTES.

### Campo (atributos)

Cada una de las columnas de una tabla. Identifica una familia de datos. Por ejemplo, el campo “FechaNacimiento” representa las fechas de nacimiento de todos los clientes que contiene una tabla CLIENTES.

### Registro

Corresponde a cada una de las filas de la tabla. También se llaman tuplas.

## Conceptos Claves

- **SQL:** Structured Query Language, el lenguaje de consultas estructuradas es un lenguaje de programación estandarizado que se utiliza para administrar bases de datos relacionales y realizar diversas operaciones con los datos que contienen.

> Se utilizará MySQL para esta materia.