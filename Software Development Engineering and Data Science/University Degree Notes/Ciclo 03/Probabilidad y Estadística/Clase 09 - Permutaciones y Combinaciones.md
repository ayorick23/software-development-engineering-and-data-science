---
Fecha de creación: 2026-03-20T18:05:00
Materia:
  - Probabilidad y Estadística
Fecha de clase: 2026-03-20
---
[[Clase 08 - Introducción a la Probabilidad|← Clase anterior]] | [[Clase 10 - Laboratorio 2|Clase siguiente →]]

# Permutaciones y Combinaciones

Las **permutaciones** y **combinaciones** son herramientas fundamentales de la **probabilidad y la estadística** que permiten contar cuántas formas existen de organizar o seleccionar elementos de un conjunto.

Se utilizan ampliamente en:

- Probabilidad
- Ciencia de datos
- Criptografía
- Algoritmos de búsqueda
- Machine Learning
- Análisis de experimentos

## Permutaciones

Una **permutación** es una forma de **ordenar elementos donde el orden sí importa**.

Si el orden cambia, se considera un resultado diferente.

**Ejemplo:**

Con los elementos:

```
A, B, C
```

Las permutaciones posibles son:

```
ABC
ACB
BAC
BCA
CAB
CBA
```

Total = **6 formas**

### Fórmula de Permutaciones

Si tenemos **n elementos** y queremos ordenar **r elementos**, la fórmula es:

$$
P(n,r)=\frac{n!}{(n-r)!}​
$$

Donde:

- $n$ = número total de elementos
- $r$ = número de elementos que se van a ordenar
- $!$ = factorial

### Factorial

El **factorial** de un número es el producto de todos los números positivos hasta ese número.

$$
n! = n(n-1)(n-2)...(1)
$$

### Ejemplo de Permutación

Si tenemos **5 estudiantes** y queremos saber de cuántas formas podemos elegir **3 para ocupar los primeros tres lugares de un concurso**.

Datos:

```
n = 5
r = 3
```

Aplicamos la fórmula:

$$
P(5,3) = \frac{5!}{(5-3)!}
$$
$$
P(5,3) = \frac{120}{2} = 60
$$

**Resultado:** Hay 60 posibles ordenamientos. Esto ocurre porque el orden de los puestos importa.

## Permutaciones en Excel

Excel tiene funciones para calcular permutaciones.

### Permutaciones sin repetición

```excel
=PERMUT(número, número_elegido)
```

**Ejemplo:**

```excel
=PERMUT(5,3)
```

**Resultado:**

```excel
60
```

### Permutaciones con repetición

Cuando los elementos **pueden repetirse**.

Fórmula matemática:

$$
n^r
$$

Ejemplo:

Un código de **4 dígitos con números del 0 al 9**.

```
10^4 = 10000 combinaciones
```

En Excel:

```excel
=10^4
```

## Combinaciones

Una **combinación** es una selección de elementos **donde el orden NO importa**.

**Ejemplo:**

Seleccionar **2 estudiantes de un grupo de 3**:

```
A B C
```

Las combinaciones posibles son:

```
AB
AC
BC
```

Observa que:

```
AB = BA
```

Por eso no se cuentan dos veces.

### Fórmula de Combinaciones

$$
!C(n,r)=\frac{n!}{r!(n-r)!}​
$$

Donde:

- $n$ = número total de elementos
- $r$ = elementos seleccionados

### Ejemplo de Combinación

Si hay **10 estudiantes** y queremos formar **equipos de 3 personas**.

Datos:

```
n = 10
r = 3
```

$$
C(10,3)=\frac{10!}{3!7!}​
$$
$$
C(10,3)=120
$$

**Resultado:**

Se pueden formar 120 equipos diferentes.

## Combinaciones en Excel

Excel tiene una función directa.

```excel
=COMBIN(número, número_elegido)
```

**Ejemplo:**

```excel
=COMBIN(10,3)
```

**Resultado:**

```excel
120
```

## Diferencia entre Permutaciones y Combinaciones

| Característica   | Permutaciones        | Combinaciones    |
| ---------------- | -------------------- | ---------------- |
| Importa el orden | Sí                   | No               |
| Ejemplo          | Podio de una carrera | Formar un equipo |
| Fórmula          | $(P(n,r))$           | $(C(n,r))$       |
| Excel            | `PERMUT()`           | `COMBIN()`       |

## Distribución Normal Estándar

La **distribución normal** es una de las distribuciones más importantes en estadística.

Describe fenómenos naturales como:

- Alturas
- Peso
- Errores de medición
- Resultados de exámenes
- Comportamiento de datos en ciencia de datos

Se representa con la famosa **curva de campana**.

## Curva de Campana (Campana de Gauss)

La **Campana de Gauss** describe una distribución donde:

- la mayoría de los datos se concentran cerca del promedio
- los valores extremos son menos frecuentes

Características:

- Simétrica
- Tiene forma de campana
- La media, mediana y moda son iguales
