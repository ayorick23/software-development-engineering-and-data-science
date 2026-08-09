---
Fecha de creación: 2026-03-14T18:02:00
Materia:
  - Álgebra Lineal para Ciencia de Datos
Fecha de clase: 2026-03-14
---
[[Clase 07 - Geometría de Transformaciones Lineales|← Clase anterior]] | [[Clase 09 - Espacios y Subespacios Vectoriales|Clase siguiente →]]

# Núcleo e Imagen de una Transformación Lineal

Un función T: V -> W entre espacios vectoriales es una transformación lineal si cumple:

- Aditividad: $T(u+v) = T(u) + T(v)$
- Homogeneidad: $T(\alpha v) = \alpha \cdot T(v)$

Ejemplo 1: Rotación en $R^2$
(ver [[Clase 07 - Geometría de Transformaciones Lineales#Tipos de Transformaciones Básicas ($\mathbb{R}^2$)|Rotación]] en la Clase 07)

$T(x,y) = (x\cos\theta - y\sin\theta,\ x\sin\theta + y\cos\theta)$ — rota cada vector un ángulo $\theta$ sin cambiar su magnitud. Es lineal porque rotar la suma de dos vectores da el mismo resultado que sumar sus rotaciones por separado.

Ejemplo 2: Derivación

$T(f) = f'$, es decir, la función que envía a cada función derivable $f$ su derivada $f'$. Es lineal porque $(f+g)' = f' + g'$ y $(cf)' = cf'$ — la derivada de una suma es la suma de las derivadas, y sacar una constante también se preserva. Aquí $V$ y $W$ ya no son $\mathbb{R}^n$, sino espacios de funciones.

Ejemplo 3: Proyección

$T(x,y,z) = (x,y,0)$ — proyecta cualquier vector de $\mathbb{R}^3$ sobre el plano $xy$, "aplastando" la componente $z$. Es lineal, y es un buen ejemplo de una transformación **no inyectiva**: todo punto de la recta vertical que pasa por $(x,y,0)$ se mapea al mismo resultado.


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

$T$ es inyectiva si a cada vector distinto de $V$ le corresponde una imagen distinta en $W$ ($u \neq v \implies T(u) \neq T(v)$). Equivale a que el núcleo sea trivial: $Ker(T) = \{0\}$. Geométricamente, ninguna información se "pierde" al aplicar $T$.

### Epimorfismo (Sobreyectiva)

$T$ es sobreyectiva si todo vector de $W$ es la imagen de al menos un vector de $V$, es decir, $Im(T) = W$. Equivale a que $\text{rango}(T) = \dim(W)$.

### Isomorfismo (Biyectiva)

$T$ es un isomorfismo si es inyectiva y sobreyectiva a la vez (monomorfismo y epimorfismo simultáneamente). Esto significa que $T$ tiene una inversa $T^{-1}$, y que $V$ y $W$ son estructuralmente "el mismo" espacio vectorial, solo con los vectores etiquetados de forma distinta.


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
rango = A.rank()
ker = A.nullspace()
nulidad = len(ker)

print(f"Verificar: {n} = {rango} + {nulidad}")
print(f"Se cumple: {n == rango+nulidad}")
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

### Ejercicio 4 - Python


