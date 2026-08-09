---
Fecha de creación: 2026-07-25T14:20:00
Materia:
  - Marcos de Trabajo Ágil
Fecha de clase: 2026-07-25
---
# Metodología Scrum

**Scrum** es un modelo de desarrollo de software para desarrollo ágil de software que se ha expandido a otras industrias.

Es un proceso en el que se aplican de manera regular un conjunto de buenas prácticas para trabajar colaborativamente, en equipo y obtener el mejor resultado posible de proyectos.

## Modelo Tradicional vs. Ágil

- **El Modelo de Boehm:** En el enfoque tradicional Cascada), el costo de implementar cambios crece exponencialmente a medida que el proyecto avanza hacia fases tardías.
- **Riesgo Tardío:** Descubrir que un requerimiento es incorrecto durante la fase de pruebas o despliegue multiplica radicalmente los costos técnicos y de negocio.
- **Mitigación en Scrum:** Scrum aplana esta curva de costo de cambio mediante entregas iterativas, retroalimentación constante del usuario y ciclos cortos de inspección.

---

## Pilares de Scrum

Los tres pilares de Scrum son **transparencia, inspección y adaptación**. Estos elementos se apoyan en el empirismo para tomar decisiones basadas en la experiencia, la realidad y las evidencias observadas, guiando de forma flexible a los equipos ágiles.

### 1. Transparencia

El proceso emergente y el trabajo deben ser visibles para aquellos que realizan el trabajo y para los que lo reciben. Sin transparencia, las decisiones se basan en suposiciones, aumentando el riesgo.

### 2. Inspección

Los artefactos de Scrum y el progreso hacia los objetivos deben inspeccionarse con frecuencia y diligencia para detectar variaciones indeseables o problemas de forma temprana.

### 3. Adaptación

Si la inspección revela que uno o más aspectos se desvían fuera de límites aceptables, el proceso o el material debe ajustarse lo antes posible para minimizar futuras desviaciones.

---

## Valores Fundamentales de Scrum

Scrum no es únicamente un conjunto de procesos mecánicos. Su éxito depende de que las personas encarnen estos cinco valores. Su ausencia genera implementaciones fallidas *Zombie Scrum*).

1. **Compromiso.** El equipo se compromete a lograr sus objetivos y a apoyarse mutuamente de manera profesional.
2. **Foco.** Atención absoluta en el trabajo del Sprint actual para lograr el mayor progreso posible.
3. **Franqueza.** Apertura sobre el trabajo y los desafíos. Sin ocultar los problemas reales que enfrenta el equipo.
4. **Respeto.** Respeto mutuo por ser personas capaces, independientes y con habilidades valiosas.
5. **Coraje.** Valentía para hacer lo correcto, trabajar en problemas difíciles y admitir cuando nos equivocamos.

---

## Roles de Scrum

En Scrum, no existe el rol de Project Manager tradicional. La gestión no recae en una sola persona que dicta órdenes. El control y la responsabilidad se distribuyen estratégicamente entre tres responsabilidades claras para fomentar la autogestión y la agilidad.

El Scrum Team es pequeño (generalmente 10 personas o menos), multifuncional y autogestionado. Deciden internamente quién hace qué, cuándo y cómo. No hay subequipos ni jerarquías dentro del Equipo Scrum.

### Product Owner

Responsable de maximizar el **valor** del producto. Es el único gestor del Product Backlog (el "Qué"). Define la visión y prioriza las necesidades, representando la voz del cliente y del negocio.

### Developers

Los profesionales que realizan el trabajo de crear un Incremento utilizable (el "Cómo"). Son dueños de la calidad, adaptan su plan diariamente Sprint Backlog) y se responsabilizan mutuamente como profesionales.

### Scrum Master

Líder servicial. Responsable de la efectividad del equipo y de establecer Scrum correctamente. Remueve impedimentos, facilita eventos y actúa como coach para asegurar que se vivan los valores.

---

## Límites y Antipatrones

