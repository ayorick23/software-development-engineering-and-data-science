# 29 - Introspección y Metaprogramación

> Continúa de [[28 - Namespaces, Scope y Closures a Fondo|28]]. La **introspección** es la capacidad de un programa de examinar su propia estructura en tiempo de ejecución (qué atributos tiene un objeto, qué argumentos recibe una función); la **metaprogramación** va un paso más allá y **modifica** esa estructura dinámicamente.

Python es un lenguaje profundamente introspectivo: prácticamente todo lo que el intérprete "sabe" sobre un objeto está disponible para el propio programa en tiempo de ejecución.

## Lo Básico: `dir()`, `vars()`, `type()`

```python
class Persona:
    especie = "Homo sapiens"

    def __init__(self, nombre):
        self.nombre = nombre

    def saludar(self):
        return f"Hola, soy {self.nombre}"

p = Persona("Dereck")

print(type(p))         # <class '__main__.Persona'>
print(dir(p))           # lista de TODOS los atributos y métodos (propios + heredados)
print(vars(p))          # {'nombre': 'Dereck'} -> solo el __dict__ de la instancia
print(p.__dict__)       # equivalente a vars(p)
print(Persona.__dict__) # namespace de la CLASE: {'especie': ..., '__init__': ..., 'saludar': ...}
```

`vars(objeto)` es literalmente `objeto.__dict__`; por eso no funciona con objetos que usan `__slots__` (ver [[36 - Descriptors|36]]), que no tienen `__dict__` propio.

## Acceso Dinámico a Atributos

```python
print(getattr(p, "nombre"))                 # "Dereck" -> equivalente a p.nombre
print(getattr(p, "edad", "desconocida"))    # "desconocida" -> valor por defecto si no existe

setattr(p, "edad", 25)                      # equivalente a p.edad = 25
print(hasattr(p, "edad"))                   # True

delattr(p, "edad")                          # equivalente a del p.edad
print(hasattr(p, "edad"))                   # False
```

Esto habilita patrones donde el **nombre del atributo es un dato en tiempo de ejecución**, no algo que puedas escribir literalmente en el código:

```python
def actualizar_desde_formulario(objeto, datos: dict):
    for campo, valor in datos.items():
        if hasattr(objeto, campo):
            setattr(objeto, campo, valor)

actualizar_desde_formulario(p, {"nombre": "Ana", "campo_inexistente": 1})
```

Este patrón es, en esencia, lo que hacen internamente frameworks como Pydantic (usado en [[Python/FastAPI/03 - Request Body y Modelos Pydantic|Python/FastAPI/03]]) y los ORMs para poblar objetos a partir de datos externos.

## `callable()` y Objetos que se Comportan como Funciones

```python
print(callable(p))          # False
print(callable(p.saludar))  # True
print(callable(print))      # True
print(callable(42))         # False
```

Cualquier objeto puede volverse invocable definiendo `__call__` (ver [[37 - Dunder Methods y Protocolos Avanzados|37]]), lo cual es la base de decoradores basados en clases y de librerías como los transformadores de scikit-learn (`fit`/`transform`).

## El Módulo `inspect`

`inspect` es la herramienta estándar para introspección seria — mucho más rica que `dir()`.

```python
import inspect

def saludar(nombre: str, saludo: str = "Hola") -> str:
    """Devuelve un saludo personalizado."""
    return f"{saludo}, {nombre}!"

# Firma completa de la función, con tipos y valores por defecto
firma = inspect.signature(saludar)
print(firma)                          # (nombre: str, saludo: str = 'Hola') -> str
for nombre, parametro in firma.parameters.items():
    print(nombre, parametro.default, parametro.annotation)

print(inspect.getsource(saludar))     # el código fuente de la función, como texto
print(inspect.getdoc(saludar))        # "Devuelve un saludo personalizado."

print(inspect.isfunction(saludar))    # True
print(inspect.isclass(Persona))       # True
print(inspect.ismethod(p.saludar))    # True (método ligado a una instancia)

# Todos los miembros de un objeto, con filtros
metodos = inspect.getmembers(Persona, predicate=inspect.isfunction)
print(metodos)  # [('saludar', <function Persona.saludar>)]
```

`inspect.signature` es exactamente lo que usan frameworks de inyección de dependencias (como FastAPI, ver [[Python/FastAPI/06 - Dependency Injection|Python/FastAPI/06]]) para descubrir automáticamente qué parámetros necesita una función y resolverlos.

## Creación Dinámica de Clases y Funciones

Además de leer la estructura del programa, Python permite **crearla** en tiempo de ejecución.

```python
# Crear una función dinámicamente
codigo = "def suma(a, b): return a + b"
espacio_local = {}
exec(codigo, {}, espacio_local)
suma = espacio_local["suma"]
print(suma(2, 3))  # 5

# Crear una clase dinámicamente con type() (ver 38 - Metaclases y type())
Punto = type("Punto", (), {"x": 0, "y": 0, "describir": lambda self: f"({self.x}, {self.y})"})
p1 = Punto()
print(p1.describir())  # (0, 0)
```

> **Advertencia de seguridad:** `exec()` y `eval()` ejecutan código arbitrario con los permisos del proceso actual. **Nunca** los uses sobre datos que provengan de un usuario o de una fuente no confiable (ver [[62 - Seguridad en Python|62 - Seguridad en Python]] para el detalle de por qué y las alternativas seguras como `ast.literal_eval`).

## Metaprogramación con Decoradores de Clase

Un patrón intermedio entre introspección pura y metaclases completas ([[38 - Metaclases y type()|38]]) es modificar una clase justo después de definirla:

```python
def agregar_repr(cls):
    """Decorador de clase: genera un __repr__ automático a partir de __dict__."""
    def __repr__(self):
        atributos = ", ".join(f"{k}={v!r}" for k, v in vars(self).items())
        return f"{cls.__name__}({atributos})"
    cls.__repr__ = __repr__
    return cls

@agregar_repr
class Producto:
    def __init__(self, nombre, precio):
        self.nombre = nombre
        self.precio = precio

print(Producto("Teclado", 49.99))  # Producto(nombre='Teclado', precio=49.99)
```

Esto anticipa exactamente lo que `@dataclass` (ver [[40 - Dataclasses, NamedTuple y Comparativa con Attrs-Pydantic|40]]) hace automáticamente por ti.

## ¿Cuándo Usar Metaprogramación?

| Escenario | ¿Recomendado? |
|---|---|
| Generar `__repr__`/`__eq__` automáticamente | Sí — pero mejor usar `@dataclass` ya construido para esto |
| Validar/transformar datos de un formulario o API externa | Sí — con `getattr`/`setattr` explícitos |
| Registrar automáticamente subclases (plugins) | Sí — con `__init_subclass__` (ver [[38 - Metaclases y type()|38]]) |
| Ejecutar código arbitrario recibido por red o de un usuario | **No** — riesgo de seguridad grave |
| "Porque se ve elegante" sin un problema concreto que resolver | No — la metaprogramación sacrifica legibilidad; solo se justifica cuando el ahorro de repetición es real |

## Ver también

- [[38 - Metaclases y type()]] — el siguiente nivel de metaprogramación: interceptar la creación misma de la clase.
- [[36 - Descriptors]] — cómo se implementa el acceso a atributos por debajo de `getattr`.
- [[42 - Type Hints y el Módulo Typing]] — cómo `inspect.signature` se combina con anotaciones de tipo.
- [[62 - Seguridad en Python]] — riesgos de `eval`/`exec` sobre entradas no confiables.
