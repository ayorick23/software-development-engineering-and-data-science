# 45 - Estilo Pythonico y Buenas Prácticas

> Continúa de [[44 - Linters y Formateadores|44]] y cierra el bloque de tipado y calidad. Aquí se recopilan los principios de diseño detrás de Python — el "por qué" que justifica muchas de las convenciones que las herramientas anteriores automatizan.

## El Zen de Python

Está literalmente incluido en el intérprete:

```python
import this
```

```text
Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
...
There should be one-- and preferably only one --obvious way to do it.
...
```

No son reglas técnicas, sino **valores de diseño** — cuando dos formas de resolver algo compiten, el Zen orienta cuál elegir. Muchas de las secciones siguientes son aplicaciones concretas de estos principios.

## PEP 8: la Guía de Estilo Oficial

PEP 8 es la convención de estilo estándar de la comunidad. Los puntos que más impactan la legibilidad diaria:

| Regla | Ejemplo correcto |
|---|---|
| Indentación: 4 espacios (nunca tabs) | `if x:\n    return x` |
| Nombres de variables/funciones: `snake_case` | `def calcular_total():` |
| Nombres de clases: `PascalCase` | `class GestorDePedidos:` |
| Constantes: `MAYUSCULAS_CON_GUIONES` | `MAX_INTENTOS = 3` |
| Longitud de línea: máximo 79-99 caracteres (equipos suelen usar 88-100) | Configurable en `ruff`/`black` |
| Dos líneas en blanco entre definiciones de nivel superior | Entre `def`/`class` en el módulo |
| Espacios alrededor de operadores | `x = a + b`, no `x=a+b` |

En la práctica, **no memorices PEP 8 al detalle** — deja que `ruff`/`black` (ver [[44 - Linters y Formateadores|44]]) lo apliquen automáticamente. Vale la pena conocerlo para entender **por qué** el formateador toma las decisiones que toma.

## EAFP vs LBYL: Dos Filosofías de Manejo de Errores

Python favorece culturalmente **EAFP** (_Easier to Ask Forgiveness than Permission_) sobre **LBYL** (_Look Before You Leap_):

```python
# LBYL: verificar condiciones ANTES de actuar
if "clave" in diccionario:
    valor = diccionario["clave"]
else:
    valor = None

# EAFP: intentar directamente y manejar el error si ocurre (más "pythonico")
try:
    valor = diccionario["clave"]
except KeyError:
    valor = None
```

```python
# LBYL con archivos: existe una condición de carrera entre el chequeo y el uso
import os
if os.path.exists(ruta):
    with open(ruta) as f:
        contenido = f.read()

# EAFP: evita la condición de carrera (el archivo podría desaparecer entre el "if" y el "open")
try:
    with open(ruta) as f:
        contenido = f.read()
except FileNotFoundError:
    contenido = None
```

| | LBYL | EAFP |
|---|---|---|
| Rendimiento en el caso "feliz" (sin error) | Paga el costo de la verificación siempre | Sin costo extra si no hay excepción |
| Rendimiento cuando el error es frecuente | Mejor (evita el overhead de lanzar excepciones) | Peor (excepciones tienen costo, ver [[49 - Excepciones Avanzadas|49]]) |
| Condiciones de carrera (multi-threading, sistema de archivos) | Vulnerable | Más seguro |
| Idiomático en Python | Menos común | Preferido culturalmente |

## Comprensiones vs `map`/`filter`/Loops Explícitos

Ya se tocó en [[31 - Funciones de Alto Orden y Programación Funcional|31]] — la regla general:

```python
# Preferible: comprensión, clara y "pythonica"
cuadrados_pares = [x**2 for x in range(20) if x % 2 == 0]

# Evitar cuando la lógica se complica demasiado dentro de la comprensión
# (esto ya es difícil de leer -> mejor un loop explícito con nombres claros)
resultado = [f(x) for x in datos if g(x) and h(x) for y in x.items() if k(y)]
```

**Regla práctica:** si una comprensión no cabe cómodamente en una línea o dos, o tiene más de un `for`/`if` anidado, un `for` tradicional con nombres descriptivos suele ser más legible — "Flat is better than nested" y "Readability counts" del Zen aplicados literalmente.

