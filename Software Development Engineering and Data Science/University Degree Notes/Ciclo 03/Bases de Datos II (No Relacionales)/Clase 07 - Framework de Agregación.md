---
Fecha de creación: 2026-03-07T15:00:00
Materia:
  - Base de Datos II (No Relacionales)
Fecha de clase: 2026-03-07
---
# Framework de Agregación

El **Framework de Agregación** en MongoDB es una herramienta poderosa que permite **procesar y analizar grandes volúmenes de datos dentro de la base de datos**, de manera similar a las operaciones `GROUP BY`, `SUM`, `AVG` y `JOIN` en bases de datos relacionales.

A diferencia de consultas simples con `find()`, el framework de agregación permite realizar **transformaciones complejas de datos**, aplicar filtros, agrupar información y generar resultados analíticos.

Este proceso se realiza mediante un **pipeline de agregación**, donde los datos pasan por diferentes etapas de procesamiento.

## Pipeline de Agregación

El pipeline funciona como una **cadena de etapas** donde cada etapa recibe los documentos de la etapa anterior, los procesa y envía el resultado a la siguiente.

Estructura general:

```javascript
db.collection.aggregate([
   { etapa1 },
   { etapa2 },
   { etapa3 }
])
```

Cada etapa comienza con el símbolo `$`.

Ejemplo conceptual:

```bash
Colección → $match → $group → $sort → Resultado
```

## Ventajas del Framework de Agregación

El framework de agregación tiene varias ventajas importantes:

- Permite **procesar datos directamente en la base de datos**, evitando transferir grandes volúmenes de información a la aplicación.
- Facilita la **creación de reportes y análisis estadísticos**.
- Permite transformar documentos y crear nuevas estructuras de datos.
- Puede combinar información de varias colecciones.

En entornos de **analítica o ciencia de datos**, es muy útil para generar indicadores, métricas y reportes.

## Desventajas del Framework de Agregación

A pesar de su potencia, también presenta algunas limitaciones:

- Puede consumir muchos recursos si se ejecuta sobre grandes volúmenes de datos sin índices adecuados.
- Las consultas pueden volverse complejas y difíciles de mantener.
- Algunas operaciones pueden ser más eficientes si se realizan en herramientas analíticas externas.

Por esta razón es importante diseñar bien el **modelado de datos y los índices**.

## Etapas Principales del Pipeline

### ``$match``

La etapa `$match` se utiliza para **filtrar documentos**, de forma similar a una cláusula `WHERE` en SQL.

**Ejemplo:**

```javascript
db.ventas.aggregate([
  {
    $match: { categoria: "Electrónica" }
  }
])
```

Esto devuelve solo los documentos donde la categoría sea **Electrónica**.

Es recomendable colocar `$match` **al inicio del pipeline** para reducir la cantidad de datos procesados.

### ``$project``

La etapa `$project` permite **seleccionar campos específicos o transformar documentos**.

Ejemplo:

```javascript
db.ventas.aggregate([
  {
    $project: {
      producto: 1,
      precio: 1,
      _id: 0
    }
  }
])
```

Esto devuelve únicamente los campos **producto y precio**.

También se puede crear nuevos campos:

```javascript
{
  $project: {
    producto: 1,
    precioConIVA: { $multiply: ["$precio", 1.13] }
  }
}
```

### ``$group``

La etapa `$group` permite **agrupar documentos según un campo**, similar a `GROUP BY` en SQL.

Ejemplo:

```javascript
db.ventas.aggregate([
  {
    $group: {
      _id: "$categoria",
      totalVentas: { $sum: "$precio" }
    }
  }
])
```

## Operadores comunes en ``$group``

Dentro de `$group` se utilizan operadores de agregación.

|Operador|Función|
|---|---|
|`$sum`|Suma valores|
|`$avg`|Calcula promedio|
|`$max`|Obtiene valor máximo|
|`$min`|Obtiene valor mínimo|
|`$count`|Cuenta documentos|

**Ejemplo:**

Calcular el precio promedio por categoría.

```javascript
db.ventas.aggregate([
  {
    $group: {
      _id: "$categoria",
      precioPromedio: { $avg: "$precio" }
    }
  }
])
```

## ``$sort``

Permite ordenar los resultados.

```javascript
db.ventas.aggregate([
  { $group: { _id: "$categoria", total: { $sum: "$precio" } } },
  { $sort: { total: -1 } }
])
```

- `1` → ascendente
- `-1` → descendente

