---
Fecha de creación: 2026-03-14T14:09:00
Materia:
  - Base de Datos II (No Relacionales)
Fecha de clase: 2026-03-14
---
# Diseño de Esquemas en Bases de Datos

El **diseño de esquemas** es una etapa fundamental en el desarrollo de bases de datos. Define cómo se organizarán los datos, cómo se relacionarán entre sí y cómo serán consultados por las aplicaciones.

En las bases de datos relacionales, el esquema suele ser **estricto y altamente estructurado**, mientras que en bases de datos NoSQL como MongoDB el enfoque es **más flexible y adaptado a las necesidades de la aplicación**.

Comprender estas diferencias es clave para diseñar sistemas eficientes, escalables y fáciles de mantener.

## ¿Qué es el esquema de una base de datos?

### Concepto tradicional (IBM)

En el contexto de bases de datos relacionales, un **esquema de base de datos** define la estructura lógica de los datos dentro del sistema.

Incluye elementos como:

- Nombres de las tablas
- Campos o columnas
- Tipos de datos
- Relaciones entre tablas
- Restricciones de integridad

El esquema funciona como un **modelo que describe cómo se organizan los datos** y cómo pueden interactuar entre sí.

### Funciones principales del esquema

#### Acceso y seguridad

El esquema ayuda a controlar quién puede acceder a los datos y qué operaciones puede realizar.

#### Organización y comunicación

Define una estructura clara que permite a desarrolladores, analistas y administradores entender cómo está organizada la base de datos.

#### Integridad de los datos

Las restricciones del esquema ayudan a garantizar que los datos almacenados sean válidos y consistentes.

## Desventajas de los Esquemas Rígidos (SQL)

Aunque los esquemas relacionales son muy útiles para mantener consistencia, también presentan algunas limitaciones.

Entre las principales desventajas se encuentran:

- **Rigidez estructural:** cualquier cambio requiere modificar el esquema.
- **Migraciones complejas:** agregar columnas o modificar estructuras puede requerir migraciones de datos.
- **Mayor tiempo de desarrollo:** cambios en el modelo pueden afectar muchas partes de la aplicación.
- **Dificultad para manejar datos no estructurados o semiestructurados.**

Estas limitaciones fueron una de las razones por las que surgieron las bases de datos NoSQL.

## ¿MongoDB necesita esquemas?

La respuesta es **sí y no al mismo tiempo**.

MongoDB es una base de datos **schema-less**, lo que significa que **no requiere un esquema fijo obligatorio como en SQL**.

Sin embargo, esto **no significa que no se deba diseñar un esquema**.

En la práctica, las aplicaciones que utilizan MongoDB **sí necesitan un diseño estructurado**, aunque este sea más flexible.

Por lo tanto:

- MongoDB **permite esquemas dinámicos**.
- Pero **el diseño del esquema sigue siendo importante** para el rendimiento y la organización de los datos.

## Diseño de Esquemas en Bases de Datos NoSQL (MongoDB)

En MongoDB los datos se almacenan en **documentos BSON**, que son similares a objetos JSON.

Un documento puede contener:

- Campos simples
- Objetos anidados
- Arreglos

Ejemplo:

```javascript
{
	nombre: "Ana López",
	correo: "ana@empresa.com",
	pedidos: [
		{ producto: "Laptop", cantidad: 1 },
		{ producto: "Mouse", cantidad: 2 }
	]
}
```

Esta estructura permite almacenar datos relacionados **dentro de un mismo documento**.

## Esquemas Flexibles: Ventaja NoSQL

Una de las principales características de MongoDB es su **flexibilidad estructural**.

### Estructura variable

Cada documento puede tener campos diferentes.

Ejemplo:

Documento 1

```javascript
{
	nombre: "Juan",
	edad: 25
}
```

Documento 2

```javascript
{
	nombre: "Ana",
	edad: 28,
	telefono: "7777-7777"
}
```

Esto no genera problemas en MongoDB.

### Sin migraciones complejas

