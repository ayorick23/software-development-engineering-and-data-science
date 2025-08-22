# Funciones de Inteligencia de Tiempo

Las **funciones de inteligencia de tiempo** en DAX te permiten manipular el contexto de filtro de fechas para realizar análisis de series temporales. Son esenciales para comparar periodos, calcular acumulados y entender tendencias a lo largo del tiempo. Para que funcionen correctamente, es crucial tener una [[Date and Time Functions#Creación de Tablas de Fecha|tabla de calendario]] dedicada y marcada como tal en tu modelo de datos.

## Desplazamiento de Periodos

Estas funciones son la clave para comparar un periodo con su homólogo anterior, como el año pasado o el mes anterior.

### SAMEPERIODLASTYEAR()

Devuelve una tabla de fechas que se desplaza un año hacia atrás en el tiempo desde el contexto de filtro actual.

```dax
SAMEPERIODLASTYEAR(<fechas>)
```

A menudo se usa como un filtro dentro de [[Filter Functions#CALCULATE El motor de DAX|CALCULATE]] para comparar una medida con el mismo período del año anterior.

### DATEADD()

Mueve una fecha un número de intervalos ([[Date and Time Functions#DAY()|DAY]], [[Date and Time Functions#MONTH()|MONTH]], [[Date and Time Functions#QUARTER()|QUARTER]], [[Date and Time Functions#YEAR()|YEAR]]) especificado hacia adelante o hacia atrás.

```dax
DATEADD(<fechas>, <número_de_intervalos>, <intervalo>)
```

Es más flexible que `SAMEPERIODLASTYEAR` porque te permite ir un número arbitrario de días, meses, trimestres o años hacia atrás o hacia adelante.

## Acumulados

Estas funciones calculan totales acumulados hasta la fecha, trimestre o mes actual. Son muy útiles para seguir el progreso hacia un objetivo.

### TOTALYTD()

Calcula el total acumulado del año hasta la fecha (_Year-to-Date_) para la expresión dada.

```dax
TOTALYTD(<expresión>, <fechas>[, <filtro>])
```

Para mostrar cómo las ventas o los ingresos se acumulan a lo largo del año fiscal.

- **Variantes:**

  - `TOTALQTD()`: Acumulado del trimestre hasta la fecha.
  - `TOTALMTD()`: Acumulado del mes hasta la fecha.

## Rangos de Fechas

Estas funciones te ayudan a definir el inicio y el fin de un periodo, lo que es útil para filtrar y agrupar datos.

### STARTOF() & ENDOF()

Devuelven la primera o última fecha del intervalo de tiempo especificado (mes, trimestre o año) en el contexto de filtro actual.

```dax
STARTOFMONTH(<fechas>)
ENDOFMONTH(<fechas>)
STARTOFQUARTER(<fechas>)
ENDOFQUARTER(<fechas>)
STARTOFYEAR(<fechas>)
ENDOFYEAR(<fechas>)
```

Comúnmente se utilizan con `CALCULATE` para obtener valores de inicio o fin de periodos.

### DATESBETWEEN()

Devuelve una tabla de fechas entre dos fechas específicas, incluyendo ambas.

```dax
DATESBETWEEN(<fechas>, <fecha_inicio>, <fecha_fin>)
```

Útil para crear un filtro para un rango de fechas estático o dinámico.