## ``$limit``

Limita la cantidad de resultados.

```javascript
db.ventas.aggregate([
  { $sort: { precio: -1 } },
  { $limit: 5 }
])
```

Devuelve los 5 productos más caros.

## ``$unwind``

La etapa `$unwind` se utiliza para **descomponer arreglos en múltiples documentos**.

Ejemplo de documento:

```json
{
  producto: "Laptop",
  tags: ["tecnología", "computadoras", "oficina"]
}
```

Aplicando ``$unwind``:

```javascript
{
	$unwind: "$tags"
}
```

Resultado:

```bash
Laptop - tecnología
Laptop - computadoras
Laptop - oficina
```

Esto permite trabajar cada elemento del arreglo individualmente.

## ``$lookup``
(similar a JOIN)

Permite **combinar datos de dos colecciones diferentes**.

**Ejemplo:**

Colección **ventas**

```yaml
{ producto_id: 1, cantidad: 2 }
```

Colección productos

```yaml
{ _id: 1, nombre: "Laptop", precio: 800 }
```

Consulta:

```javascript
db.ventas.aggregate([
  {
    $lookup: {
      from: "productos",
      localField: "producto_id",
      foreignField: "_id",
      as: "info_producto"
    }
  }
])
```

Esto agrega información del producto a la venta.

## ``$facet``

Permite ejecutar **múltiples agregaciones en paralelo dentro de la misma consulta**.

**Ejemplo:**

```javascript
db.ventas.aggregate([
{
  $facet: {
    ventasPorCategoria: [
      { $group: { _id: "$categoria", total: { $sum: "$precio" } } }
    ],
    precioPromedio: [
      { $group: { _id: null, promedio: { $avg: "$precio" } } }
    ]
  }
}
])
```

Esto genera dos análisis diferentes en una sola consulta.

## ``$sortByCount``

Cuenta documentos agrupados automáticamente.

**Ejemplo:**

```javascript
db.ventas.aggregate([
  { $sortByCount: "$categoria" }
])
```

Resultado:

```bash
Electrónica → 120
Ropa → 80
Hogar → 50
```

## ``$out``

La etapa `$out` guarda el resultado de una agregación en una colección nueva o existente.

**Ejemplo:**

```javascript
db.ventas.aggregate([
  { $group: { _id: "$categoria", total: { $sum: "$precio" } } },
  { $out: "reporte_ventas" }
])
```

Esto crea una colección llamada ``reporte_ventas``.

Es poco habitual porque generalmente las agregaciones se usan solo para análisis.

## Operaciones con Fechas

MongoDB permite trabajar con fechas.

**Ejemplo:** obtener el año actual.

```javascript
{
	$project: {
		añoActual: { $year: "$$NOW" }
	}
}
```

$addFlields, $substract

## Modelaje de Datos en Bases NoSQL

El **modelado de datos en NoSQL** es diferente al de bases relacionales.

En SQL se busca **normalización**, mientras que en NoSQL se prioriza **rendimiento de consultas**.

Existen dos estrategias principales:

### Embebido (Embedded Documents)

Los documentos relacionados se guardan **dentro del mismo documento**.

**Ejemplo:**

```javascript
{
  cliente: "Juan",
  pedidos: [
    { producto: "Laptop", precio: 800 },
    { producto: "Mouse", precio: 20 }
  ]
}
```

#### Cuándo usar embebido

- Relación **uno a pocos**
- Los datos se consultan juntos frecuentemente
- El tamaño del documento no crece demasiado

Ventajas:

- Consultas más rápidas
- Menos joins

### Referencias (Referencing)

Los documentos se guardan en **colecciones separadas** y se conectan mediante IDs.

**Ejemplo:**

Colección **clientes**

```json
{ _id: 1, nombre: "Juan" }
```

Colección **pedidos**

```json
{ cliente_id: 1, producto: "Laptop" }
```

#### Cuándo usar referencias

- Relaciones **uno a muchos grandes**
- Datos que cambian frecuentemente
- Documentos demasiado grandes

## Análisis de Cardinalidad

La **cardinalidad** describe la relación entre entidades.

**Tipos comunes:**

|Tipo|Ejemplo|
|---|---|
|Uno a uno|Persona - Pasaporte|
|Uno a muchos|Cliente - Pedidos|
|Muchos a muchos|Estudiantes - Cursos|

En MongoDB, analizar la cardinalidad ayuda a decidir entre:

- **Embebido**
- **Referencias**
