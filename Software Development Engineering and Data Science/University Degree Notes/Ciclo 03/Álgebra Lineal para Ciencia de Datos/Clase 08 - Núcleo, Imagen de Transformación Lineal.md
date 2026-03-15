---
Fecha de creación: 2026-03-14T18:02:00
Materia:
  - Probabilidad y Estadística
Fecha de clase: 2026-03-14
---
# Núcleo e Imagen de una Transformación Lineal

Un función T: V -> W entre espacios vectoriales es una transformación lineal si cumple:

- Aditividad: $T(u+v) = T(u) + T(v)$
- Homogeneidad: $T(\alpha v) = \alpha \cdot T(v)$

Ejemplo 1: Rotación en $R^2$



Ejemplo 2: Derivación



Ejemplo 3: Proyección



Representación Matricial
Toda T lineal tiene: $T(x) = A\cdot x$

donde A es la matriz estándar de T

## Núcleo de una Transformación Lineal

$$
Nu(T) = Ker(T) = \{ v \pertenece V : T(v) = 0 \}
$$

### Propiedades del Núcleo

- Subespacio
- Nulidad
- Vector cero
- Inyectividad

## Imagen de una Transformación Lineal

$$
Im(T) = \{ T(v) : v \pertenece V \} = \{ w \pertenece W : \Einvertida v tal que T(v) = w \}
$$

### Propiedades de la Imagen

- Subespacio
- Espacio columna
- Rango
- Sobreyectividad

## Teorema Fundamental

$$
dim(V) = dim(Ker(T) + dim(Im(T)))
$$
$$
N = nulidad(T) + rango(T)
$$

- T: R^3 -> R^4, rango=2
- T: R^4 -> R^2 
- T: R^3 -> R^3, nul=0
- T: R^5 -> R^3

## Propiedades y Clasificación de $T$

### Monomorfismo (Inyectiva)


### Epimorfismo (Sobreyectiva)


### Isomorfismo (Biyectiva)


### Ejercicio 1 - Python

Sea $T(x,y,z) = (x+2y+3z, 2x+4y+6z)$ encontrar el núcleo de $T$ y su dimensión.

```python
import numpy as np
from scipy.linalg import null_space
from sympy import Matrix

# Matríz de la transformada
# Columnas 3, filas 2 (T: R^3 -> R^2)
A = np.array([[1, 2, 3], [2, 4, 6]])

# Núcelo Ax = 0
ker = null_space(A)
print(f"El núcleo es: {ker}")

# Nulidad
nulidad = ker.shape[1]
print(f"La nulidad es: {nulidad}")

# Verificación
M = Matrix([[1, 2, 3], [2, 4, 6]])
print("Resultado:", M.rref())
```

Explicación de la librería ``scipy`` y ``null_space``
Explicación de la librería ``sympy`` y ``Matrix``, ``M.rref()``

Explicación del resultado

### Ejercicio 1 - MATLAB

```matlab
A = [1 2 3; 2 4 6]

% Núcleo
ker = null(A);
fprintf('El núcleo es: ');
disp(ker)

% Nulidad
nulidad = sike(ker, 2);
fprintf('Nulidad: %d', nulidad)

% Verificación
M = rref(A);
fprintf('Resultado: %d', M)

% Rango
rango = rank(A)
fprintf('Rango: %d', rango)
```

### Ejercicio 2 - Python

Sea T con Matriz A = [ ]

```python
import numpy as np
from sympy import Matrix

A = Matrix([[1,2,3,4], [0,1,2,3], [1,3,5,7]])
n = A.shape[1]

```

### Ejercicio 2 - MATLAB

```matlab
% Realizar la Matriz
A = [1 2 0; 0 1 1; 1 3 1]

% Rango de la matriz A = dim de Im(T)
rango = rank(A);
fprintf('Rango = dim(Im(T)) = %d', rango);

% Base
[R, pivote] = rref(A);
fprintf('Pivotes: ');
disp(pivote)
fprintf('La Base de Im(T): ')
disp(A(:, pivote))
```
