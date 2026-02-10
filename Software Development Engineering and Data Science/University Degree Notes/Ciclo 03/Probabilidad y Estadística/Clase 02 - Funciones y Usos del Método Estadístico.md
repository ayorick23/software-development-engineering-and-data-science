---
Fecha de creación: 2026-01-30T18:11:00
Materia:
  - Probabilidad y Estadística
Fecha de clase: 2026-01-30
---
# Funciones y Usos del Método Estadístico

La **estadística** es una disciplina fundamental en ingeniería porque permite **recopilar, organizar, analizar, interpretar y comunicar datos** para apoyar la toma de decisiones en contextos reales como producción, calidad, logística, finanzas, salud y tecnología.

Su propósito principal es transformar datos crudos en **información útil**, reduciendo la incertidumbre y permitiendo conclusiones objetivas basadas en evidencia.

## Niveles de Medición de Variables

Uno de los conceptos más importantes (_y subestimados_) en estadística es el **nivel de medición** de las variables.  

Este determina:

- Qué operaciones matemáticas son válidas
- Qué estadísticas se pueden calcular
- Qué gráficos son adecuados
- Qué conclusiones son correctas

Existen **cuatro niveles**, organizados jerárquicamente:

> **Nominal → Ordinal → Intervalo → Razón**

Cada nivel superior incluye las propiedades del anterior y agrega nuevas capacidades.

### Nivel Nominal

Clasifica datos en **categorías sin orden**.

#### Características:

- No existe jerarquía
- Solo se puede contar frecuencias
- No se pueden hacer operaciones matemáticas

#### Ejemplos:

- Tipo de material: acero, aluminio, plástico
- Carrera universitaria
- Estado civil
- Tipo de falla en un sistema

#### Estadísticos válidos:

- Frecuencia
- Moda

#### Gráficos comunes:

- Gráfico de barras
- Gráfico circular

### Nivel Ordinal

Clasifica datos en **categorías con orden**, pero sin distancias iguales entre ellas.

#### Características:

- Existe jerarquía
- No se puede calcular diferencia ni promedio

#### Ejemplos:

- Nivel de satisfacción: bajo, medio, alto
- Puestos en una competencia: 1°, 2°, 3°
- Escala de riesgo: bajo, moderado, alto

#### Estadísticos válidos:

- Mediana
- Moda

#### Gráficos comunes:

- Barras ordenadas

### Nivel de Intervalo

Datos numéricos donde:

1. Existe orden  
2. Las diferencias son significativas  
3. El cero **no** es absoluto

#### Características:

- Se puede sumar y restar
- No se puede hacer proporciones

#### Ejemplos:

- Temperatura en °C o °F
- Fechas en el calendario
- Puntajes de pruebas estandarizadas

#### Estadísticos válidos:

- Media
- Varianza
- Desviación estándar

#### Ejemplo:

> 30 °C no significa el doble de calor que 15 °C, porque el cero no representa ausencia total de temperatura.

### Nivel de Razón

Datos numéricos con:  
1. Orden  
2. Diferencias iguales  
3. **Cero absoluto**

Permite **todas las operaciones matemáticas**, incluidas proporciones.

#### Ejemplos:

- Peso (kg)
- Altura (m)
- Tiempo (s)
- Número de defectos
- Ingresos

#### Estadísticos válidos:

- Todos (media, mediana, desviación, razones, etc.)

#### Ejemplo:

> Una máquina que produce 20 piezas/hora produce el doble que otra que produce 10 piezas/hora.

| Nivel     | Orden | Diferencias | Cero absoluto | Ejemplo               |
| --------- | ----- | ----------- | ------------- | --------------------- |
| Nominal   | ❌     | ❌           | ❌             | Tipo de material      |
| Ordinal   | ✔     | ❌           | ❌             | Nivel de satisfacción |
| Intervalo | ✔     | ✔           | ❌             | Temperatura °C        |
| Razón     | ✔     | ✔           | ✔             | Peso, tiempo          |

## Funciones del Método Estadístico en Ingeniería

### Función Descriptiva

Permite **resumir grandes volúmenes de datos** usando números y gráficos comprensibles.

Incluye:

- Media, mediana, moda
- Tablas de frecuencia
- Histogramas

#### Ejemplo:

> Calcular el promedio de defectos diarios en una línea de producción durante un mes.

### Función de Organización y Presentación

Consiste en **ordenar y estructurar los datos** para facilitar su interpretación.

Incluye:

- Tablas de distribución de frecuencia
- Diagramas de barras
- Histogramas

#### Ejemplo:

> Agrupar los tiempos de entrega de pedidos por rangos de horas.

### Función Exploratoria

Busca **detectar patrones, anomalías o tendencias** sin asumir hipótesis previas.

Incluye:

- Diagramas de caja
- Gráficos de dispersión
- Análisis preliminar

#### Ejemplo:

> Identificar si ciertos turnos de producción generan más errores que otros.

### Función Inferencial

Permite **sacar conclusiones sobre una población** usando una muestra.

Incluye:

- Intervalos de confianza
- Pruebas de hipótesis
- Regresión

#### Ejemplo:

> Estimar el tiempo promedio de atención al cliente en todas las sucursales usando una muestra de 100 registros.

### Función Predictiva

Busca **anticipar resultados futuros** a partir de datos históricos.

Incluye:

- Regresión lineal
- Series de tiempo
- Modelos predictivos

#### Ejemplo:

> Predecir la demanda mensual de productos para planificar inventarios.

### Función de Toma de Decisiones

Integra los resultados estadísticos para **elegir la mejor alternativa posible bajo incertidumbre**.

#### Ejemplo:

> Decidir entre dos proveedores comparando tiempos de entrega, costos y tasas de defectos.

## Distribución de Frecuencias e Intervalos

Cuando se trabaja con datos cuantitativos continuos (como peso, tiempo, edad, temperatura), es común organizarlos en **intervalos de clase**.

### Conceptos Clave

- **Rango**

$$
\text{Rango} = \text{Valor máximo - Valor mínimo}
$$

- **Número de intervalos (regla de Sturges)**

$$
k = 1 + 3.322\log_{10}(n)
$$

- **Amplitud de clase**

$$
A = \frac{\text{Rango}}{k}
$$

### Tipos de Frecuencia

| Tipo                                   | Significado                     |
| -------------------------------------- | ------------------------------- |
| **Frecuencia absoluta (fi)**           | Número de datos en un intervalo |
| **Frecuencia relativa (hi)**           | $fi / n$                        |
| **Frecuencia porcentual (pi)**         | $hi × 100$                      |
| **Frecuencia acumulada (Fi)**          | Suma progresiva de $fi$         |
| **Frecuencia relativa acumulada (Hi)** | $Fi / n$                        |
