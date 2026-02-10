---
Fecha de creación: 2026-02-09T18:11:00
Materia:
  - Programación Web I (Backend)
Fecha de clase: 2026-02-09
---
# Bases de Datos y Code First

## Historia de las Bases de Datos
(ver [[Clase 01 - Definición de Base de Datos#Evolución Histórica y Tipos de Bases de Datos|Historia de las Bases de Datos]])

Con la aparición de los sistemas de gestión de bases de datos relacionales (RDBMS) en la década de 1970, liderados por el trabajo de Edgar F. Codd, se revolucionó la manera de organizar datos. Se introdujo el concepto de tablas, relaciones y SQL, permitiendo consultas estructuradas y garantizando integridad mediante reglas como claves primarias y foráneas.

Durante los años 2000, el aumento de la información y la necesidad de alta disponibilidad generaron la aparición de bases de datos no relacionales (NoSQL), que permiten almacenar información en formatos flexibles como documentos JSON, pares clave-valor, grafos o columnas, optimizando la escalabilidad horizontal y el rendimiento en sistemas distribuidos ([[Clase 01 - Fundamentos de las Bases de Datos NoSQL#Tipos de Bases de Datos NoSQL|Tipos de Bases de Datos NoSQL]]).

## Introducción a los ORMs y EF Core

Con el avance de la programación orientada a objetos y la separación entre **código y base de datos**, surgieron los **Object-Relational Mappers (ORMs)**. Estos permiten mapear clases y objetos en código directamente a tablas de bases de datos, reduciendo la necesidad de escribir consultas SQL manualmente y facilitando la sincronización entre la aplicación y la base de datos.

**Entity Framework Core (EF Core)** es un ORM moderno para .NET que soporta múltiples motores de base de datos. Su enfoque **Code First** permite definir la base de datos directamente desde las clases de dominio, facilitando el desarrollo ágil y la evolución del sistema sin perder consistencia.

## Tipos de Bases de Datos

### Bases de Datos Relacionales (RDBMS)
(ver clase [[Clase 01 - Definición de Base de Datos]])

Las bases de datos relacionales organizan la información en tablas con filas y columnas, donde cada fila representa un registro y cada columna un atributo.

- **Ventajas:**
	- Integridad referencial mediante claves primarias y foráneas.
	- Transacciones ACID garantizan consistencia y confiabilidad.
	- SQL estandarizado facilita interoperabilidad entre sistemas.
- **Desventajas:**
	- Escalabilidad horizontal más compleja.
	- Rigidez frente a cambios frecuentes en el esquema.

**Ejemplos:** SQL Server, MySQL, PostgreSQL, Oracle.

**Tabla ``Productos``**

| Id  | Nombre | Precio | Stock |
| --- | ------ | ------ | ----- |
| 1   | Laptop | 1200   | 100   |

### Bases de Datos No Relacionales (NoSQL)
(ver [[Clase 01 - Fundamentos de las Bases de Datos NoSQL]])

Diseñadas para alta escalabilidad y flexibilidad de esquema, soportan distintos modelos de datos:

- **Documental:** MongoDB, CouchDB - almacena datos en documentos JSON.
- **Clave-valor:** Redis - rápido acceso por clave única.
- **Columnar:** Cassandra - optimizado para grandes volúmenes de datos.
- **Grafos:** Ne04j - enfocado en relaciones complejas.

- **Ventajas:**
	- Escalabilidad horizontal.
	- Flexibilidad de esquema.
	- Alto rendimiento en consultas simples y masivas.
- **Desventajas:**
	- Menor consistencia estricta (algunos casos eventual consistency).
	- No soportan relaciones complejas como en RDBMS.

### Comparación Resumida

| Característica       | Relacional (RDBMS) | No Relacional (NoSQL)          |
| -------------------- | ------------------ | ------------------------------ |
| Estructura           | Tablas             | Documentos, Clave-valor, Grafo |
| Esquema              | Fijo               | Dinámico                       |
| Lenguaje de consulta | SQL                | Propio o API                   |
| Integridad           | Alta (ACID)        | Variable (BASE)                |
| Escalabilidad        | Vertical           | Horizontal                     |

> [!IMPORTANT] Propiedad de navegación se puede ver como las llaves foráneas en SQL.

## Concepto de Code First

**Code First** permite crear la base de datos **a partir de los modelos definidos en el código**, evitando diseñar la base de datos manualmente.

 - **Ventaja:**
	- Sincronización automática entre modelos y base de datos.
	- Facilita iteraciones rápidas en desarrollo ágil.
	- Permite versionar cambios de esquema mediante migraciones.

### Flujo de trabajo típico en Code First:

1. Crear clases de dominio en .NET 8.
2. Configurar relaciones y validaciones (Data Annotations o Fluent API).
3. Generar migraciones (``dotnet ef migrations add<nombre>``)
4. Aplicar cambios a la base de datos (dotnet ef database update).
5. Usar DbContext para CRUD y consultas.

### Comparación con Database First

| Enfoque        | Code First                     | Database First                     |
| -------------- | ------------------------------ | ---------------------------------- |
| Diseño inicial | Clases en código               | Base de datos existente            |
| Flexibilidad   | Alta, evoluciona con el código | Baja, depende del esquema          |
| Migraciones    | Automáticas                    | Manuales                           |
| Uso típico     | Nuevos proyectos               | Proyectos con base de datos legada |

## Migraciones y Sincronización de Base de Datos

Una de las grandes ventajas de trabajar con **Code First en Entity Framework Core** es la capacidad de reflejar los cambios en los modelos de dominio de manera incremental en la base de datos mediante las migraciones. Este mecanismo evita la necesidad de crear manualmente scripts SQL para actualizar la estructura de la base de datos cada vez que agregamos o modificamos propiedades de nuestras clases.

### Concepto y propósito de las migraciones

Las migraciones son un conjunto de instrucciones que representan cambios en los modelos de código que deben aplicarse a la base de datos, Su principal objetivo es mantener consistencia entre la aplicación y la base de datos, garantizando que cualquier cambio en el modelo tenga su reflejo exacto en la estructura de tablas, columnas, índices y relaciones.

**Ventajas principales:**

- Permite la evolución de la base de datos sin perder datos existentes.
- Facilita el desarrollo ágil, ya que los cambios en modelos se traducen automáticamente a la base de datos.
- Integra control de versiones, lo que permite revertir cambios si es necesario,

**Historia breve:**

En los primeros sistemas de desarrollo, los cambios en la base de datos se hacían manualmente mediante scripts SQL. Con la aparición de ORMs como Entity Framework, surgió la necesidad de automatizar este proceso, reduciendo errores humanos y asegurando sincronización constante entre código y base de datos.

### Flujo de trabajo de las migraciones desde cero

Cuando comenzamos un proyecto nuevo en .NET 8 utilizando Entity Framework Core, normalmente no existe ni la base de datos ni los modelos iniciales. EI flujo de trabajo con Code First permite crear todo desde el código, de manera ordenada y controlada.

#### Crear el modelo inicial

Primero, definimos nuestras clases de dominio que representarán las entidades de nuestra aplicación. Por ejemplo, para un sistema de inventario, creamos la clase ``Producto``:

```csharp
using System.ComponentModel.DataAnnotations;

public class Producto {
	[Key]
	public int Id { get; set; }
	
	[Required]
	[MaxLength(100)]
	public string Nombre { get; set; }
	
	[Range(0.01, double.MaxValue)]
	public decimal Precio { get; set; }
	
	[Range(0, int.MaxValue)]
	public int Stock { get; set; }
}
```

**Explicación:**

- ``Id``: clave primaria de la tabla.
- ``Nombre``: obligatorio, con máximo 100 caracteres.
- ``Precio``: debe ser mayor a cero.
- ``Stock``: cantidad disponible, no puede ser negativa.

En este punto **no existe ninguna base de datos**; solo tenemos nuestro modelo en código.

#### Configurar el DbContext

El siguiente paso es crear la clase ``DbContext``, que actúa como **puente entre el modelo y la base de datos**:

```csharp
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext {
	public DbSet<Producto> Productos { get; set; }
	
	protected override void OnConfiguring(DbContextOptionBuilder options) => options.UseSqlServer("Server=.;Database=InventarioDB;Trusted_Connection=True;")
}
```

> [!IMPORTANT] Para el ``DBContext`` hay que instalar el ``EntityFrameworkCore`` y llamarlo con ``using``.

**Explicación:**

- ``DbSet<Producto>`` indica que queremos una tabla para la entidad ``Producto``.
- ``UseSqlServer`` establece la conexión a SQL Server.
- En este punto **aún no se ha creado la base de datos**, solo está configurada la conexión.

#### Crear la migración inicial

Ahora generamos la **migración inicial**, que EF Core usará para crear la base de datos y la tabla correspondiente al modelo ``Producto``:

**dotnet ef migrations add Inicial**

Esto genera una carpeta ``Migrations`` con un archivo C# que contiene dos métodos:

```csharp
protected override void Up(MigrationBuilder migrationBuilder) {
	migrationBuilder.CreateTable(
		name: "Productos",
		columns: table => new {
			Id = table.Column<int>(nullable: false).Annotation("SqlServer:Identity", "1, 1"),
			Nombre = table.Column<string>(maxLength: 100, nullable: false),
			Precio = table.Column<decimal>(nullable: false),
			Stock = table.Column<int>(nullable: false)
		},
		constraints: table => {
			table.PrimaryKey("PK_Productos", x => x.Id);
		}
	);
}

protected override void Down(MigrationBuilder migrationBuilder) {
	migrationBuilder.DropTable(
		name: "Productos"
	);
}
```

**Explicación:**

- ``Up()``: Define la creación de la tabla con sus columnas y restricciones.
- ``Down()``: Define cómo revertir la migración, eliminando la tabla.

#### Aplicar la migración a la base de datos

Con la migración lista, ahora sí creamos la base de datos física y la tabla ``Productos``:

**dotnet ef database update**

**Resultado:**

- EF Core conecta con SQL Server.
- Crea la base de datos ``InventarioDB``.
- Crea la tabla Productos con las columnas definidas en el modelo.

En este punto, ya tenemos **modelo y base de datos sincronizados**, listos para empezar a insertar datos y hacer consultas.
