---
Fecha de creación: 2026-04-02T15:24:00
Materia:
  - Álgebra Lineal para Ciencia de Datos
Fecha de clase: 2026-03-21
---
[[Clase 08 - Núcleo, Imagen de Transformación Lineal|← Clase anterior]] | [[Clase 10 - Valores y Vectores Propios|Clase siguiente →]]

# Espacios y Subespacios Vectoriales

## Espacio Vectorial

Un espacio vectorial es un conjunto no vacío $V$ de objetos, llamados vectores, en el que están definidas dos operaciones, llamadas suma y multiplicación por escalares (números reales), sujetas a los diez axiomas (o reglas).

1. La suma de $u$ y $v$, denotada mediante $u + v$, está en $V$.
2. $u + v = v + u$.
3. $(u + v) + w = u + (v + w)$.
4. Existe un vector cero $0$ en $V$ tal que $u + 0 = u$.
5. Para cada $u$ en $V$, existe un vector $-u$ en $V$ tal que $u + (-u) = 0$.
6. El múltiplo escalar de $u$ por $c$, denotado mediante $cu$, está en $V$.
7. $c(u+v) = cu+cv$.
8. $(c+d)u = cu + du$.
9. $c(du) = (cd)u$.
10. $1u = u$.

## Subespacio

Un subespacio de un espacio vectorial $V$ es un subconjunto $H$ de $V$ que tiene tres propiedades:

1. El vector cero de V está en H.
2. H es cerrado bajo la suma de vectores. Esto es, para cada $u$ y $v$ en $H$, la suma $u+v$ está en $H$.
3. $H$ es cerrado bajo la multiplicación por escalares. Esto es, para cada $u$ en $H$ y cada escalar $c$, el vector $cu$ está en $H$.