En bases relacionales, agregar un campo requiere modificar la tabla.

En MongoDB simplemente se empieza a usar el nuevo campo en nuevos documentos.

### Adaptabilidad

Los esquemas pueden evolucionar conforme crece la aplicación.

Esto es muy útil en proyectos ágiles donde los requisitos cambian constantemente.

## Buenas Prácticas de Diseño

Aunque MongoDB permite flexibilidad, es importante aplicar buenas prácticas.

### Identificar entidades

Antes de modelar los documentos, se deben identificar las entidades principales del sistema.

Ejemplo en un sistema de ventas:

- Clientes
- Productos
- Pedidos

### Diseñar según consultas

En bases relacionales se diseña pensando en **normalización**.

En MongoDB se diseña pensando en **cómo se consultarán los datos**.

Por ejemplo, si una aplicación necesita mostrar pedidos junto con información del cliente constantemente, puede ser conveniente **embebir los datos del cliente en el pedido**.

### Desnormalización estratégica

En MongoDB es común duplicar información para mejorar el rendimiento de lectura.

Ejemplo:

```javascript
{
	pedido_id: 1001,
	cliente: "Ana López",
	producto: "Laptop",
	precio: 8800
}
```

Aunque el nombre del cliente se repita, la consulta es más rápida.

## Eliminación de la Impedancia Objeto-Relacional

En bases de datos relacionales existe un problema llamado **impedancia objeto-relacional**.

Este problema surge porque:

- Las aplicaciones usan **objetos**
- Las bases de datos usan **tablas**

Esto requiere herramientas como **ORMs (Object Relational Mapping)** para convertir entre ambos modelos.

### Solución en MongoDB

MongoDB utiliza documentos JSON/BSON que se parecen mucho a los objetos utilizados en lenguajes de programación.

Ejemplo en JavaScript:

Objeto en la aplicación:

```javascript
{
	nombre: "Dereck",
	edad: 27
}
```

Documento en MongoDB:

```javascript
{
	nombre: "Dereck",
	edad: 27
}
```

Esto simplifica el desarrollo y reduce la necesidad de transformaciones complejas.

## Escalabilidad Horizontal

MongoDB está diseñado para escalar horizontalmente, lo que significa que la base de datos puede distribuirse entre varios servidores.

Esto se logra mediante **Sharding**.

### Sharding

El **sharding** consiste en dividir los datos en diferentes servidores llamados **shards**.

**Ejemplo:**

Servidor 1 → usuarios A–F  
Servidor 2 → usuarios G–M  
Servidor 3 → usuarios N–Z

Los documentos autocontenidos facilitan esta distribución.

## Versionado de esquemas

MongoDB no maneja automáticamente versiones de esquema.

Sin embargo, es posible implementar **versionado dentro del documento**.

**Ejemplo:**

```javascript
{
	nombre: "Dereck",
	edad: 27,
	schemaVersion: 2
}
```

Esto permite a la aplicación manejar diferentes estructuras de datos.

## Validaciones de Esquema en MongoDB

Aunque MongoDB permite esquemas flexibles, también es posible aplicar **validaciones para asegurar la calidad de los datos**.

MongoDB permite definir reglas utilizando **JSON Schema**.

### Ejemplo de Validación

```javascript
{
  $jsonSchema: {
    bsonType: "object",
    required: ["nombre", "edad"],
    properties: {
      nombre: {
        bsonType: "string"
      },
      edad: {
        bsonType: "int",
        minimum: 0
      }
    }
  }
}
```

Esta validación exige que:

- El documento tenga un campo **nombre**
- El campo **edad** sea un número entero mayor o igual a 0.

## Validación en MongoDB Compass

MongoDB Compass permite configurar validaciones de forma gráfica.

Pasos generales:

1. Abrir la colección.
2. Ir a la pestaña **Validation**.
3. Crear una regla de validación.
4. Definir el esquema JSON.
5. Guardar la configuración.

A partir de ese momento, los documentos que no cumplan con la validación **no podrán insertarse o actualizarse**.
