---
Fecha de creación: 2026-09-05T14:26:00
Materia:
  - Marcos de Trabajo Ágil
Fecha de clase: 2026-09-05
---
# Requisitos Ágiles, Historias de Usuario y Flujos en Jira

Los **requisitos ágiles en Jira** se gestionan estructurando las necesidades del cliente en épicas, historias de usuario e iniciativas para moverlas a través de flujos de trabajo visuales.

---

## Requisitos Ágiles

Los **requisitos ágiles** son necesidades de un proyecto que se capturan, gestionan y refinan de forma continua y flexible a lo largo de todo el proceso de desarrollo.

A diferencia de los métodos tradicionales (como cascada) que exigen una especificación detallada e inalterable al inicio, el enfoque ágil se adapta al cambio.

### Modelo Tradicional (_Cascada_)

- **Documentos Extensos:** Especificaciones rígidas de más de 100 páginas antes de programar.
- **Rápida Obsolescencia:** Los requerimientos cambian antes de que el código llegue a producción.
- **Foco en el Contrato:** Se prioriza el cumplimiento formal del documento sobre la necesidad real.

### Enfoque Ágil e Incremental

- **Unidades Atómicas de Valor:** Requisitos descompuestos en entregables funcionales pequeños.
- **Adaptabilidad Constante:** Evolución dinámica según el feedback y métricas de uso real.
- **Foco en el Usuario:** El software funcional responde a dolores específicos del cliente.

---

## Anatomía del Requisito Ágil

### Propiedades Fundamentales

- **Orientado al Usuario Final:** Describe el qué y el para qué, evitando prescribir soluciones técnicas cerradas.
- **Verticalidad Funcional:** Atraviesa todas las capas de la arquitectura Frontend, Backend, Base de Datos).
- **Estimable y Negociable:** Permite dimensionar la complejidad técnica sin generar bloqueos en el equipo.

> [!IMPORTANT] Principio Rector
> Un requisito ágil **no es un contrato cerrado**, sino un punto de partida para la colaboración continua entre el negocio y la ingeniería de software.

> _"El software funcionando es la medida principal de progreso." - Manifiesto Ágil_

---

## Modelo de las 3C

Las **3C de las historias de usuario**, presentadas por **Ron Jeffries**, ofrecen una forma sencilla pero eficaz de comprender y gestionar los requisitos en proyectos ágiles.

1. **Card (_Tarjeta_).** Soporte físico o digital que sintetiza la necesidad del usuario. Es breve, concisa y delimita la intención funcional del ítem.
2. **Conversation.** Promesa de diálogo continuo entre desarrolladores, QA, Product Owner y Stakeholders para resolver dudas de arquitectura y diseño.
3. **Confirmation.** Criterios de aceptación explícitos que verifican cuándo la historia está funcionalmente terminada y lista para validación.

---

## Estructura Estándar de la User Story

La estructura de una historia de usuario se compone de un título descriptivo, una descripción basada en un rol, objetivo y beneficio, y unos criterios de aceptación.

```text
COMO [rol / arquetipo de usuario] QUIERO [acción / capacidad en el sistema] PARA [beneficio / valor de negocio cuantificable]
```

- **¿Quién? (_Rol_).**
	Identifica al beneficiario directo (e.g., Cliente de Banca en Línea, Cajero, Repartidor). Evitar el genérico "Usuario".
- **¿Qué? (_Acción_).**
	Describe la interacción funcional que el usuario desea ejecutar sin detallar tecnología o bases de datos internas.
- **¿Por qué? (_Valor_).**
	Justifica el retorno de inversión o resolución de un problema concreto del negocio o usuario.

---

## Antipatrones vs. Buenas Prácticas

Un **antipatrón** de diseño de software es una práctica común que parece una solución útil, pero que en realidad genera problemas graves a largo plazo, mientras que una **buena práctica** es un enfoque estructurado y comprobado para escribir código flexible y limpio.

**Antipatrón Común (_Tarea Técnica_):**

> _"Cómo desarrollador quiero crear una base de datos postgreSQL para almacenar los usuarios del sistema."_

**Falla metodológica:** Describe una tarea interna de ingeniería. No aporta valor visible ni verificable para el cliente final.

**Redacción Correcta (_Valor Real_):**

