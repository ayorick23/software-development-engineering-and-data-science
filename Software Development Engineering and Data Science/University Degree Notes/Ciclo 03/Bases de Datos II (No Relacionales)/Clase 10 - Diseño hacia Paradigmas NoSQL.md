---
Fecha de creación: 2026-03-28T14:07:00
Materia:
  - Base de Datos II (No Relacionales)
Fecha de clase: 2026-03-28
---
# Diseño, Optimización y Migración hacia Paradigmas NoSQL

## Diseñar el Esquema Según Patrones de Acceso

MongoDB es una base de datos NoSQL orientada a documentos, 10 que permite modelar los datos de forma flexible. Una buena práctica fundamental es embeber (incluir) los datos que se consultan juntos en un mismo documento. Esto reduce la necesidad de realizar múltiples consultas o ``JOINS``, mejorando significativamente el rendimiento.

EI diseño debe centrarse en cómo la aplicación accede a los datos, no en normalizar como en SQL. La desnormalización es una técnica poderosa en NoSQL que permite almacenar datos redundantes para evitar operaciones costosas de unión.

## ¿Qué es un Patrón de Acceso?



## Diseñar el Esquema Según Patrones de Acceso

1. **Identificar Patrones:** Analizar qué datos se consultan juntos frecuentemente en la aplicación
2. **Embeber Datos:** Incluir documentos relacionados dentro de un documento principal.
3. **Optimizar Consultas:** Reducir operaciones de ``JOIN`` y múltiples accesos a la base de datos.

## Embeber vs. Referenciar

### Embeber

- Datos que siempre se consultan juntos.
- Documentos de tamaño acotado (dentro del límite de 16MB).
- Relaciones de uno a pocos que no crecen indefinidamente (menos de 100 documentos).

### Referenciar

- Datos con alta reutilización en múltiples contextos (usuarios).
- Arrays de crecimiento ilimitado mayor a 100 registros (ej: logs, historial, comentarios de Tik tok).
- Datos que se actualizan frecuentemente de forma independiente.

 >[!IMPORTANT] **La regla es:** modela tu base según tus consultas.

**Ejemplo de dato referenciado en múltiples contextos:**

Usuario:

```json
{
	"_id": "65a1b2c3d4e5f6789abcdef",
	"email": "carlos@empresa.com",
	"nombre": "Carlos López",
	"rol_ids": [
		"",
		""
	]
}
```

Rol 1:

```json
{
	"_id": "",
	"nombre": "cliente",
	"permisos": [
		"comprar",
		"ver_productos",
		"devoluciones"
	],
	"contextos": [
		"ecommerce",
		"app_movil"
	]
}
```

Rol 2:

```json
{
	"_id": "",
	"nombre": "cliente",
	"permisos": [
		"comprar",
		"ver_productos",
		"devoluciones"
	],
	"contextos": [
		"ecommerce",
		"app_movil"
	]
}
```

## Ejemplo 1: Sistema de Gestión de Pedidos

Una aplicación de gestión de pedidos de una tienda en línea ilustra perfectamente las diferencias entre SQL y NoSQL. En SQL, tendríamos tablas normalizadas como Clientes, Pedidos y Productos. En NoSQL (MongoDB), se puede optimizar embebiendo los datos relacionados.

### Estructura SQL (Normalizada)

Tabla Clientes separada
Tabla Pedidos independiente
Tabla Productos individual
Joins necesarios para consultar
Relaciones mediante claves foráneas

### Estructura NoSQL (Desnormalizada)

Documento único con todos los datos
Cliente embebido en el pedido
Productos incluidos en array
Sin joins, consulta directa
Acceso rápido y eficiente

```json
{
	"_id": "ORD-001",
	"cliente": {
		"id_cliente": "C-123",
		"nombre": "María López",
		"correo": "maria@uees.com"
	},
	"fecha": "2026-10-29",
	"productos": [
		{
			"_id_productos"
		}
	]
}
```

