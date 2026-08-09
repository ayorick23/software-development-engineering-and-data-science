---
Fecha de creación: 2025-10-15T18:02:00
Materia:
  - Matemática II
Fecha de clase: 2025-10-15
---
[[Clase 13 - Integración por Sustitución|← Clase anterior]] | [[Clase 15 - Integración por Partes|Clase siguiente →]]

# Sustitución Trigonométrica
(caso especial de la [[Clase 13 - Integración por Sustitución|Integración por Sustitución]] de la Clase 13, útil cuando el integrando contiene una raíz cuadrada de la forma $\sqrt{a^2-x^2}$, $\sqrt{a^2+x^2}$ o $\sqrt{x^2-a^2}$)

## Trigonometría

### Pitágoras

$$
h = \sqrt{a^2+b^2}
$$

$$
a = \sqrt{h^2-b^2}
$$

### Razones Trigonométricas

SOHCAHTOA

$$
S\frac{O}{H}C\frac{A}{H}T\frac{O}{A}
$$

### Identidades Pitagóricas

A partir de $\sin^2\theta+\cos^2\theta=1$ se derivan las tres identidades que hacen funcionar la sustitución trigonométrica, ya que permiten eliminar la raíz cuadrada:

$$
\sin^2\theta+\cos^2\theta=1
$$

$$
1+\tan^2\theta=\sec^2\theta
$$

$$
\sec^2\theta-1=\tan^2\theta
$$

## Sustitución según la Forma del Integrando

| Forma en el integrando | Sustitución       | Identidad usada             | Resultado tras sustituir |
| ----------------------- | ------------------ | ---------------------------- | -------------------------- |
| $\sqrt{a^2-x^2}$        | $x = a\sin\theta$  | $1-\sin^2\theta=\cos^2\theta$ | $\sqrt{a^2-x^2}=a\cos\theta$ |
| $\sqrt{a^2+x^2}$        | $x = a\tan\theta$  | $1+\tan^2\theta=\sec^2\theta$ | $\sqrt{a^2+x^2}=a\sec\theta$ |
| $\sqrt{x^2-a^2}$        | $x = a\sec\theta$  | $\sec^2\theta-1=\tan^2\theta$ | $\sqrt{x^2-a^2}=a\tan\theta$ |

Después de sustituir e integrar en términos de $\theta$, se usa un triángulo rectángulo (construido a partir de la sustitución elegida, con $a$ y $x$ como catetos/hipotenusa según el caso) para devolver el resultado final a la variable original $x$.

### Derivadas e Integrales

![[Pasted image 20251015182307.png]]

### Complemento de Cuadrados

Cuando el integrando tiene la forma $\sqrt{ax^2+bx+c}$ (sin ser ya un cuadrado perfecto restado o sumado, como $a^2\pm x^2$), primero hay que completar el cuadrado para llevarlo a una de las tres formas de la tabla anterior:

$$
ax^2+bx+c = a\left(x+\frac{b}{2a}\right)^2 + \left(c-\frac{b^2}{4a}\right)
$$

Con esto, la expresión queda en términos de $(x+\frac{b}{2a})^2$ más o menos una constante, lo que permite identificar cuál de las tres sustituciones trigonométricas aplicar.
