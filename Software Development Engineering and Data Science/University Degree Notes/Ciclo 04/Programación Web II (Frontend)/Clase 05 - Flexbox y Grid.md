---
Fecha de creación: 2026-08-14T18:16:00
Materia:
  - Programación Web II (Frontend)
Fecha de clase: 2026-08-14
---
# Flexbox y Grid

**Flexbox** y **CSS Grid** son dos herramientas de CSS modernas para diseñar páginas web. **Flexbox** funciona en una sola dimensión (en fila o en columna), ideal para alinear elementos y repartir espacios pequeños. **Grid** funciona en dos dimensiones (filas y columnas a la vez), perfecto para estructurar el diseño completo de una pantalla.

---

## Flexbox

**Flexbox** (_Flexible Box Layout_) es un modelo de diseño unidimensional de CSS diseñado para alinear y distribuir espacio entre elementos dentro de un contenedor de manera eficiente. Permite organizar contenido en filas o columnas adaptándose a distintos tamaños de pantalla de forma automática y fluida.

Permite controlar:

- Dirección de los elementos
- Alineación horizontal y vertical
- Distribución del espacio
- Separación entre elementos
- Adaptación de los elementos al espacio disponible.

### Conceptos Fundamentales

Para utilizar Flexbox se trabaja principalmente con:

- **Contenedor:** Elemento al que se aplica display: ``flex``.
- **Eje principal:** Define la dirección en la que se organizan los elementos.
- **Eje secundario:** Permite controlar la alineación perpendicular al eje principal.
- **Elementos Flex:** Son los elementos hijos que se encuentran dentro del contenedor.

### Propiedades Principales

- ``flex-direction``: Define la dirección de los elementos.
- ``justify-content``: Controla la distribución de los elementos sobre el eje principal.
- ``align-items``: Controla la alineación sobre el eje secundario.
- ``flex-wrap``: Permite que los elementos pasen a una nueva línea cuando no existe suficiente espacio.
- ``gap``: Define el espacio entre los elementos.

---

## Grid

**CSS Grid** es un sistema moderno de diseño web en **dos dimensiones** (filas y columnas al mismo tiempo). Permite crear estructuras de páginas complejas y ordenadas de forma muy fácil, usando líneas, celdas y espacios sin necesidad de escribir código difícil o usar tablas antiguas.

Permite controlar:

- Filas
- Columnas
- Espacios entre elementos
- Distribución del contenido
- Posición de los elementos dentro de la estructura.

### Conceptos Fundamentales

- **Grid Container:** Elemento que contiene la estructura de la cuadrícula.
- **Grid Items:** Elementos que forman parte de la cuadrícula.
- **Columnas:** Dividen el espacio verticalmente.
- **Filas:** Dividen el espacio horizontalmente.
- **Gap:** Define el espacio entre filas y columnas.

### Propiedades Principales

- `display: grid`: Activa el modo de diseño de cuadrícula en el elemento.
- `grid-template-columns`: Define el tamaño y el número de las columnas (ej. `repeat(3, 1fr)`).
- `grid-template-rows`: Define el tamaño y el número de las filas.
- `gap` (o `grid-gap`): Establece el espacio de separación entre filas y columnas.
- `grid-template-areas`: Permite diseñar la estructura usando nombres de áreas visuales.

---

## Comparativa: Flexbox vs. CSS Grid

| Flexbox                                            | Grid                                           |
| -------------------------------------------------- | ---------------------------------------------- |
| Trabaja principalmente en una dimensión            | Trabaja en dos dimensiones                     |
| Organiza elementos en una fila o columna           | Organiza elementos mediante filas y columnas   |
| Ideal para menús, botones, tarjetas y alineaciones | Ideal para estructuras de páginas y galerías   |
| Facilita la distribución de elementos              | Permite controlar la posición de los elementos |

---

## Diseño Adaptable

El diseño adaptable (o _responsive design_) es una técnica de desarrollo que ajusta y cambia la apariencia de un sitio web de forma fluida para que se vea y funcione bien en cualquier pantalla, ya sea un teléfono móvil, una tableta o una computadora.

Para lograrlo podemos combinar:

- Flexbox
- CSS Grid
- Unidades relativas
- ``min-width`` y ``max-width``.
- ``flex-wrap``
- Media Queries
- Diseño Mobile First

---

## Componentes Reutilizables

Los componentes reutilizables son piezas de código, diseño o elementos modulares que se crean una sola vez y se usan muchas veces en diferentes partes de un proyecto digital. Ejemplos comunes incluyen botones, barras de navegación, tarjetas de presentación y cuadros de entrada de texto.

**Ejemplos:**

- Tarjetas de libros
- Botones
- Menús de navegación
- Formularios
- Alertas
- Paneles informativos

**Beneficios:**

- Evita repetir código
- Facilita el mantenimiento
- Mantiene una apariencia uniforme
- Facilita la evolución de la aplicación
