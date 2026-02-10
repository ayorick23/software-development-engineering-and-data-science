---
Fecha de creación: 2026-01-31T19:19:00
Materia:
  - Álgebra Lineal para Ciencia de Datos
Fecha de clase: 2026-01-31
---
# Matrices y Vectores en Matlab

## Matrices

Una matriz es una forma ordenada de organizar números en filas y columnas, parecida a una tabla. Esta organización nos ayuda a representar información de manera clara y a realizar operaciones que serían muy complicadas si se hicieran de otra forma.

## Vectores

Entidad matemática/física caracterizada por tener **magnitud** y **dirección**, a diferencia de un escalar que solo tiene magnitud.

- **Representación:** Geométricamente como un segmento de línea dirigido con un punto inicial y final.
- **Notación:** En negrita ($a$) o con flecha superior ($\vec{a}$).
- **Norma (Longitud):** Se denota como $\|\mathbf{a}\|$.

$$
\|\mathbf{a}\| = \sqrt{a^2_1 + a^2_2 + ··· + a^2_n}
$$

- **Igualdad:** Dos vectores son iguales si tienen la misma magnitud y dirección, _independientemente de su posición_ en el espacio.

## Operaciones Algebraicas Básicas

### Suma y Resta de Vectores

- **Suma ($a + b$):** Se coloca la cola de $b$ en la cabeza de $a$. El resultado va desde la cola de $a$ hasta la cabeza de $b$.
- **Resta ($a - b$):** Definida como la suma con el vector opuesto: $a + (-b)$. El vector opuesto tiene misma magnitud pero dirección contraria.

### Propiedades Fundamentales

- **Conmutativa:** El orden no altera la suma.

$$
a+b=b+a
$$

- **Asociativa:** Agrupar vectores no cambia el resultado.

$$
(a+b)+c = a+(b+c)
$$

- **Identidad Aditiva:** Existencia del vector cero.

### Multiplicación por Escalar

Operación que multiplica un vector $a$ por un número real $\lambda$ (escalar).

$$
\vec{v} = \lambda a
$$

### Efectos Geométricos sobre el Vector

- **Estiramiento:** La magnitud aumenta, mantiene dirección.

$$
\lambda > 1
$$

- **Compresión:** La magnitud disminuye, mantiene dirección.

$$
0<\lambda<1
$$

- **Inversión:** Cambia al sentido opuesto. Puede estirar o comprimir.

$$
\lambda<0
$$

## Vectores en el Sistema de Coordenadas 3D

### Espacio Cartesiano Tridimensional

El espacio 3D se define por tres ejes ortogonales (perpendiculares) entre sí: eje x (abscisas), eje y (ordenadas) y eje z (cotas), que se interceptan en el origen $(0,0,0)$.

- **Base Estándar (Canónica):** Conjunto de tres vectores unitarios dirigidos a lo largo de los ejes positivos: $i = (1,0,0), j = (0,1,0), k = (0,0,1)$
- **Expansión de un Vector:** Cualquier vector $v$ en $\mathbb{R}^3$ puede expresarse como una combinación lineal única de los vectores de la base estándar: $v=v_1i+v_2j+v_3k$ 
- **Componentes:** Los escalares $v_1, v_2, v_3$ representan las proyecciones del vector sobre cada uno de los ejes coordenados.

## Definición de Vectores en MATLAB

- **Vectores Fila:** Se crean encerando los elementos entre corchetes ``[]`` y separándolos por **espacios** y **comas**.
- **Vectores Columna:** Se separan los elementos por **punto y coma** (`;`) dentro de los corchetes.
- **Transposición:** El operador **comilla simple** (`'`) convierte un vector fila en columna y viceversa.

```matlab
% Crear un vector fila
u = [1 2 3]

% Crear un vector columna
v = [1; 2; 3]

% Transpocisión (fila a columna)
w = u'
```


### Funciones Generadoras en MATLAB

#### Zeros y Ones

``zeros(m,n)`` y ``ones(m,n)`` cran matrices de ceros y unos respectivamente. Fundamentales para inicializar variables antes de bucles.

#### Linspace

``linspace(a,b,n)`` genera ``n`` puntos linealmente espaciados entre ``a`` y ``b``. Es la forma estándar de crear vectores para ejes de gráficos.

#### Logspace

``logspace(a,b,n)`` crea puntos espaciados logarítmicamente entre $(10^a)$ y $(10^b)$, útil para diagramas de Bode o análisis de frecuencia.

```matlab
% Matríz de 1x5 ceros
z = zeros(1,5)

% Matríz de 1x4 unos
u = ones(1,4)

% 5 puntos del 1 al 5
x = linspace(1,5,5)

$ Logarítmico: 10^0 a 10^3 (4 puntos)
y = logspace(0,3,4)
```

### Acceso e Indexación en MATLAB

- **Indexación desde 1:** A diferencia de C++ o Python, **MATLAB comienza a contar desde 1.** El primer elemento de un vector ``u`` se accede como `u(1)`.
- **Funciones de Tamaño:** `length(u` devuelve el número de elementos (longitud máxima). ``size(u)`` devuelve las dimensiones exactas (filas x columnas).

```matlab
u = [10 20 30]

val = u(1) % Primer elemento

% Longitud
len = length(u)

% Dimensiones
dim = size(u)

% Fuera de rango
u(0)
u(4)
```

### Operaciones Elemento a Elemento

- **Operador Punto (``.``):** Para operaciones miembro a miembro, se antepone un punto al operador aritmético convencional. Sin él, MATLAB intenta realizar álgebra matricial.
- **Operadores Principales:**
	- ``.*`` Multiplicación elemento a elemento.
	- ``./`` División elemento a elemento.
	- ``.^`` Potencia elemento a elemento.

```matlab
a = [1 2 3]; b = [4 5 6]

% Potencia elemento a elemento (.^)
a .^ 2

% Multiplicación elemento a elemento (.*)
a .* b

% Comparación: Producto Matricial (*)
a * b' % 1x3 * 3x1 = escalar (producto punto)
```

### Combinaciones Lineales y Span

Un vector $v$ es una combinación lineal de un conjunto de vectores si existen escalares tales que:

$$
v = c_1v_1+c_2v_2+
$$

- **Span (Espacio Generado):** Es el conjunto de _todas_ las combinaciones lineales posibles de un conjunto de vectores.

$$
span\{v_1,v_2\} = \{c_1v_1+c_2v_2:c_1,c_2\mathbb\in{R}\}
$$

- **Interpretación Geométrica:** El span de dos vectores no paralelos en el espacio 3D genera un **plano** infinito que pasa por el origen.
- **Aplicaciones:** Fundamental para determinar si un sistema de ecuaciones lineales tiene solución y para analizar subespacios vectoriales (como el espacio columna).

### Producto Punto y Cruz

El producto punto nos dice cuánto se alinean dos vectores (0 si son perpendiculares).

El producto cruz genera un nuevo vector perpendicular a ambos. Por definición:

$$
a \cdot (a\times b) = 0
$$

```matlab
% Definir vectores en el espacio 3D
a = [1, 3, -4];
b = [-1, 1, -2];

% 1. Producto Punto (Escalar)
res_dot = dot(a,b)

% 2. Producto Cruz (Vectorial)
res_cross = cross(a,b)

% 3. Verificación: El producto cruz es ortogonal
check = dot(a, res_cross)
```
