# Declaraciones DAX

En DAX, una **declaración** es la forma en que defines y nombras una expresión para que pueda ser utilizada en tu modelo. Las tres declaraciones principales son las **Medidas**, las **Columnas Calculadas** y las **Variables**. Entender la diferencia entre ellas es crucial para el modelado de datos eficiente y para crear fórmulas claras y de alto rendimiento.

## Medidas: Cálculos Dinámicos

Una **medida** es una declaración que define un cálculo de agregación, como una suma o un promedio. A diferencia de las columnas, las medidas no almacenan el resultado en tu modelo de datos; se calculan dinámicamente en el momento en que se visualizan en un informe, ajustándose al contexto de filtro.

### Cuándo usar medidas

- Para cálculos que dependen del contexto de un filtro (por ejemplo, el total de ventas para una región o un año específico).
- Para agregar datos.
- Son más eficientes en términos de memoria, ya que no guardan valores para cada fila.

### Sintaxis

```dax
Total Ventas = SUM(Ventas[Importe])
```

**Importante:** Se pueden usar en visualizaciones como gráficos de barras, tarjetas o tablas dinámicas.

## Columnas Calculadas: Nuevos Atributos en el Modelo

Una **columna calculada** es una declaración que agrega una nueva columna a una tabla existente. La expresión DAX se evalúa fila por fila y el resultado se almacena en el modelo de datos. Este valor se recalcula solo cuando se actualiza el modelo.

### Cuándo usar columnas calculadas

- Para crear nuevos atributos que no existen en los datos originales, como una categoría de edad o un margen de beneficio por transacción.
- Cuando necesitas una columna para una segmentación de datos, un eje en un gráfico o para una relación con otra tabla.

### Sintaxis

```dax
Margen de Beneficio = Ventas[Ingreso] - Ventas[Costo]
```

**Consideraciones:** Consumen memoria. Si puedes lograr el mismo resultado con una medida, es preferible usar la medida para optimizar el rendimiento.

## Variables: Optimizando la Lectura y el Rendimiento

Las **variables (`VAR`)** son declaraciones que almacenan temporalmente el resultado de una expresión. Son increíblemente útiles dentro de medidas o columnas calculadas para mejorar la legibilidad y evitar repetir cálculos.

### Cuándo usar variables

- Para descomponer una fórmula compleja en pasos más pequeños y comprensibles.
- Para capturar un resultado que necesitas reutilizar varias veces, lo que evita que DAX lo recalcule, mejorando así el rendimiento.

### Sintaxis

```dax
Total Ventas vs Año Pasado =
VAR VentasEsteAño = SUM(Ventas[Importe])
VAR VentasAñoPasado = CALCULATE(SUM(Ventas[Importe]), SAMEPERIODLASTYEAR('Calendario'[Date]))
RETURN
   VentasEsteAño - VentasAñoPasado
```

**Importante:** Una declaración `VAR` debe ser seguida por un `RETURN` (misma lógica de [[Functions#Valores de Retorno|Valores de Retorno en Python]]) que especifica qué valor debe devolver la medida o la columna.
