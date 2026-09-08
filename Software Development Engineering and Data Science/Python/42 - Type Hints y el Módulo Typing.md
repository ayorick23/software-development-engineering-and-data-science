# 42 - Type Hints y el Módulo Typing

> Continúa de [[41 - Patrones de Diseño Pythonicos|41]] y abre el bloque de tipado y calidad de código. Python sigue siendo un lenguaje de **tipado dinámico** en runtime — los type hints no cambian eso — pero permiten que herramientas externas ([[43 - Análisis Estático con Mypy y Pyright|mypy/pyright]]) detecten errores de tipo **antes** de ejecutar el código.

## Lo Básico: Anotando Variables y Funciones

```python
nombre: str = "Ana"
edad: int = 25
activo: bool = True
precio: float = 19.99

def saludar(nombre: str, veces: int = 1) -> str:
    return (f"Hola, {nombre}! " * veces).strip()
```

> **Clave:** estas anotaciones **no se validan en runtime**. `saludar(nombre=123)` se ejecuta sin error — Python las ignora en tiempo de ejecución (solo quedan disponibles vía `__annotations__` para introspección, ver [[29 - Introspección y Metaprogramación|29]]). La validación real la hace una herramienta externa como mypy, o una librería como Pydantic (ver [[40 - Dataclasses, NamedTuple y Comparativa con Attrs-Pydantic|40]]) que sí valida en runtime.

## Tipos Compuestos

```python
from typing import Optional, Union

def procesar(datos: list[int]) -> dict[str, int]:          # sintaxis moderna (3.9+)
    return {"total": sum(datos)}

def buscar(id: int) -> Optional[str]:                       # equivalente a: str | None
    return None

def convertir(valor: Union[int, float]) -> float:            # equivalente a: int | float
    return float(valor)

# Sintaxis moderna con el operador | (Python 3.10+), preferida sobre Optional/Union
def buscar_moderno(id: int) -> str | None:
    return None

def convertir_moderno(valor: int | float) -> float:
    return float(valor)
```

| Sintaxis clásica (`typing`) | Sintaxis moderna (3.9+/3.10+) |
|---|---|
| `List[int]` | `list[int]` |
| `Dict[str, int]` | `dict[str, int]` |
| `Optional[str]` | `str \| None` |
| `Union[int, str]` | `int \| str` |
| `Tuple[int, ...]` | `tuple[int, ...]` |

> Si tu proyecto necesita soportar Python < 3.9, la sintaxis clásica de `typing` sigue siendo necesaria; en código nuevo sobre 3.10+, la sintaxis con `|` y minúsculas es la recomendada.

## `Any`, `NoReturn` y `None`

```python
from typing import Any, NoReturn

def procesar_lo_que_sea(dato: Any) -> Any:
    """Any desactiva el chequeo de tipos para este valor -- úsalo con moderación."""
    return dato

def fallar_siempre() -> NoReturn:
    """Indica que la función NUNCA retorna normalmente (siempre lanza o hace exit)."""
    raise RuntimeError("Esto siempre falla")

def no_retorna_nada() -> None:
    print("Esta función no tiene 'return' explícito con valor")
```

## Genéricos: Funciones y Clases que Funcionan con Cualquier Tipo

```python
from typing import TypeVar, Generic

T = TypeVar("T")

def primero(lista: list[T]) -> T:
    """T se infiere del tipo de entrada: list[int] -> T es int."""
    return lista[0]

print(primero([1, 2, 3]))        # tipado como int
print(primero(["a", "b"]))       # tipado como str

class Pila(Generic[T]):
    def __init__(self):
        self._items: list[T] = []

    def apilar(self, item: T) -> None:
        self._items.append(item)

    def desapilar(self) -> T:
        return self._items.pop()

pila_enteros: Pila[int] = Pila()
pila_enteros.apilar(1)
# pila_enteros.apilar("texto")  # mypy lo marcaría como error de tipo
```

Desde Python 3.12, existe una sintaxis más concisa para genéricos (PEP 695):

```python
def primero_moderno[T](lista: list[T]) -> T:
    return lista[0]

class PilaModerna[T]:
    def __init__(self):
        self._items: list[T] = []
```

## `Protocol`: Tipado Estructural ("Duck Typing" con Chequeo Estático)

