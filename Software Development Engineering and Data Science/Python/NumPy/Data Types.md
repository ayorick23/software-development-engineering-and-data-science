# Data Types

## Tipos de Datos

En **NumPy**, los tipos de datos de un array se refieren al tipo de datos que contiene cada elemento del array. NumPy ofrece una variedad de tipos de datos numéricos, incluyendo enteros, flotantes, complejos y booleanos, cada uno con diferentes precisiones. También existen tipos de datos para cadenas y objetos.

```python
import numpy as np
```

### Integers

- ``int8``, ``int16``, ``int32``, ``int64``: Enteros con diferentes tamaños de bits (8, 16, 32 y 64 bits respectivamente), determinando el rango de valores que pueden representar.
- ``uint8``, ``uint16``, ``uint32``, ``uint64``: Enteros sin signo, también con diferentes tamaños de bits.

```python
#Enteros
np.int8, np.int16, np.int32, np.int64
```

    (numpy.int8, numpy.int16, numpy.int32, numpy.int64)

```python
#Enteros sin signo
np.uint8, np.uint16, np.uint32, np.uint64
```

    (numpy.uint8, numpy.uint16, numpy.uint32, numpy.uint64)

### Floats

- ``float16``, ``float32``, ``float64``: Números reales con diferentes niveles de precisión (16, 32, y 64 bits respectivamente). ``float64`` es el tipo de punto flotante más común.

```python
#Flotantes
np.float16, np.float32, np.float64
```

    (numpy.float16, numpy.float32, numpy.float64)

### Complex

- ``complex64``, ``complex128``: Números complejos con diferentes niveles de precisión (64 y 128 bits respectivamente).

```python
#Complejos
np.complex64, np.complex128
```

    (numpy.complex64, numpy.complex128)

### Other types

- ``bool_``: Representa valores booleanos (``True`` o ``False``).
- ``str_``: Para cadenas de texto.
- ``object_``: Puede contener cualquier tipo de objeto Python, no solo datos numéricos.

```python
#Booleanos, cadenas y objetos
np.bool, np.str_, np.object_
```

    (numpy.bool, numpy.str_, numpy.object_)