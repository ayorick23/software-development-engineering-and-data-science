# Derivadas e Integrales Fundamentales

> **Referencia rápida de cálculo diferencial e integral**  
> Estas tablas contienen las fórmulas más utilizadas para resolver ejercicios de derivación e integración en matemáticas y física.

## Derivadas Fundamentales

| Nº  | Función $f(x)$          | Derivada $f'(x)$           | Observaciones             |
| --- | ----------------------- | -------------------------- | ------------------------- |
| 1   | $c$                     | $0$                        | Derivada de una constante |
| 2   | $x$                     | $1$                        | Lineal                    |
| 3   | $x^n$                   | $n x^{n-1}$                | Potencia ($n ≠ 0$)        |
| 4   | $\sqrt{x} = x^{1/2}$    | $\dfrac{1}{2\sqrt{x}}$     | Raíz cuadrada             |
| 5   | $\dfrac{1}{x} = x^{-1}$ | $-\dfrac{1}{x^2}$          | Inversa                   |
| 6   | $e^x$                   | $e^x$                      | Exponencial natural       |
| 7   | $a^x$                   | $a^x \ln(a)$               | Exponencial base $a$      |
| 8   | $\ln(x)$                | $\dfrac{1}{x}$             | Logaritmo natural         |
| 9   | $\log_a(x)$             | $\dfrac{1}{x\ln(a)}$       | Logaritmo base $a$        |
| 10  | $\sin(x)$               | $\cos(x)$                  | Trigonométrica            |
| 11  | $\cos(x)$               | $-\sin(x)$                 | Trigonométrica            |
| 12  | $\tan(x)$               | $\sec^2(x)$                | Trigonométrica            |
| 13  | $\cot(x)$               | $-\csc^2(x)$               | Trigonométrica            |
| 14  | $\sec(x)$               | $\sec(x)\tan(x)$           | Trigonométrica            |
| 15  | $csc(x)$                | $-\csc(x)\cot(x)$          | Trigonométrica            |
| 16  | $\arcsin(x)$            | $\dfrac{1}{\sqrt{1-x^2}}$  | Inversa trigonométrica    |
| 17  | $\arccos(x)$            | $-\dfrac{1}{\sqrt{1-x^2}}$ | Inversa trigonométrica    |
| 18  | $\arctan(x)$            | $\dfrac{1}{1+x^2}$         | Inversa trigonométrica    |
| 19  | $\text{arccot}(x)$      | $-\dfrac{1}{1+x^2}$        | Inversa trigonométrica    |
| 20  | $f(x) \cdot g(x)$       | $f'g + fg'$                | Regla del producto        |
| 21  | $\dfrac{f(x)}{g(x)}$    | $\dfrac{f'g - fg'}{g^2}$   | Regla del cociente        |
| 22  | $f(g(x))$               | $f'(g(x)) \cdot g'(x)$     | Regla de la cadena        |

## Integrales Fundamentales