> _"Cómo cliente, quiero una app completa para gestionar mis finanzas y pedir préstamos inmediatos."_

**Acierto ágil:** Define una necesidad real, genera beneficio inmediato y permite múltiples soluciones técnicas.

---

## Criterios de Aceptación

Los **criterios de aceptación** son las condiciones específicas y comprobables que deben cumplirse para que una historia de usuario o funcionalidad se considere terminada y cumpla con las expectativas del usuario.

### Propósito y Alcance

- **Límites Precisos:** Delimitan qué está dentro y fuera del alcance de la historia.
- **Condición de Éxito:** Proveen al equipo de QA y desarrollo el estándar de validación funcional.
- **Prevención de Deuda:** Evitan la ambigüedad en revisiones y despliegues.

### Enfoques de Formulación

- **Enfoque BDD *Behavior-Driven Development*):** Basado en escenarios mediante sintaxis formal Gherkin *Dado / Cuando / Entonces*).
- **Enfoque Basado en Reglas *Verification List*):** Restricciones técnicas, límites de rendimiento y reglas de validación.

---

## Formulación BDD (Gherkin)

La formulación en BDD consiste en documentar los ejemplos concretos acordados en la fase de descubrimiento usando un lenguaje estructurado y natural que humanos y computadoras puedan leer.

### Sintaxis Estructurada

- **Dado *Given*** El estado inicial o precondición del sistema.
- **Cuando *When*** La acción ejecutada por el usuario o evento.
- **Entonces *Then*** La respuesta observable de la UI, API o base de datos.

#### Ejemplo: Login Seguro.

**Escenario:** Validación de credenciales no registradas en inicio de sesión.

**Dado** que el usuario se encuentra en la pantalla de inicio de sesión de la plataforma,
**Cuando** introduce un correo electrónico no registrado y presiona el botón "Iniciar Sesión",
**Entonces** el sistema muestra el mensaje de error: _"Credenciales no válidas"_ y no expone si el correo existe o no en la base de datos por motivos de seguridad.

---

## Reglas Técnicas de Verificación

- Rendimiento & Límites Tiempos de respuesta máximos bajo concurrencia (e.g., "El balance general debe cargar en < 1.5s con 50 usuarios concurrentes").
- Formatos & Archivos Restricciones explícitas de carga (e.g., "Comprobantes en .PDF, .JPG y .PNG con tamaño máximo no superior a 5 MB").
- Políticas de Seguridad Validaciones de integridad (e.g., "Contraseñas con mínimo 12 caracteres, al menos una mayúscula, un número y un carácter especial").

---

## Criterios de Aceptación vs. DoD

Los **criterios de aceptación** son específicos para cada historia de usuario, mientras que la **Definición de Terminado (DoD)** es un estándar global que aplica a todas las tareas del equipo.

| Dimensión               | Criterios de Aceptación (AC)                                            | Definition of Done (DoD)                                                                          |
| ----------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **Nivel de Aplicación** | Específico e intrínseco a cada Historia de Usuario                      | Transversal y global para todas las historias del proyecto.                                       |
| **Variabilidad**        | Cambia según la funcionalidad particular requerida.                     | Estándar estricto acordado por el equipo de ingeniería.                                           |
| **Ejemplo Típico**      | _"El carrito debe aplicar un descuento del 10% si el cupón es válido."_ | _"Cobertura de pruebas unitarias ≥ 80%, SonarQube sin alertas críticas y despliegue en Staging."_ |

---

## Criterios de Calidad **INVEST** (Bill Wake)

Los **criterios de calidad INVEST** son un conjunto de seis pautas para evaluar y asegurar la alta calidad de las historias de usuario en el desarrollo ágil de proyectos.

### Las 6 Características de INVEST

1. **I - Independent.** Sin dependencias críticas que bloqueen su desarrollo simultáneo.
2. **N - Negotiable.** Detalles abiertos a discusión técnica entre equipo y PO.
3. **V - Valuable.** Aporta valor tangible y percibido por el cliente o negocio.
4. **E - Estimable.** Claridad suficiente para dimensionar esfuerzo y complejidad.
5. **S - Small.** Tamaño adecuado para completarse en un único ciclo o sprint.
6. **T - Testable.** Verificable mediante pruebas automatizadas o manuales.

---