Python favorece el _duck typing_ ("si camina como pato y grazna como pato, es un pato"): no importa la clase exacta de un objeto, sino si tiene los métodos/atributos necesarios. `Protocol` permite expresar esto de forma **tipada**, sin requerir herencia explícita.

```python
from typing import Protocol

class TieneArea(Protocol):
    def area(self) -> float: ...

class Circulo:
    def __init__(self, radio):
        self.radio = radio

    def area(self) -> float:
        return 3.14159 * self.radio ** 2

class Cuadrado:
    def __init__(self, lado):
        self.lado = lado

    def area(self) -> float:
        return self.lado ** 2

def imprimir_area(figura: TieneArea) -> None:
    print(f"Área: {figura.area()}")

imprimir_area(Circulo(5))    # OK -> Circulo tiene .area(), aunque NO herede de TieneArea
imprimir_area(Cuadrado(4))   # OK -> mismo caso
```

A diferencia de una clase abstracta (`ABC`, ver [[17 - OOP|17 - OOP]]), que requiere **herencia explícita**, `Protocol` valida la forma (_shape_) del objeto — cualquier clase que tenga un método `area() -> float` satisface `TieneArea`, sin necesidad de declararlo. Esto es tipado estructural, similar a las interfaces de Go o TypeScript.

## `TypedDict`: Tipando Diccionarios con Forma Fija

```python
from typing import TypedDict

class Usuario(TypedDict):
    nombre: str
    edad: int
    activo: bool

def crear_usuario(datos: Usuario) -> None:
    print(datos["nombre"])

usuario: Usuario = {"nombre": "Ana", "edad": 25, "activo": True}
# usuario_invalido: Usuario = {"nombre": "Ana"}  # mypy: faltan claves 'edad' y 'activo'
```

`TypedDict` es útil cuando trabajas con diccionarios de forma fija que vienen de JSON/APIs (ver [[14 - JSON and YAML|14]]) y quieres chequeo estático sin pagar el costo de crear una clase completa o un modelo de Pydantic.

## `Literal`: Restringiendo a Valores Específicos

```python
from typing import Literal

def establecer_modo(modo: Literal["lectura", "escritura", "append"]) -> None:
    ...

establecer_modo("lectura")     # OK
# establecer_modo("borrar")   # mypy: "borrar" no es un valor válido
```

Comparar con [[39 - Enums Avanzados|39]]: `Literal` es más ligero para casos simples de validación estática; `Enum` es preferible cuando necesitas iterar los valores, agregarles métodos, o validar en **runtime**.

## `@overload`: Firmas Múltiples para una Sola Función

```python
from typing import overload

@overload
def procesar(dato: int) -> str: ...
@overload
def procesar(dato: str) -> int: ...

def procesar(dato):
    if isinstance(dato, int):
        return str(dato)
    return len(dato)

# mypy sabe que procesar(5) devuelve str, y procesar("hola") devuelve int
```

`@overload` documenta para el type-checker (no en runtime — las firmas decoradas con `@overload` nunca se ejecutan) que la relación entre tipo de entrada y tipo de salida varía. Complementa a `functools.singledispatch` (ver [[33 - Functools en Profundidad|33]]), que resuelve el mismo problema pero a nivel de **comportamiento** real, no solo de tipos.

## `TYPE_CHECKING`: Evitando Importaciones Circulares Solo para Tipos

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from mi_paquete.modelos import Usuario   # solo se importa durante el chequeo estático

def procesar_usuario(usuario: "Usuario") -> None:  # anotación como string ("forward reference")
    ...
```

Útil cuando dos módulos necesitan referenciarse mutuamente solo para anotaciones de tipo — evita el `ImportError` circular en runtime (ver [[51 - Módulos e Imports a Fondo|51]] para el problema general de imports circulares).

## Ver también

- [[43 - Análisis Estático con Mypy y Pyright]] — la herramienta que realmente aprovecha todo lo anotado aquí.
- [[40 - Dataclasses, NamedTuple y Comparativa con Attrs-Pydantic]] — Pydantic como la capa que sí valida tipos en runtime.
- [[39 - Enums Avanzados]] — la alternativa con validación en runtime a `Literal`.
- `Python/FastAPI/03 - Request Body y Modelos Pydantic.md` — type hints llevados a su máxima utilidad práctica.
