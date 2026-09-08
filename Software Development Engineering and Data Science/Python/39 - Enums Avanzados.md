# 39 - Enums Avanzados

> Continúa de [[38 - Metaclases y type()|38]]. Los **enums** (enumeraciones) representan un conjunto fijo y nombrado de valores relacionados — la alternativa tipada y segura a usar strings o números "mágicos" sueltos por el código.

## El Problema que Resuelven

```python
# Sin Enum: strings mágicos, sin autocompletado, sin protección contra typos
def procesar_pedido(estado):
    if estado == "pendiente":
        ...
    elif estado == "pendinte":  # typo silencioso -> bug difícil de detectar
        ...
```

```python
from enum import Enum

class EstadoPedido(Enum):
    PENDIENTE = "pendiente"
    ENVIADO = "enviado"
    ENTREGADO = "entregado"
    CANCELADO = "cancelado"

def procesar_pedido(estado: EstadoPedido):
    if estado == EstadoPedido.PENDIENTE:
        ...
    elif estado == EstadoPedido.PENDINTE:  # AttributeError inmediato: no existe
        ...
```

Con `Enum`, un typo se convierte en un `AttributeError` detectable al instante (o incluso antes, con un linter/type-checker, ver [[43 - Análisis Estático con Mypy y Pyright|43]]), en vez de un bug silencioso que falla en producción.

## Uso Básico

```python
from enum import Enum

class Color(Enum):
    ROJO = 1
    VERDE = 2
    AZUL = 3

print(Color.ROJO)          # Color.ROJO
print(Color.ROJO.name)     # "ROJO"
print(Color.ROJO.value)    # 1
print(Color(1))            # Color.ROJO -> buscar un miembro por su valor
print(Color["ROJO"])       # Color.ROJO -> buscar un miembro por su nombre

for color in Color:        # los Enum son iterables
    print(color.name, color.value)

print(Color.ROJO == Color.ROJO)  # True
print(Color.ROJO is Color.ROJO)  # True -> los miembros son SINGLETONS
```

> Cada miembro de un `Enum` es un **singleton**: `Color.ROJO` siempre es el mismo objeto, sin importar cuántas veces lo referencies. Esto permite usar `is` con seguridad (a diferencia de la regla general de [[26 - Modelo de Objetos y Memoria en CPython|26]] de preferir `==`).

## Variantes de Enum

```python
from enum import IntEnum, Flag, auto, unique

# IntEnum: se comporta como int además de como Enum (comparable, sumable con números)
class Prioridad(IntEnum):
    BAJA = 1
    MEDIA = 2
    ALTA = 3

print(Prioridad.ALTA > Prioridad.BAJA)  # True -> comparación numérica directa
print(Prioridad.ALTA + 1)                # 4 -> se comporta como int

# auto(): asigna valores automáticamente, sin especificarlos a mano
class Dia(Enum):
    LUNES = auto()
    MARTES = auto()
    MIERCOLES = auto()

print(Dia.LUNES.value, Dia.MARTES.value)  # 1 2

# Flag: permite combinar miembros con operadores bit a bit (|, &)
class Permiso(Flag):
    LEER = auto()
    ESCRIBIR = auto()
    EJECUTAR = auto()

permisos_usuario = Permiso.LEER | Permiso.ESCRIBIR
print(permisos_usuario)                       # Permiso.LEER|ESCRIBIR
print(Permiso.LEER in permisos_usuario)       # True
print(Permiso.EJECUTAR in permisos_usuario)   # False

# @unique: previene valores duplicados accidentalmente
@unique
class Estado(Enum):
    ACTIVO = 1
    INACTIVO = 2
    # DESHABILITADO = 1  # ValueError si se descomenta: valor duplicado
```

| Variante | Cuándo usarla |
|---|---|
| `Enum` | Caso general: un conjunto cerrado de valores con nombre |
| `IntEnum` | Necesitas que los valores se comporten como enteros (comparaciones, serialización numérica) |
| `StrEnum` (3.11+) | Necesitas que los valores se comporten como strings (útil para JSON/APIs) |
| `Flag` | Los valores representan **combinaciones** de opciones (permisos, banderas de configuración) |
| `auto()` | No te importa el valor exacto, solo que cada miembro sea único |

## Métodos Personalizados en un Enum

Los enums pueden tener métodos normales, propiedades y hasta lógica de negocio:

```python
from enum import Enum

class Moneda(Enum):
    USD = "USD"
    EUR = "EUR"
    MXN = "MXN"

    @property
    def simbolo(self):
        simbolos = {"USD": "$", "EUR": "€", "MXN": "$"}
        return simbolos[self.value]

    @classmethod
    def desde_codigo_iso(cls, codigo: str) -> "Moneda":
        return cls(codigo.upper())

print(Moneda.EUR.simbolo)                    # "€"
print(Moneda.desde_codigo_iso("usd"))        # Moneda.USD
```

## Enums con Valores Compuestos

```python
class Planeta(Enum):
    MERCURIO = (3.3e23, 2.4e6)
    VENUS = (4.9e24, 6.1e6)
    TIERRA = (6.0e24, 6.4e6)

    def __init__(self, masa, radio):
        self.masa = masa
        self.radio = radio

    @property
    def gravedad_superficial(self):
        G = 6.674e-11
        return G * self.masa / (self.radio ** 2)

print(f"{Planeta.TIERRA.gravedad_superficial:.2f} m/s²")  # ~9.8 m/s²
```

## `EnumMeta`: la Metaclase por Debajo

Volviendo a [[38 - Metaclases y type()|38]] — la sintaxis especial de `Enum` (iterar con `for`, buscar por valor con `Color(1)`, impedir crear nuevas instancias fuera de las declaradas) es posible gracias a que `Enum` usa `EnumMeta` (llamada `EnumType` desde 3.11) como metaclase, en vez de `type` directamente:

```python
from enum import Enum, EnumMeta

print(type(Color))  # <enum 'EnumMeta'> -> Color no es una clase "normal"

# Intentar crear una instancia "libre" falla, a diferencia de una clase común:
# Color(99)  # ValueError: 99 is not a valid Color
```

## Enums vs Alternativas

| | `Enum` | Constantes sueltas (`ESTADO_ACTIVO = 1`) | `Literal["activo", "inactivo"]` (typing) |
|---|---|---|---|
| Namespace agrupado | Sí | No (contaminan el módulo) | No |
| Iterable | Sí | No | No |
| Validación en runtime | Sí (`Color(valor)` lanza si no existe) | No | No (solo en type-checking estático) |
| Validación estática (mypy) | Sí | Parcial | Sí |
| Uso típico | Estados, categorías, configuración fija | Scripts pequeños, casos triviales | Firmas de función con opciones fijas de string |

## Ver también

- [[38 - Metaclases y type()]] — cómo `EnumMeta` habilita la sintaxis especial de `Enum`.
- [[42 - Type Hints y el Módulo Typing]] — `Literal` como alternativa ligera para casos simples.
- [[40 - Dataclasses, NamedTuple y Comparativa con Attrs-Pydantic]] — modelado de datos estructurados, el complemento natural de los enums.
- [[06 - Dictionaries]] — usar enums como claves de diccionario de forma segura (gracias a que son hashables).