## Ejemplo 2: Sistema de Usuarios con Direcciones

Consideremos una colección de usuarios donde cada usuario tiene varias direcciones. Si las direcciones siempre se consultan junto con la información del usuario, es apropiado incluirlas directamente en el documento del usuario.

### Documento Embebido

Las direcciones se almacenan como un array dentro del documento del usuario, permitiendo acceso directo sin consultas adicionales.

```json
{
	"nombre": "Ana",
	"direcciones": [
		{ "ciudad": "San Salvador", "tipo": "casa" },
		{ "ciudad": "Santa Ana", "tipo": "trabajo" }
	]
}
```

#### Ventajas

- Consulta única para obtener usuario y direcciones
- No requiere JOINS o consultas múltiples
- Mejor rendimiento en lecturas
- Estructura intuitiva y fácil de entender

#### Consideraciones
- EI documento puede crecer con muchas direcciones
- Actualizaciones de direcciones modifican todo el documento
- Evaluar si las direcciones se consultan siempre juntas
- Considerar referencias si las direcciones son compartidas

## Usar Índices Apropiados

Los índices permiten acceder rápidamente a los datos sin tener que recorrer toda la colección. Es fundamental crear índices en los campos que se consultan con frecuencia, se ordenan o se filtran. Una estrategia de indexación adecuada puede mejorar el rendimiento de las consultas en órdenes de magnitud.

### índice Simple

Creado sobre un solo campo. Ideal para búsquedas básicas y filtros en campos individuales.

```javascript
// Creación
db.usuarios.createIndex({ "nombre": 1 })

// Consultas que soporta
db.usuarios.find({ "nombre": "Juan Pérez" }) // Igualdad exacta
db.usuarios.find({ "nombre": /^Juan/ })      // Prefijo (regex optimizado)
db.usuarios.find({  })
db.usuarios.find({  })

// NO soporta búsqueda por palabras parciales
db.usuarios.find({ "nombre": /Pérez/ }) // Escaneo completo
```

### Índice Compuesto

Sobre varios campos combinados. Perfecto para consultas que filtran por múltiples criterios.

### Índice Geoespacial

Para datos de ubicación y consultas de proximidad. Útil en aplicaciones de mapas y geolocalización como: Kontrol, Golán, inDrive y Uber.

### Índice de Texto

Para búsquedas de texto completo. Habilita búsquedas avanzadas con palabras clave y relevancia.

```javascript
// Creación
db.usuarios.createIndex({ "nombre": "text" })

// Consultas que soporta
db.usuarios.find({ $text: { $search: "Juan" } }) // Palabra suelta
db.usuarios.find({ $text: { $search: "Juan Pérez" } }) // Palabras múltiples
db.usuarios.find({ $text: { $search: "\"Juan\"" } }) // Frase exacta
db.usuarios.find({ $text: { $search: "Juan -María" } }) // Exclusión

// NO soporta 


// 

```

## Aggregation Pipeline

EI **aggregation pipeline** es una herramienta poderosa para realizar transformaciones, agrupaciones y análisis complejos sobre los datos. Funciona como una serie de etapas (stages) que procesan los documentos paso a paso, permitiendo operaciones sofisticadas de procesamiento de datos.

```javascript
db.ventas.aggregate([
	{ $match: { categoria: "electrónica" } },
	
])
```


Este pipeline filtra ventas de electrónica, agrupa por producto sumando las cantidades vendidas, y ordena los resultados de mayor a menor. Es equivalente a una consulta SQL con WHERE, GROUP BY y ORDER BY, pero con mayor flexibilidad y potencia.

## Migración de SQL a MongoDB

Crear una base de datos para una
plataforma de cursos en línea ilustra el
proceso de migración desde SQL a
MongoDB. La plataforma contiene
usuarios (estudiantes y docentes), cursos,
inscripciones y lecciones.
