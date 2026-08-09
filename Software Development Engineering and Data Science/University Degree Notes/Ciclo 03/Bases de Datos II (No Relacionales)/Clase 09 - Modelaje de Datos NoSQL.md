---
Fecha de creación: 2026-03-21T14:07:00
Materia:
  - Base de Datos II (No Relacionales)
Fecha de clase: 2026-03-21
---
[[Clase 08 - Diseño de Esquemas en Bases de Datos|← Clase anterior]] | [[Clase 10 - Diseño hacia Paradigmas NoSQL|Clase siguiente →]]

# Modelaje de Datos NoSQL

El **modelaje de datos en bases de datos NoSQL** consiste en diseñar la estructura de los datos de manera que las consultas sean eficientes y el sistema pueda escalar correctamente.

A diferencia de las bases de datos relacionales, donde el diseño se basa principalmente en la **normalización de datos**, en NoSQL el diseño se enfoca en **cómo se accederá a los datos en la aplicación**.

Por esta razón, el modelado en NoSQL suele priorizar:

- Rendimiento de lectura
- Escalabilidad
- Flexibilidad estructural

## Diferencias Conceptuales con SQL

El enfoque de diseño en bases de datos NoSQL presenta varias diferencias importantes respecto a SQL.

- Modelaje orientado a consultas
- Desnormalización estratégica
- Escalabilidad horizontal
- Flexibilidad de esquema

## Desnormalización en NoSQL

La **desnormalización** consiste en almacenar información relacionada dentro del mismo documento o duplicar datos en diferentes documentos para optimizar consultas.

Ventajas:

- Consultas más rápidas
- Menos JOINS
- Menor complejidad de consultas

Desventajas:

- Posible duplicación de datos
- Mayor cuidado al actualizar información