## Nombrado: Claridad sobre Brevedad

```python
# Evitar
d = {}
for i in l:
    if i[0] == "a":
        d[i[1]] = i[2]

# Preferir
resultados_por_id = {}
for fila in filas:
    if fila.categoria == "activo":
        resultados_por_id[fila.id] = fila.valor
```

- Prefijo `_` (un guion bajo): convención de "uso interno" (ver [[17 - OOP|17 - OOP]] para encapsulamiento).
- Sufijo/prefijo `_` para evitar colisión con palabras reservadas: `class_`, `type_`.
- Evita nombres de una sola letra salvo en contadores de loop muy cortos (`i`, `j`) o convenciones matemáticas ya establecidas (`x`, `y` en coordenadas).

## Docstrings: Documentando la Intención, no la Sintaxis

```python
def calcular_descuento(precio: float, porcentaje: float) -> float:
    """Calcula el precio final tras aplicar un descuento porcentual.

    Args:
        precio: Precio original antes de descuento.
        porcentaje: Porcentaje de descuento a aplicar (0-100).

    Returns:
        El precio final con el descuento aplicado.

    Raises:
        ValueError: Si el porcentaje está fuera del rango 0-100.
    """
    if not 0 <= porcentaje <= 100:
        raise ValueError("El porcentaje debe estar entre 0 y 100")
    return precio * (1 - porcentaje / 100)
```

Un docstring útil explica **qué hace y por qué**, no repite lo que el nombre de la función ya deja claro. Herramientas como Sphinx generan documentación automáticamente a partir de estos docstrings — y `help(calcular_descuento)` los muestra directamente en el intérprete.

## Convenciones de Guiones Bajos

| Patrón | Significado |
|---|---|
| `_variable` | Convención de "uso interno" — no forma parte de la API pública (ver [[17 - OOP|17]]) |
| `__variable` | Name mangling dentro de una clase (ver [[17 - OOP|17]]) |
| `__variable__` | "Dunder" — reservado por el lenguaje (`__init__`, `__len__`, ver [[37 - Dunder Methods y Protocolos Avanzados\|37]]) — nunca inventes tus propios nombres dunder |
| `_` (solo guion bajo) | Convención para "valor que deliberadamente se ignora": `for _ in range(3):` o `_, resto = tupla` |

## El Principio de "Explícito es Mejor que Implícito" en la Práctica

```python
# Implícito: ¿qué hace realmente esta función sin leer su cuerpo completo?
def procesar(datos, **kwargs):
    ...

# Explícito: la firma documenta el contrato (ver 32 - Args, Kwargs y Firmas Avanzadas)
def procesar(datos: list[dict], normalizar: bool = True, formato: str = "json") -> dict:
    ...
```

```python
# Implícito: import * contamina el namespace, es imposible saber de dónde viene 'calcular'
from mi_modulo import *
resultado = calcular(5)

# Explícito: se sabe exactamente de dónde viene cada nombre
from mi_modulo import calcular
resultado = calcular(5)
```

`from modulo import *` está desaconsejado en código de producción precisamente por romper este principio — ver [[51 - Módulos e Imports a Fondo|51]] para el detalle completo de imports.

## Checklist Rápido de Revisión de Código Propio

- ¿El nombre de cada función/variable dice lo que hace sin necesitar un comentario adicional?
- ¿Usé EAFP donde tiene sentido (I/O, diccionarios), y LBYL donde el error es la excepción real y no el flujo esperado?
- ¿Las comprensiones siguen siendo legibles en una o dos líneas?
- ¿Evité `from modulo import *`?
- ¿Los docstrings explican el "por qué", no repiten el nombre de la función?
- ¿Dejé que `ruff`/`black` formatearan el código, en vez de discutir estilo manualmente?

## Ver también

- [[44 - Linters y Formateadores]] — cómo automatizar la aplicación de estas convenciones.
- [[49 - Excepciones Avanzadas]] — el trade-off de rendimiento entre EAFP y LBYL, explicado en detalle.
- [[41 - Patrones de Diseño Pythonicos]] — simplicidad aplicada a la elección de patrones de diseño.
- [[51 - Módulos e Imports a Fondo]] — por qué `import *` rompe el principio de explicitud.
