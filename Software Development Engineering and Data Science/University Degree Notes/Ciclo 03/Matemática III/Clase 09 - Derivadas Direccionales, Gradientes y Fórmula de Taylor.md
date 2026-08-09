---
Fecha de creación: 2026-03-18T18:14:00
Materia:
  - Matemática III
Fecha de clase: 2026-03-18
---
[[Clase 08 - Regla de la Cadena, Derivación Implícita y Gradiente|← Clase anterior]] | [[Clase 10 - Multiplicadores de Lagrange|Clase siguiente →]]

# Derivadas Direccionales, Gradientes y Fórmula de Taylor

## Derivada Direccional

La derivada direccional de $f(x, y)$ en el punto ($x_0, y_0$) en la dirección de un **vector unitario** $\vec{u} = ⟨a, b⟩$, se denota como $D_{\vec{u}}f(x_0, y_0)$ y se define mediante el límite:

$$
D_{\vec{u}}f(x_0, y_0) = \lim_{h\to0} \frac{f(x_0+ha,y_0+hb) - f(x_0,y_0)}{h}
$$

siempre que este límite exista. Representa la pendiente de la superficie (tasa de cambio) si la cruzamos con un plano vertical inclinado en la dirección exacta de $\vec{u}$.

### Cálculo Práctico

Si $f$ es una función diferenciable de $x$ e $y$, entonces f tiene una derivada direccional en la dirección de cualquier vector unitario $\vec{u} = ⟨a, b⟩$, y está dada por el producto punto entre el vector de derivadas parciales y el vector dirección:

$$
D_{\vec{u}}f(x,y) = \nabla f(x,y) \cdot \vec{u} = \frac{\partial f}{\partial x}a + \frac{\partial f}{\partial y}b
$$

donde $\nabla f(x,y) = \left\langle \dfrac{\partial f}{\partial x}, \dfrac{\partial f}{\partial y} \right\rangle$ es el [[Clase 08 - Regla de la Cadena, Derivación Implícita y Gradiente#Gradiente|Gradiente]] visto en la Clase 08.

**Importante:** $\vec{u}$ debe ser unitario ($|\vec{u}| = 1$). Si se da un vector de dirección $\vec{w}$ que no lo es, primero hay que normalizarlo: $\vec{u} = \dfrac{\vec{w}}{|\vec{w}|}$.

### Dirección de Máximo Crecimiento

De la fórmula anterior y de la definición del producto punto ($\nabla f \cdot \vec{u} = |\nabla f||\vec{u}|\cos\theta$, con $|\vec{u}|=1$), se deduce que $D_{\vec{u}}f$ es máxima cuando $\cos\theta = 1$, es decir, cuando $\vec{u}$ apunta en la misma dirección que $\nabla f$. Por eso el gradiente señala la dirección de mayor crecimiento de $f$, y su magnitud $|\nabla f|$ es esa tasa máxima de cambio. En la dirección opuesta ($-\nabla f$) está el descenso más pronunciado — la idea detrás del **descenso de gradiente** en machine learning.

## Fórmula de Taylor (Segundo Orden)

Así como en una variable la Fórmula de Taylor aproxima $f(x)$ cerca de un punto usando sus derivadas, en dos variables se aproxima $f(x,y)$ cerca de $(x_0,y_0)$ usando sus derivadas parciales. La aproximación de segundo orden es:

$$
f(x,y) \approx f(x_0,y_0) + \nabla f(x_0,y_0)\cdot(x-x_0,\,y-y_0)
$$
$$
{}+ \frac{1}{2}\Big[f_{xx}(x_0,y_0)(x-x_0)^2 + 2f_{xy}(x_0,y_0)(x-x_0)(y-y_0) + f_{yy}(x_0,y_0)(y-y_0)^2\Big]
$$

- El primer término, $f(x_0,y_0)$, es el valor de la función en el punto.
- El segundo término, con el gradiente, es la aproximación **lineal** (el plano tangente).
- El tercer término, con las segundas derivadas parciales, corrige esa aproximación lineal usando la curvatura de la superficie — es la misma matriz de segundas derivadas ($f_{xx}, f_{xy}, f_{yy}$) que forma la **Matriz Hessiana**, usada en la Clase 10 para clasificar puntos críticos.

Esta aproximación es la base de métodos numéricos de optimización de segundo orden (como el método de Newton multivariable), que usan esta expansión cuadrática para converger más rápido que el descenso de gradiente puro.