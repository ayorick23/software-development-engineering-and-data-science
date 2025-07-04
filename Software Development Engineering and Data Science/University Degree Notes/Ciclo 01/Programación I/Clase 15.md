# PROGRAMACIÓN I: clase 15

Fecha de creación: 3 de mayo de 2025 14:29
Clase: PROGRAMACIÓN I
Fecha de la clase: 3 de mayo de 2025

# Pruebas Unitarias

## ¿Qué son las pruebas unitarias?

Las pruebas unitarias son pequeños programas que verifican que una unidad mínima de código (normalmente una función o método) funcione correctamente de manera automática.

Su propósito es:

- Confirmar que una función hace lo que se espera.
- Detectar errores rápidamente.
- Facilitar mantenimiento y cambios futuros sin romper el programa.

<aside>
<img src="notion://custom_emoji/b356a38d-82ed-44c1-bf18-c24d9d5e694a/1b4554ee-ae2e-80a3-bab8-007a8b65ff5e" alt="notion://custom_emoji/b356a38d-82ed-44c1-bf18-c24d9d5e694a/1b4554ee-ae2e-80a3-bab8-007a8b65ff5e" width="40px" />

NOTA: Piensa en las pruebas unitarias como pequeños “guardianes” que protegen cada parte de tu código.

</aside>

## ¿Cómo se hacen pruebas unitarias en Python?

Python incluye el módulo `unittest` en su biblioteca estándar.

```python
import unittest

def suma(a, b):
	return a + b
	
class TestSuma(unittest.TestCase):

	def test_suma_positivos(self):
		self.assertEqual(suma(2, 3), 5)
		
unittest.main()
```

- `TestSuma` es una clase de prueba que hereda de `unittest.TestCase`.
- Cada método que empieza `test_` es una prueba.
- Se utiliza `assertEqual`, `assertTrue`, `assertRaises`, etc., para verificar comportamientos.

## Principales métodos de aserción (`assert`)

| Método | ¿Qué verifica? |
| --- | --- |
| `assertEqual(a, b)` | Que a == b |
| `assertNotEqual(a, b)` | Que a ≠ b |
| `assertTrue(x)` | Que x es `True` |
| `assertFalse(x)` | Que x es `False` |
| `assertIsNone(x)` | Que x es `None` |
| `assertIsInstance(x, Tipo)` | Que x es instancia de `Tipo` |
| `assertRaises(Error)` | Que un error específico es lanzado |

### Buenas prácticas en pruebas unitarias

- Una prueba debe probar una sola cosa.
    - Cada método de prueba debe validar un solo comportamiento.
- Nombre de prueba descriptivo.
    - Usa nombre que expliquen qué están probando
- Prueba lo normal, lo raro y los errores.
    - Casos típicos, casos extremos (ejemplo: cero, negativos) y excepciones.
- Aísla las pruebas.
    - Cada prueba debe ser independiente: no debe depender de otras.
- Usa `setUp` y `tearDown` si necesitas preparar o limpiar datos.

```python
def setUp(self):
	self.user = Usuario("Ana")
	
def tearDown(self):
	del self.user
```

# Módulos en Python

## ¿Qué es un módulo en Python?

Un módulo es simplemente un archivo .py que contiene funciones, clases o variables que podemos importar y reutilizar en otros programas. Los módulos ayudan a organizar y reutilizar el código.

## ¿Cómo crear un módulo?

Por ejemplo, crea un archivo llamado **matemáticas.py**:

```python
def suma(a, b):
	return a + b
	
def resta(a, b):
	return a - b
```

### Usa el módulo en otro archivo

Archivo **programa.py**:

```python
import matematica

print(matematica.suma(5, 3)) #resultado: 8
print(matematica.resta(10, 4)) #resultado: 6
```

### Importaciones específicas

También puedes importar solo lo que necesites:

```python
from matematica import suma

print(suma(7, 2)) # 9
```