Son restricciones inherentes al marco de trabajo y prácticas incorrectas que parecen útiles pero dañan la productividad, destacando la falta de adaptabilidad a entornos no iterativos, la microgestión y las reuniones convertidas en reportes de estado.

> [!WARNING] PO dictando el "Cómo"
> Cuando el Product Owner le dice a los Developers cómo resolver un problema técnico, invade su autonomía y reduce la calidad.

> [!WARNING] SM actuando como Jefe
> Si el Scrum Master asigna tareas o pide reportes de estado, destruye la autogestión y se convierte en un micro-gestor.

> [!WARNING] Developers ignorando el Valor
> Si construyen lo que les gusta técnicamente pero ignoran la priorización del PO, se pierde el alineamiento con el negocio.

---

## Artefactos

Los **artefactos de scrum** son herramientas esenciales que aportan transparencia, inspección y adaptación al trabajo. Los tres artefactos principales de [Scrum](https://www.scrum.org/resources/scrum-artifacts-0) son el **Product Backlog**, el **Sprint Backlog** y el **Incremento**.

- **Product Backlog.** Lista emergente y ordenada de lo que se necesita para mejorar el producto.
- **Sprint Backlog.** El plan de los Developers: los elementos seleccionados y el plan para entregarlos.
- **Incremento.** Un paso concreto hacia el Objetivo del Producto. Debe ser utilizable y terminado.

### Compromisos de los Artefactos

Cada artefacto de Scrum tiene un compromiso específico que aporta valor y guía: el **Objetivo del Producto** (Product Goal) para la Pila del Producto, el **Objetivo del Sprint** (Sprint Goal) para la Pila del Sprint, y la **Definición de Terminado** (Definition of Done) para el Incremento.

- **Requerimientos vs. PBI.**
	Un Product Backlog Item PBI **no es un requerimiento tradicional cerrado.** Es una invitación a la conversación. Se centra en el valor que recibe el usuario, no en la especificación técnica rígida. Cada artefacto contiene un compromiso para medir el progreso:
	- Product Backlog → **Objetivo del Producto**
	- Sprint Backlog → **Objetivo del Sprint**
- **La "Definition of Done" *DoD*.**
	Es el compromiso asociado al **Incremento**. Es una descripción formal del estado del Incremento cuando cumple con las medidas de calidad exigidas. Si un PBI no cumple con la DoD, **no se puede entregar ni presentar** en la Sprint Review. Asegura que no exista deuda técnica oculta.

---

## Eventos de Scrum

Los cinco eventos oficiales de Scrum son el **Sprint**, la **Planificación del Sprint** y el **Scrum Diario**, los cuales sirven para organizar el trabajo y mantener la transparencia según la [Guía de Scrum de Scrum.org](https://www.scrum.org/resources/introduction-scrum-events).

### El Sprint y el Timeboxing

- **Concepto de Timebox:** Todo en Scrum tiene una duración máxima fija. Esto genera enfoque, reduce el desperdicio en debates interminables y asegura un ritmo predecible.
- **El Sprint:** Es el corazón de Scrum. Es un contenedor (timebox de 1 mes o menos) para todos los demás eventos y trabajo de desarrollo.
- **Regla de oro:** Durante el Sprint, no se realizan cambios que pongan en peligro el Objetivo del Sprint establecido.

### El Ciclo de Inspección

1. **Sprint Planning.** Inicia el Sprint. Se define Por qué es valioso este Sprint, Qué se puede hacer, y Cómo se realizará el trabajo elegido.
2. **Daily Scrum.** Evento diario de 15 min para los Developers. Inspeccionan el progreso hacia el Sprint Goal y adaptan el plan del día. No es un reporte de estatus al jefe
3. **Sprint Review.** Al final del Sprint. El equipo y los stakeholders inspeccionan el resultado del Sprint Incremento) y determinan futuras adaptaciones.
4. **Sprint Retrospective.** Cierra el Sprint. El equipo inspecciona cómo les fue en cuanto a personas, interacciones y procesos para planificar mejoras para el siguiente Sprint.
