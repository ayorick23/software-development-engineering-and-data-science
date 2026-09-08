# 37 - Dunder Methods y Protocolos Avanzados

> Continúa de [[36 - Descriptors|36]]. [[17 - OOP|17 - OOP]] presentó los dunder methods más comunes (`__init__`, `__str__`, `__len__`); aquí se cubre el catálogo completo por categoría, para que tus propias clases puedan integrarse naturalmente con la sintaxis nativa de Python.

Los métodos "dunder" (_double underscore_) son el mecanismo mediante el cual Python permite que **operadores y funciones built-in** funcionen sobre objetos definidos por el usuario. Cuando escribes `a + b`, Python internamente busca `type(a).__add__(a, b)`.

## Representación de Objetos

```python
class Punto:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __repr__(self):
        """Representación inequívoca, pensada para desarrolladores/debugging."""
        return f"Punto(x={self.x}, y={self.y})"

    def __str__(self):
        """Representación legible, pensada para el usuario final."""
        return f"({self.x}, {self.y})"

p = Punto(1, 2)
print(p)          # usa __str__: (1, 2)
print([p])        # usa __repr__ dentro de contenedores: [Punto(x=1, y=2)]
print(repr(p))    # Punto(x=1, y=2)
```

> **Regla práctica:** siempre implementa `__repr__` (idealmente que sea código Python válido para recrear el objeto). `__str__` es opcional — si no lo defines, Python usa `__repr__` como respaldo.

## Igualdad y Hash

```python
class Punto:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __eq__(self, otro):
        if not isinstance(otro, Punto):
            return NotImplemented
        return self.x == otro.x and self.y == otro.y

    def __hash__(self):
        return hash((self.x, self.y))

p1, p2 = Punto(1, 2), Punto(1, 2)
print(p1 == p2)              # True (sin __eq__, sería False: compararía identidad)
print({p1, p2})              # {Punto...} -> un solo elemento, gracias a __hash__ consistente
```

> **Regla de oro:** si defines `__eq__`, Python automáticamente pone `__hash__ = None` (el objeto deja de ser hashable) a menos que definas `__hash__` explícitamente. Dos objetos que son `==` **deben** tener el mismo `hash()` — romper esta invariante causa bugs silenciosos en diccionarios y sets (ver [[06 - Dictionaries|06]] y [[07 - Sets|07]]).

Devolver `NotImplemented` (no `False` ni lanzar una excepción) cuando el tipo no es comparable le permite a Python intentar la comparación al revés (`otro.__eq__(self)`) antes de rendirse.

## Comparación de Orden

```python
class Version:
    def __init__(self, numero):
        self.numero = numero

    def __eq__(self, otro):
        return self.numero == otro.numero

    def __lt__(self, otro):
        return self.numero < otro.numero
```

Ver [[33 - Functools en Profundidad|33]] para `@functools.total_ordering`, que genera `__le__`, `__gt__`, `__ge__` automáticamente a partir de `__eq__` + `__lt__`.

## Sobrecarga de Operadores Aritméticos

```python
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y

    def __add__(self, otro):
        return Vector(self.x + otro.x, self.y + otro.y)

    def __sub__(self, otro):
        return Vector(self.x - otro.x, self.y - otro.y)

    def __mul__(self, escalar):
        return Vector(self.x * escalar, self.y * escalar)

    def __rmul__(self, escalar):
        """Se invoca cuando el operador izquierdo NO sabe operar con Vector: 3 * vector."""
        return self.__mul__(escalar)

    def __repr__(self):
        return f"Vector({self.x}, {self.y})"

v1, v2 = Vector(1, 2), Vector(3, 4)
print(v1 + v2)   # Vector(4, 6)
print(v1 * 3)    # Vector(3, 6)  -> usa __mul__
print(3 * v1)    # Vector(3, 6)  -> int no sabe multiplicar por Vector, Python prueba __rmul__
```

| Operador | Método "normal" | Método "reflejado" (`r`) | Método "in-place" (`i`) |
|---|---|---|---|
| `+` | `__add__` | `__radd__` | `__iadd__` (`+=`) |
| `-` | `__sub__` | `__rsub__` | `__isub__` (`-=`) |
| `*` | `__mul__` | `__rmul__` | `__imul__` (`*=`) |
| `/` | `__truediv__` | `__rtruediv__` | `__itruediv__` |
| `//` | `__floordiv__` | `__rfloordiv__` | `__ifloordiv__` |
| `**` | `__pow__` | `__rpow__` | `__ipow__` |