## Teorema CAP
(ver [[Clase 01 - Fundamentos de las Bases de Datos NoSQL#Teorema CAP (Fundamental)|Teorema CAP]])

El **Teorema CAP** establece que un sistema distribuido no puede garantizar simultáneamente las tres propiedades siguientes:

- **Consistency (Consistencia)**: Todos los nodos ven los mismos datos al mismo tiempo.
- **Availability (Disponibilidad)**: El sistema siempre responde a las solicitudes.
- **Partition Tolerance (Tolerancia a particiones)**: El sistema sigue funcionando aunque haya fallos de red.

En bases de datos NoSQL normalmente se prioriza:

```
Disponibilidad + Tolerancia a particiones
```

MongoDB intenta equilibrar consistencia y disponibilidad mediante mecanismos de **replicación y replica sets**.

## Bases de Datos Clave-Valor

Este es el modelo más simple de NoSQL. Cada elemento se almacena como un par clave-valor, similar a un diccionario o tabla hash distribuida.

### Diseño de claves

Las claves deben diseñarse cuidadosamente porque determinan cómo se recuperan los datos.

Ejemplo:

```
user:1001
order:2024:345
session:abc123
```

### Estructura de valores

El valor puede ser:

- JSON
- texto
- binario

## ¿Qué es el polimorfismo?

Aplicado a bases de datos, el **polimorfismo de esquema** describe la capacidad de almacenar documentos con estructuras distintas dentro de la misma colección, siempre que representen variantes de una misma entidad. Por ejemplo, en una colección `productos`, un documento de tipo "libro" puede tener el campo `autor` y otro de tipo "electrónico" puede tener `garantia`, sin que eso rompa la colección. Es una consecuencia directa de que MongoDB no exige un esquema fijo (ver [[Clase 08 - Diseño de Esquemas en Bases de Datos#¿MongoDB necesita esquemas?|¿MongoDB necesita esquemas?]]).

## Bases de Datos Documentales

MongoDB pertenece a esta categoría.

Los datos se almacenan como **documentos JSON/BSON**.

- **Embedding:** Los datos relacionados se guardan dentro del mismo documento.
- **Referencing:** Los documentos se conectan mediante IDs.
- **Esquema Polimórfico:** En MongoDB diferentes documentos pueden tener estructuras distintas.

### Diferencia entre JSON y BSON

JSON:

- Formato de texto
- Fácil de leer
- Utilizado en APIs

BSON:

- Formato binario
- Más eficiente para almacenamiento
- Soporta más tipos de datos

MongoDB almacena documentos en **BSON**.

## Bases de Datos de Familia de Columnas

Este modelo organiza los datos en columnas en lugar de filas.

Se utilizan principalmente para:

- Datos de series temporales
- Análisis de grandes volúmenes
- Alta escritura

**Ejemplos:**

- Apache Cassandra
- HBase

## Bases de Datos de Grafos

Diseñadas para modelar relaciones complejas.

- **Nodos:** Representan entidades.
- **Aristas:** Representan relaciones.
- **Propiedades:** Atributos de nodos o relaciones

**Ejemplos:**

- Neo4j
- Amazon Neptune

## Estrategias de Diseño

Para diseñar correctamente un sistema NoSQL se deben considerar varios factores.

- **Comprender patrones de acceso:** Antes de diseñar la base de datos se deben identificar las consultas frecuentes, operaciones de lectura y escritura.
- **Desnormalización intencional:** Duplicar datos cuando mejora el rendimiento.
- **Índices estratégicos:** Crear índices para campos que se consultan frecuentemente.

## Patrones Avanzados por Tipo de Base de Datos

### Clave-Valor

- Diseñar claves jerárquicas para escaneos por prefijo
- Usar TTL para datos temporales
- Implementar patrones de cache-aside
- Operaciones atómicas para evitar condiciones de carrera

### Documentales

- Aplicar embedding 1:N para cuando N es pequeño
- Usar referencias para datos grandes o compartidos
- Crear índice compuestos para múltiples campos
- Aprovechar pipeline de agregación

### Familia de Columnas

- Diseñar tablas orientadas a consultas específicas
- Elegir claves de partición que distribuya uniformemente
- Usar claves de clustering para ordenar datos
- Minimizar consultas entre particiones

### Grafos

- Indexar propiedades frecuentemente consultadas
- Usar etiquetas para categorizar nodos
- Diseñar relaciones con direccionalidad específica
- Limitar profundidad de traversal

## Casos de Estudio: Redes Sociales

Una aplicación de redes sociales necesita mostrar feeds personalizados, perfiles de usuario y gestionar relaciones sociales. Aquí se combinarían diferentes paradigmas:

- **Base documental:** Perfiles de usuario con información embebida como preferencia, configuración y estadísticas.
- **Base de grafos:** Modelar relaciones sociales (seguir, amistades, bloques) y recomendaciones.
- **Base de columnas:** Almacenar eventos y actividad histórica.
- **Base clave-valor:** Cachear feeds pre-calculados y sesiones de usuario.

## Ejemplos de Validación de Esquemas

Las validaciones pueden realizarse en **dos niveles**:

1. En el backend (código)
2. En la base de datos

La mejor práctica es usar **ambas**.

### Ejemplo de Validación en MongoDB

```javascript
db.createCollection("users", {
 validator: {
   $jsonSchema: {
     bsonType: "object",
     required: ["name","email"],
     properties: {
       name: { bsonType: "string" },
       email: { bsonType: "string" },
       age: { bsonType: "int", minimum: 0 }
     }
   }
 }
})
```

### Ejemplo de Validación en JavaScript (Node.js)

Instalar driver:

```
npm install mongodb
```

**Conexión a MongoDB:**

```javascript
const { MongoClient } = require("mongodb");

const client = new MongoClient("mongodb://localhost:27017");

async function connectDB(){
 await client.connect();
 return client.db("appdb");
}
```

**POST - Crear usuario:**

```javascript
app.post("/users", async (req,res)=>{

 const db = await connectDB();

 const user = {
  name: req.body.name,
  email: req.body.email,
  age: req.body.age
 }

 const result = await db.collection("users").insertOne(user);

 res.json(result);
});
```

La validación ocurre en el backend y nuevamente en MongoDB.

**GET - Obtener usuarios:**

```javascript
app.get("/users", async (req,res)=>{

 const db = await connectDB();

 const users = await db.collection("users").find().toArray();

 res.json(users);
});
```

**PUT - Actualizar usuario:**

```javascript
app.put("/users/:id", async (req,res)=>{

 const db = await connectDB();

 await db.collection("users").updateOne(
  {_id: ObjectId(req.params.id)},
  {$set: req.body}
 );

 res.send("Usuario actualizado");
});
```

### Ejemplo de Validación con Python (Flask)

**Instalar librería:**

```
pip install pymongo
```

**config.py:**

```python
from pymongo import MongoCliente

client = MongoCliente("mongodb://localhost:27017")
db = client["appdb"]
```

**database.py:**

```python
from config import db

users_collection = db["users"]
```

**POST:**

```python
@app.route("/users", methods=["POST"])
def create_user():

 data = request.json

 user = {
  "name": data["name"],
  "email": data["email"],
  "age": data["age"]
 }

 users_collection.insert_one(user)

 return {"message":"User created"}
```

**GET:**

```python
@app.route("/users", methods=["GET"])
def get_users():

 users = list(users_collection.find())

 return json.dumps(users)
```

## Cómo se conectan con MongoDB

Cuando se ejecuta un **POST o PUT**, ocurre lo siguiente:

```
Cliente
   ↓
API (Node o Python)
   ↓
Validación en código
   ↓
MongoDB
   ↓
Validación de esquema
   ↓
Documento guardado
```

Esto garantiza que los datos sean correctos antes de almacenarse.
