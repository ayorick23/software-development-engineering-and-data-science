---
Fecha de creación: 2025-07-09T20:58:00
Materia:
  - Matemática II
Fecha de clase: 2025-07-09
---
# Derivadas Avanzadas: Funciones Sucesivas y Trascendentes

## Derivadas Sucesivas

Las derivadas sucesivas representan el proceso de derivar repetidamente una función:
- Primera derivada: $f'(x)$ - tasa de cambio
- Segunda derivada: $f''(x)$ - aceleración
- Tercera derivada: $f'''(x)$ - cambio en aceleración
- Enésima derivada: $f(n)(x)$

Dependiendo de la naturaleza de la función, las derivadas pueden continuar indefinidamente o alcanzar un valor constante (o cero).

## Funciones Trascendentes

Las funciones trascendentes son aquellas que no pueden expresarse mediante operaciones algebraicas finitas (no son polinómicas ni algebraicas).
- Exponenciales
	- $f(x) = ex, ax$
	- Crecimiento/decremento a ritmo proporcional a su valor
- Logarítmicas
	- $f(x) = ln(x), loga(x)$
	- Inversas de las exponenciales, crecimiento lento
- Trigonométricas
	- $sin(x), cos(x), tan(x)$
	- Modelan fenómenos periódicos y ondulatorios

## Derivadas de Funciones Trascendentes

| Función  | Derivada      | Ejemplo                                 |
| -------- | ------------- | --------------------------------------- |
| $e^x$    | $e^x$         | La única función igual a su derivada    |
| $ax$     | $ax.ln(a)$    | $\frac{d}{dx}(2x)=2x.ln(2)$             |
| $ln(x)$  | $\frac{1}{x}$ | $\frac{d}{dx}[ln(3x)]=\frac{1}{3x}.3=1$ |
| $sin(x)$ | $cos(x)$      | $\frac{d}{dx}[sin(2x)]=cos(2x).2$       |
| $cos(x)$ | $-sin(x)$     | $\frac{d}{dx}[cos(x^2)]=-sin(x^2).2x$   |

## Derivadas de Funciones Trigonométricas Inversas

Las funciones trigonométricas inversas son operaciones que "deshacen" las funciones trigonométricas estándar. Sus derivadas se obtienen aplicando la regla de la función inversa:

$$
\frac{d}{dx}f^-1(x)=\frac{1}{f'(f^-1(x))}
$$

Estas derivadas son fundamentales en cálculo integral, donde aparecen en patrones de sustitución frecuentes.

## Tabla: Derivadas Trigonométricas Inversas

1. Derivada del Arcoseno (arcsin)
$$
\frac{d}{dx}arcsin(x)=\frac{1}{\sqrt(1-x^2)}
$$
Válida para $|x|<1$. Importante en integrales que involucran raíces cuadradas.

2. Derivada del Arcocoseno (arccos)
$$
\frac{d}{dx}arccos(x)=-\frac{1}{\sqrt(1-x^2)}
$$
Observe el signo negativo comparado con $arcsin$.

3. Derivada del Arcotangente (arctan)
$$
\frac{d}{dx}arctan(x)=\frac{1}{1+x^2}
$$
Válida para todos los valores reales de $x$. Muy útil en integrales racionales.