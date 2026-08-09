---
Fecha de creación: 2026-04-24T18:19:00
Materia:
  - Probabilidad y Estadística
Fecha de clase: 2026-04-24
---
[[Clase 12 - Distribución Normal|← Clase anterior]] | [[Clase 14 - Laboratorio 3|Clase siguiente →]]

# Tipos de Distribución
(la [[Clase 12 - Distribución Normal|Distribución Normal]] de la Clase 12 es continua; aquí se ubica dentro del panorama general de distribuciones, incluyendo las discretas. Se profundiza en Binomial y Poisson en [[Clase 01 - Distribuciones Discretas, Binomial y Poisson|Estadística Inferencial, Clase 01]])

Las distribuciones de probabilidad se clasifican según el tipo de variable que modelan.

## Distribuciones Discretas

Modelan variables que solo pueden tomar valores contables (enteros), como conteos.

### Distribución Binomial

Modela el número de "éxitos" en $n$ ensayos independientes, cada uno con la misma probabilidad de éxito $p$ (ej. número de piezas defectuosas en un lote de 20).

$$
P(X=k) = \binom{n}{k}p^k(1-p)^{n-k}
$$

### Distribución de Poisson

Modela el número de veces que ocurre un evento en un intervalo fijo de tiempo o espacio, cuando esos eventos ocurren de forma independiente y a una tasa promedio constante $\lambda$ (ej. número de llamadas que recibe un call center por hora).

$$
P(X=k) = \frac{\lambda^k e^{-\lambda}}{k!}
$$

## Distribuciones Continuas

Modelan variables que pueden tomar cualquier valor dentro de un rango.

### Distribución Uniforme

Todos los valores dentro de un intervalo $[a,b]$ tienen la misma probabilidad de ocurrir. Es el modelo continuo más simple.

### Distribución Exponencial

Modela el tiempo que transcurre entre eventos de un proceso de Poisson (ej. tiempo entre llegadas de clientes a una fila). Muy usada en confiabilidad y análisis de fallas de equipos.

### Distribución Normal
(ver [[Clase 12 - Distribución Normal|Clase 12]])

La más usada de todas: modela fenómenos donde los valores se concentran simétricamente alrededor de un promedio.

## ¿Cómo elegir la distribución correcta?

- ¿La variable es discreta (conteos) o continua (mediciones)? → Discreta y Continua
- ¿Cuento "éxitos" en un número fijo de intentos? → Binomial
- ¿Cuento ocurrencias en un intervalo de tiempo/espacio? → Poisson
- ¿Mido el tiempo hasta que ocurra el próximo evento? → Exponencial
- ¿Los datos se agrupan simétricamente alrededor de una media, sin límites definidos? → Normal
