# Funciones Lógicas
(misma lógica de [[Advanced Functions#4. Conditional Functions (_Funciones Condicionales_)|Funciones Condicionales en SQL]])

Las **funciones lógicas** en **DAX** te permiten evaluar condiciones y tomar decisiones basadas en si una expresión es verdadera o falsa. Son la base para crear fórmulas dinámicas que se adaptan a diferentes escenarios, como categorizar datos, manejar errores o crear lógica de negocio compleja.

## IF

La función `IF` es la más fundamental. Evalúa una condición lógica y devuelve un resultado si la condición es verdadera y otro si es falsa. El argumento para el valor si es falso es opcional.

```dax
IF(<condición_lógica>, <valor_si_verdadero>, [<valor_si_falso>])
```

## Operadores Lógicos
(ver [[DAX Operators#Operadores Lógicos|Operadores Lógicos]])

Estas funciones te permiten combinar o invertir condiciones lógicas, lo que es esencial para crear lógica más compleja. Se utilizan comúnmente dentro del primer argumento de una función `IF`.

### AND

Devuelve `TRUE` solo si ambas condiciones son verdaderas.

```dax
AND(<lógica1>, <lógica2>)
```

### OR

Devuelve `TRUE` si al menos una de las condiciones es verdadera.

```dax
OR(<lógica1>, <lógica2>)
```

### NOT

Invierte el valor lógico de una expresión. Convierte `TRUE` en `FALSE` y viceversa.

```dax
NOT(<lógica>)
```

## SWITCH

La función `SWITCH` es una alternativa más limpia y legible a anidar múltiples funciones `IF` dentro de otra. Evalúa una expresión contra una lista de valores y devuelve un resultado si hay una coincidencia.

```dax
SWITCH(<expresión>, <valor1>, <resultado1>, [<valor2>, <resultado2>], ..., [<resultado_si_ninguno>])
```

**Nota:** El uso de `SWITCH(TRUE(), ...)` es una práctica común para evaluar múltiples condiciones en lugar de un solo valor.

## IFERROR

Esta función te permite manejar de forma controlada los errores que pueden surgir en una fórmula, como la división por cero (`#DIV/0!`). Si la expresión principal produce un error, `IFERROR` devuelve el valor que definas.

```dax
IFERROR(<expresión_principal>, <valor_si_hay_error>)
```