## Protocolo de Contenedor: Que tu Clase se Comporte como una Lista/Dict

```python
class ListaLimitada:
    """Una lista que solo acepta hasta N elementos."""

    def __init__(self, maximo):
        self.maximo = maximo
        self._datos = []

    def __len__(self):
        return len(self._datos)

    def __getitem__(self, indice):
        return self._datos[indice]

    def __setitem__(self, indice, valor):
        self._datos[indice] = valor

    def __delitem__(self, indice):
        del self._datos[indice]

    def __contains__(self, valor):
        return valor in self._datos

    def __iter__(self):
        return iter(self._datos)

    def agregar(self, valor):
        if len(self._datos) >= self.maximo:
            raise OverflowError("Lista llena")
        self._datos.append(valor)

lista = ListaLimitada(maximo=3)
lista.agregar(1)
lista.agregar(2)

print(len(lista))       # 2   -> usa __len__
print(lista[0])         # 1   -> usa __getitem__
print(2 in lista)       # True -> usa __contains__
for x in lista:         # usa __iter__
    print(x)
```

> **Nota:** definir solo `__getitem__` (sin `__iter__`) ya vuelve a un objeto iterable en Python — el intérprete cae en un modo de compatibilidad que llama a `__getitem__(0)`, `__getitem__(1)`, ... hasta capturar `IndexError`. Es un comportamiento heredado de Python 2 que sigue funcionando, pero definir `__iter__` explícitamente es más claro y eficiente.

## `__call__`: Objetos Invocables

```python
class Multiplicador:
    def __init__(self, factor):
        self.factor = factor

    def __call__(self, valor):
        return valor * self.factor

triplicar = Multiplicador(3)
print(triplicar(10))     # 30 -> triplicar(10) es azúcar para triplicar.__call__(10)
print(callable(triplicar))  # True (ver 29 - Introspección y Metaprogramación)
```

Es la base de clases que se usan como decoradores con estado y de objetos "funcionales" en frameworks de ML (transformadores con `fit`/`transform`, o modelos con `forward` en PyTorch).

## Protocolo de Context Manager: `__enter__` / `__exit__`

```python
class Temporizador:
    def __enter__(self):
        import time
        self.inicio = time.perf_counter()
        return self

    def __exit__(self, tipo_excepcion, valor_excepcion, traceback):
        import time
        print(f"Tiempo transcurrido: {time.perf_counter() - self.inicio:.4f}s")
        return False   # False/None propaga la excepción; True la suprime

with Temporizador():
    sum(range(10_000_000))
```

Se profundiza en este protocolo, incluyendo el manejo fino de excepciones en `__exit__`, en [[50 - Context Managers Avanzados|50]] (la base ya se vio en [[15 - Context Manager|15]]).

## Tabla de Referencia por Categoría

| Categoría | Métodos |
|---|---|
| Representación | `__repr__`, `__str__`, `__format__` |
| Igualdad/orden | `__eq__`, `__ne__`, `__lt__`, `__le__`, `__gt__`, `__ge__`, `__hash__` |
| Aritmética | `__add__`, `__sub__`, `__mul__`, `__truediv__`, y sus versiones `r*`/`i*` |
| Contenedor | `__len__`, `__getitem__`, `__setitem__`, `__delitem__`, `__contains__`, `__iter__` |
| Invocable | `__call__` |
| Context manager | `__enter__`, `__exit__` |
| Atributos (descriptors) | `__get__`, `__set__`, `__delete__`, `__set_name__`, `__getattr__`, `__setattr__` |
| Construcción | `__new__`, `__init__`, `__init_subclass__` |
| Booleano | `__bool__` (usado por `if objeto:`) |

## Ver también

- [[17 - OOP]] — la base de dunder methods que este archivo completa.
- [[36 - Descriptors]] — `__get__`/`__set__`/`__set_name__` en profundidad.
- [[33 - Functools en Profundidad]] — `@total_ordering` para no implementar los 6 métodos de comparación a mano.
- [[50 - Context Managers Avanzados]] — `__enter__`/`__exit__` llevado a casos avanzados.
