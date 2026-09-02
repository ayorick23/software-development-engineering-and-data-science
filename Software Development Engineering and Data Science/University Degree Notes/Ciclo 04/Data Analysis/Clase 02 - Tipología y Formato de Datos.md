---
Fecha de creación: 2026-07-14T18:22:00
Materia:
  - Data Analysis
Fecha de clase: 2026-07-14
---
# Tipología y Formato de Datos


## Sintaxis vs. Semántica

| España | 2015 | 2018 |
| ------ | ---- | ---- |
| Italia | 2012 | 2017 |
Sintaxis pura (ambigua)

| País   | Inicio | Fin  |
| ------ | ------ | ---- |
| España | 2015   | 2018 |
| Italia | 2012   | 2017 |
Semántica (comprensible)

## Ecosistema de los Datos



### Datos Estructurados

- Estructura Base: Filas (registros/ejemplos) y Columnas (atributos/variables).
- Acceso: Por nombres de columnas bajo un esquema definido.
- Lenguaje Universal: SQL (Structured Query Language).
- Evolución Big Data (Escalabilidad):
	- NoSQL: Almacenamiento clave-valor (MongoDB, HBase).
	- Acceso Columnar: Cassandra, Hive, Impala.

### Datos No Estructurados

Diseñados para transmisión web (HTTP), combinan estructura y datos de forma flexible.

#### XML (_Extensible Markup Language_)

Basado en etiquetas lógicas. Pesado y verboso. Validación estricta vía DTD.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE pais PUBLIC "pais.dtd">
<pais>
	<nombre>Pais</nombre>
	<inicio>2015</inicio>
	<final>2018</final>
</pais>
```



