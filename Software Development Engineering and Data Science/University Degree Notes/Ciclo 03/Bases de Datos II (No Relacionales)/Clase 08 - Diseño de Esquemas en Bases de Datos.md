---
Fecha de creación: 2026-03-14T14:09:00
Materia:
  - Base de Datos II (No Relacionales)
Fecha de clase: 2026-03-14
---
# Diseño de Esquemas en Bases de Datos

## ¿Qué es el esquema de una base de datos?

**Concepto tradicional (IBM)**

Un esquema de base de datos define cómo se organizan los datos dentro de una base de datos relacional; incluye restricciones lógicas como los nombres de las tablas, los campos, los tipos de datos y las relaciones entre estas entidades.

- Acceso y seguridad
- Organización y comunicación
- Integridad

## ¿Qué desventajas tienen los esquemas de bases de datos?

Son muy rígidos, inflexibles, dificulta la actualización de una aplicación, alto tiempo de desarrollo.

## ¿MongoDB necesita esquemas de base datos? ¿Si o no?


## Diseño de Esquemas en Bases de Datos NoSQL (MongoDB)


## Esquemas Flexibles: La Ventaja NoSQL

- Estructura Variable
- Sin Migraciones
- Adaptibilidad

A diferencia de SQL, NoSQL permite esquemas dinámicos que evolucionan con la aplicación, facilitando el almacenamiento de datos sin 

## Buenas Prácticas de Diseño

- Identificar Entidades: Definir relaciones antes de modelar la estructura de datos.
- Desnormalización Estratégica: Preferida sobre normalización excesiva para mejorar lectura.
- Diseñar para Consultas: Optimizar según patrones de acceso más frecuentes.

Ejemplo: Sistema de Pedidos

```javascript
{
	"cliente": {
		"nombre": "Ana López",
		"correo": "ana@uees.edu.sv",
		"pedidos": [
			{"producto": "Laptop", "cantidad": 1},
			{"producto": "Mouse", "cantidad": 2}
		]	
	}
}
```

## Eliminación de la Impedancia

La impedancia es la incompatibilidad entre representación de datos en código (objetos) y almacenamiento en base de datos.

La solución en MongoDB documentos JSON mapean naturalmente a objetos en lenguajes orientados a objetos, eliminando complejidad de ORMs.

Escalabilidad Horizontal

Documento autocontenido
Sharding
Múltiples Servidores

Los esquemas flexibles facilitan el sharding, permitiendo distribuir documentos entre servidores sin preocuparse por mantener relaciones complejas.

El versionado de esquemas es automático?

## Validaciones
