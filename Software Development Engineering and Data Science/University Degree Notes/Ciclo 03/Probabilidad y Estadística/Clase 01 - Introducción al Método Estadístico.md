---
Fecha de creación: 2026-01-23T18:04:00
Materia:
  - Probabilidad y Estadística
Fecha de clase: 2026-01-23
---
[[Clase 02 - Funciones y Usos del Método Estadístico|Clase siguiente →]]

# Introducción al Método Estadístico

La estadística en la ingeniería no es solo "contar cosas"; es la herramienta fundamental para la **toma de decisiones bajo incertidumbre**. Nos permite transformar datos brutos en información accionable.

## Conceptos Fundamentales

Aunque suelen enseñarse juntas, tienen enfoques opuestos pero complementarios:

> [!IMPORTANT] **Probabilidad:** Parte de un **modelo teórico** para predecir qué resultados pueden ocurrir (deductivo). **Estadística:** Parte de los **datos observados** para construir o validar un modelo (inductivo).

### El Puente entre ambas

- **La Probabilidad** es el lenguaje que usamos para cuantificar la incertidumbre en la **Estadística Inferencial**.

## El Ciclo del Método Estadístico

Para que un estudio sea válido, debe seguir un proceso riguroso:

1. **Planteamiento:** Definir el "qué" y el "para qué".
2. **Recolección:** Diseño de experimentos o encuestas.
3. **Organización:** Limpieza de datos y tabulación.
4. **Análisis:** Aplicación de fórmulas y modelos.
5. **Interpretación:** ¿Qué significan estos números en el mundo real?
6. **Decisión:** Acción basada en la evidencia.

## Ramas de la Estadística

| **Característica** | **Estadística Descriptiva**                  | **Estadística Inferencial**                                   |
| ------------------ | -------------------------------------------- | ------------------------------------------------------------- |
| **Objetivo**       | Resumir y caracterizar un conjunto de datos. | Obtener conclusiones de una población basadas en una muestra. |
| **Herramientas**   | Media, desviación estándar, histogramas.     | Pruebas de hipótesis, intervalos de confianza, regresión.     |
| **Alcance**        | Solo los datos presentes.                    | Proyecta y predice hacia el futuro o hacia grupos mayores.    |

## Clasificación de Datos (Variables)

Para analizar datos, primero debemos saber qué "estamos midiendo".

### Cualitativos (Categorías)

- **Nominales:** No tienen un orden natural (ej. colores, profesión).
- **Ordinales:** Tienen una jerarquía (ej. nivel de estudios, satisfacción del cliente).

### Cuantitativos (Números)

- **Discretos:** Valores aislados, generalmente conteos (ej. número de hijos, piezas defectuosas). No existen "2.5 hijos".
- **Continuos:** Pueden tomar cualquier valor en un rango, generalmente mediciones (ej. temperatura, tiempo, peso). Aquí usamos precisión decimal.

## Población vs. Muestra

Es raro tener acceso a todos los datos de una población (sería un _Censo_), por eso usamos muestras.

- **Población ($N$):** El universo completo de estudio.
- **Muestra ($n$):** Un pedacito de ese universo. Para que sea válida, debe ser **aleatoria** y **representativa**.

> **Dato curioso:** Si la muestra no es representativa, caemos en un "sesgo", lo que hace que toda nuestra estadística sea, básicamente, una mentira muy bien adornada.

## Aplicaciones en Ingeniería

- **Control de Calidad:** ¿Cuántas piezas pueden fallar antes de que la producción sea un desastre?
- **Optimización de Procesos:** Reducir tiempos de espera en una línea de montaje.
- **Análisis de Riesgos:** Probabilidad de que una estructura falle bajo ciertas cargas.