| Nº  | Función $f(x)$                | Integral $\displaystyle \int f(x)\,dx$ | Observaciones              |
| --- | ----------------------------- | -------------------------------------- | -------------------------- |
| 1   | $0$                           | $C$                                    | Constante de integración   |
| 2   | $1$                           | $x + C$                                | Constante unitaria         |
| 3   | $x^n$                         | $\dfrac{x^{n+1}}{n+1} + C$             | Potencia ($n \neq -1$)     |
| 4   | $\dfrac{1}{x}$                | $\ln (x) + C$                          | Logarítmica                |
| 5   | $e^x$                         | $e^x + C$                              | Exponencial natural        |
| 6   | $a^x$                         | $\dfrac{a^x}{\ln(a)} + C$              | Exponencial base ($a$)     |
| 7   | $\sin(x)$                     | $-\cos(x) + C$                         | Trigonométrica             |
| 8   | $\cos(x)$                     | $\sin(x) + C$                          | Trigonométrica             |
| 9   | $\tan(x)$                     | $-\ln\cos(x)+ C$                       | Trigonométrica             |
| 10  | \( \cot(x) \)                 | $\ln\sin(x)+ C$                        | Trigonométrica             |
| 11  | \( \sec(x) \)                 | $\ln\sec(x) + \tan(x) + C$             | Trigonométrica             |
| 12  | \( \csc(x) \)                 | $-\ln\csc(x) + \cot(x) + C$            | Trigonométrica             |
| 13  | $\dfrac{1}{\sqrt{1-x^2}}$     | $\arcsin(x) + C$                       | Inversa trigonométrica     |
| 14  | $\dfrac{1}{1+x^2}$            | $\arctan(x) + C$                       | Inversa trigonométrica     |
| 15  | $\dfrac{1}{x\sqrt{x^2-1}}$    | $\arcsec(x) + C$                       | Inversa trigonométrica     |
| 16  | $e^{u(x)} u'(x)$              | $e^{u(x)} + C$                         | Regla de la cadena         |
| 17  | $\dfrac{u'(x)}{u(x)}$         | $\ln u(x) + C$                         | Logarítmica compuesta      |
| 18  | $\cosh(x)$                    | $\sinh(x) + C$                         | Hiperbólica                |
| 19  | $\sinh(x)$                    | $\cosh(x) + C$                         | Hiperbólica                |
| 20  | $\dfrac{1}{\sqrt{x^2 + a^2}}$ | $\ln x + \sqrt{x^2 + a^2} + C$         | Sustitución trigonométrica |

## Integración por Partes

### Fórmula general

$$
\int u \, dv = u v - \int v \, du
$$

>Un Día Vi = Una Vaca Sin Cola Vestida De Uniforme

### Regla **ILATE / LIATE**

Sirve para elegir qué función será ($u$) y cuál será ($dv$).

| Prioridad | Tipo de función        | Ejemplo                    |
| --------- | ---------------------- | -------------------------- |
| **I**     | Inversa Trigonométrica | ($\arctan x, \arcsin x$)   |
| **L**     | Logarítmica            | ($\ln x, \log_a x$)        |
| **A**     | Algebraica             | ($x^2, 3x, (x+1)$)         |
| **T**     | Trigonométrica         | ($\sin x, \cos x, \tan x$) |
| **E**     | Exponencial            | ($e^x, a^x$)               |

## Integración por Fracciones Parciales

Se aplica cuando el integrando es una **fracción racional**:

$$
\frac{P(x)}{Q(x)}
$$
donde \(P(x)\) y \(Q(x)\) son polinomios y ($\deg(P) < \deg(Q)$).

### Pasos generales

1. **Verificar grados:**  
   Si ($\deg(P) \ge \deg(Q)$), primero realiza la división de polinomios.  
2. **Factorizar el denominador** \(Q(x)\).  
3. **Escribir la descomposición** según el tipo de factores.  
4. **Determinar las constantes** comparando coeficientes.  
5. **Integrar cada término** fácilmente.

---

### 🔹 Tipos de factores y sus formas

| Tipo de factor                                           | Forma del término parcial                                                           | Ejemplo                    |
| -------------------------------------------------------- | ----------------------------------------------------------------------------------- | -------------------------- |
| **Lineal simple** ($(x - a)$)                            | ($\dfrac{A}{x - a}$)                                                                | ($\dfrac{1}{x - 2}$)       |
| **Lineal repetido** ($(x - a)^n$)                        | ($\dfrac{A_1}{x - a} + \dfrac{A_2}{(x - a)^2} + \dots + \dfrac{A_n}{(x - a)^n}$)    | ($\dfrac{1}{(x - 1)^2}$)   |
| **Cuadrático irreducible** ($(x^2 + bx + c)$)            | ($\dfrac{Ax + B}{x^2 + bx + c}$)                                                    | ($\dfrac{1}{x^2 + 1}$)     |
| **Cuadrático irreducible repetido** ($(x^2 + bx + c)^n$) | ($\dfrac{A_1x + B_1}{x^2 + bx + c} + \dots + \dfrac{A_nx + B_n}{(x^2 + bx + c)^n}$) | ($\dfrac{1}{(x^2 + 4)^2}$) |

### 🔹 Ejemplo 1: Denominador lineal simple

$$
\int \frac{5x + 2}{x^2 + 3x + 2} dx
$$

**Paso 1:** Factorizar denominador:
$$
x^2 + 3x + 2 = (x + 1)(x + 2)
$$

**Paso 2:** Descomponer:
$$
\frac{5x + 2}{(x + 1)(x + 2)} = \frac{A}{x + 1} + \frac{B}{x + 2}
$$

**Paso 3:** Igualar y resolver:
$$
5x + 2 = A(x + 2) + B(x + 1)
$$

$$
\Rightarrow
\begin{cases}
A + B = 5 \\
2A + B = 2
\end{cases}
\Rightarrow A = -3, \, B = 8
$$

**Paso 4:** Integrar:
$$
\int \frac{5x + 2}{x^2 + 3x + 2} dx = -3 \ln|x + 1| + 8 \ln|x + 2| + C
$$

### Ejemplo 2: Cuadrático irreducible

$$
\int \frac{x + 2}{x^2 + 4} dx
$$

Separar:
$$
\int \frac{x}{x^2 + 4} dx + 2 \int \frac{1}{x^2 + 4} dx
$$

$$
\Rightarrow \frac{1}{2} \ln|x^2 + 4| + \tan^{-1}\left(\frac{x}{2}\right) + C
$$

### Ejemplo 3: Lineal repetido

$$
\int \frac{1}{(x-1)^2 (x+2)} dx
$$

$$
\frac{1}{(x-1)^2 (x+2)} = \frac{A}{x+2} + \frac{B}{x-1} + \frac{C}{(x-1)^2}
$$

Luego se determinan \(A, B, C\) y se integran los términos logarítmicos y potencias.

## Consejos finales

- Si la integral tiene **producto de funciones**, intenta primero **por partes**.  
- Si tiene **fracción racional**, usa **fracciones parciales**.  
- En algunos casos se combinan ambos métodos.  
- Recuerda derivadas típicas:

$$
\frac{d}{dx}(\ln|x|) = \frac{1}{x}, \quad 
\frac{d}{dx}(\tan^{-1}x) = \frac{1}{1+x^2}
$$

## ✍️ Resumen visual rápido

| Situación                | Método sugerido        | Fórmula clave                                              |
| ------------------------ | ---------------------- | ---------------------------------------------------------- |
| Producto $(u \cdot v')$  | Integración por partes | $(\int u\,dv = uv - \int v\,du)$                           |
| Fracción racional simple | Fracciones parciales   | $(\frac{P(x)}{Q(x)} \rightarrow \sum \frac{A_i}{x - a_i})$ |
| Cuadrático irreducible   | Fracciones parciales   | $(\frac{Ax + B}{x^2 + bx + c})$                            |
| Combinación de ambos     | Mezclado               | Aplica ambos métodos si es necesario                       |
