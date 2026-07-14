---
Fecha de creación: 2026-07-07T18:19:00
Materia:
  - Data Analysis
Fecha de clase: 2026-07-07
---
# Fundamentos del Análisis de Datos

- **El objetivo:** Descubrir conocimiento aplicable de manera útil al mundo real.
- **El proceso:** Los objetos (físicos o abstractos) se definen e identifican por sus características.
- **El evento:** Medir es 'congelar' el estado del mundo en una circunstancia de validación específica. Asumimos una relación persistente entre el dato y el fenómeno real.

## La Distorsión

- **Errores físicos:** Calibración incorrecta y límites inevitables de precisión matemática.
- **Errores ambientales:** Expresan la incertidumbre inherente del mundo y las interacciones complejas.
- **Sesgo vs. Ruido:** El sesgo es sistemático; el ruido mapea la variabilidad real de un mundo incierto.

## Escalas

1. **Nominal.** Etiquetas de identificación básica (sin orden inherente).
2. **Categórica.** Grupos clasificados, denota clase, pero sin magnitud.
3. **Ordinal.** Establece orden bajo transitividad, sin definir distancias.
4. **Intervalo.** Medición cuantitativa pura. Orden y distancia medible exacta.
5. **Proporción.** Máximo nivel informativo. Valores adimensionales y ratios absolutos.

## Anatomía de los Datos Estructurados

### Dataset (Tabla Maestra)

Formato columnar estructurado clásico.

### Celdas

Valores específicos medidos (pueden ser nulos, faltantes o ruidosos).

| Age | Spectacle prescription | Astigmatism | Tear production rate | Contact lenses |
| --- | ---------------------- | ----------- | -------------------- | -------------- |
| 24  | Myope                  | No          | Reduced              | Soft           |
| 35  | Hypermetrope           | Yes         | Normal               | Hard           |
| 18  | Myope                  | No          | Normal               | Soft           |
| 40  | Hypermetrope           | Yes         | Reduced              | None           |
## Anatomía de los Datos Semi-Estructurados

- **Formato autodescriptivo:** No depende de un esquema externo rígido: las etiquetas viajan con el dato.
- **Jerarquía y Anidamiento:** Permite relaciones complejas (objetos dentro de objetos o arreglos) sin necesidad de tablas unidas.
- **Claves y valores:** La unidad básica es el par ``"clave": valor``, lo que facilita el "Schema-on-Read".

## Anatomía de los Datos No Estructurados

- Contenido Crudo: Información sin modelo de datos predefinido (Texto libre, Pixeles, Onda sonora).
- Opacidad para Máquinas: 

## Ecosistema de Datos Semi-Estructurados

APIs y Web
Dispositivos IoT
Logs de Servidor

## De lo Crudo a lo Accionable

- NLP: Natural Language Processing para extraer entidades y sentimientos de textos.
- Computer Vision: Redes neuronales para etiquetas y clasificar contenido en imágenes y video.
- ETL/ELT: Transformación de JSON y XML hacia tablas analíticas para BI.

## OLTP y OLAP

### OLTP (Online Transaction Processing)

### OLAP (Online Analytical Processing)

### La Dinámica

## Minería de Datos

La **minería de datos** es una técnica asistida por computadora que se utiliza en los análisis para procesar y explorar grandes conjuntos de datos. Gracias a las herramientas y métodos de minería de datos, las organizaciones pueden descubrir patrones y relaciones ocultas en sus datos. La minería de datos transforma datos en bruto en conocimiento práctico. Las compañías utilizan dicho conocimiento para resolver problemas, analizar las consecuencias en el futuro de decisiones empresariales y aumentar sus márgenes de beneficio.

- Mito
	La "minería" no consiste en extraer los datos; los datos ya existen almacenados (en bases relacionales, data warehouses).

- Realidad
	Es el proceso sistemático de extraer conocimiento práctico y significativo a partir de esos grandes volúmenes de datos en bruto.

### Metodología CRISP-DM

1. **Comprensión del negocio:** Identificar objetivos empresariales.
2. **Comprensión de los datos:** Recopilación y evaluación.
3. **Preparación de los datos:** Limpieza y formateo crítico.
4. **Modelado:** Aplicación de algoritmos ML.
5. **Evaluación:** Medición frente al objetivo inicial.
6. **Implementación:** Despliegue para generar inteligencia.

## Preparación de Datos

- **Consumo de Tiempo:** Es, por mucho, la fase más larga y crítica del ciclo CRISP-DM. Los algoritmos requieren alta calidad.
- **Limpiar:** Gestionar datos faltantes, valores nulos predeterminados y errores físicos de medición.
- **Integrar:** Fusionar distintos conjuntos de datos dispares (OLAP, CSV, JSON) en un esquema unificado.
- **Formatear:** Configurar y transformar estrictamente los tipos y escalas de datos para la tecnología de minería.

## Técnicas de Extracción: Clasificación y Clústeres

Clasificación (Supervisado)

Agrupación en Clústeres (No Supervisado)
