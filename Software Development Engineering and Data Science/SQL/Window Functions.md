# Window Functions

Las **Funciones de Ventana** (_Window Functions_) son una de las características más avanzadas y potentes de SQL. Permiten realizar cálculos sobre un conjunto de filas relacionadas con la fila actual, conocido como "ventana", sin agrupar el resultado de la consulta. A diferencia de las [[Aggregate Functions|funciones de agregación estándar]] (`SUM`, `AVG`, `COUNT`), las funciones de ventana no colapsan las filas; es decir, el número de filas en el resultado de la consulta es el mismo que en la entrada, pero a cada fila se le añade un valor calculado basado en su ventana.

Las funciones de ventana son increíblemente útiles para tareas como calcular promedios móviles, clasificar resultados, calcular sumas acumuladas, o comparar valores entre filas relacionadas.

La sintaxis clave para una función de ventana es la cláusula `OVER()`.

```sql
FUNCION_DE_VENTANA(columna) OVER (
    [PARTITION BY columna1, columna2, ...]
    [ORDER BY columna3 [ASC|DESC], columna4 [ASC|DESC], ...]
    [ROWS/RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW / valor PRECEDING AND valor FOLLOWING]
)
```

1. **`PARTITION BY`:** Divide el conjunto de resultados en particiones (grupos) a las que se aplica la función de ventana de forma independiente. Es similar a [[DQL (Data Query Language)#4. `GROUP BY` (_Agrupación de Filas_)|GROUP BY]] pero no colapsa las filas. Si se omite, la función opera sobre todo el conjunto de resultados como una sola partición.

2. **`ORDER BY`:** Define el orden de las filas dentro de cada partición. Esto es crucial para funciones de ranking o cálculos acumulativos.

3. **`ROWS` / `RANGE` (Definición del Marco de Ventana):** Opcional. Especifica las filas exactas dentro de la partición sobre las que operará la función.

   - `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`: Incluye todas las filas desde el inicio de la partición hasta la fila actual. Útil para sumas acumuladas.
   - `ROWS BETWEEN X PRECEDING AND Y FOLLOWING`: Incluye X filas antes y Y filas después de la fila actual.
   - `ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING`: Incluye desde la fila actual hasta el final de la partición.
   - Si se omite `ROWS` o `RANGE`, el marco predeterminado varía según la función y si hay un [[DQL (Data Query Language)#6. `ORDER BY` (_Ordenación de Resultados_)|ORDER BY]]. Para funciones de ranking, es `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. Para agregaciones sin `ORDER BY` en el `OVER()`, es `RANGE BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` (toda la partición).

## Tipos Comunes de Funciones de Ventana

Las funciones de ventana se pueden clasificar en:

### 1. Funciones de Ranking (Clasificación)

Devuelven un valor de clasificación para cada fila dentro de la partición.

- `ROW_NUMBER()`: Asigna un número secuencial único a cada fila dentro de su partición, comenzando desde 1.
- `RANK()`: Asigna el mismo rango a las filas que tienen valores idénticos en la columna de ordenación. Si hay empates, el siguiente rango se salta.
- `DENSE_RANK()`: Similar a `RANK()`, pero no salta rangos en caso de empates. Los rangos son consecutivos.
- `NTILE(N)`: Divide las filas de cada partición en `N` grupos (o percentiles) y asigna un número de grupo a cada fila.

```sql
función_de_ranking() OVER (PARTITION BY columna_partición ORDER BY columna_orden [ASC|DESC])
```

### 2. Funciones de Agregación como Funciones Ventana

Las funciones de agregación estándar (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`) pueden usarse como funciones de ventana. Cuando se usan con `OVER()`, calculan la agregación para cada fila basada en su ventana, sin colapsar las filas.

```sql
función_de_agregación() OVER (PARTITION BY columna_partición ORDER BY columna_orden [ASC|DESC])
```

### 3. Funciones de Valor (Value Functions)

Estas funciones permiten acceder a filas específicas dentro de la ventana.

- `LAG(columna, offset, default)`: Devuelve el valor de `columna` de una fila anterior dentro de la partición. `offset` (opcional) indica cuántas filas atrás, `default` (opcional) es el valor si no hay fila en el `offset`.
- `LEAD(columna, offset, default)`: Devuelve el valor de `columna` de una fila posterior dentro de la partición. `offset` (opcional) indica cuántas filas adelante, `default` (opcional) es el valor si no hay fila en el `offset`.
- `FIRST_VALUE(columna)`: Devuelve el valor de `columna` de la primera fila en la ventana.
- `LAST_VALUE(columna)`: Devuelve el valor de `columna` de la última fila en la ventana.

```sql
función_de_valor() OVER (PARTITION BY columna_partición ORDER BY columna_orden [ASC|DESC])
```

## Cuándo Usar Funciones de Ventana

- **Ranking:** Encontrar los `N` elementos principales/inferiores en un grupo (ej., top 5 ventas por región).
- **Totales o Promedios Acumulativos/Móviles:** Calcular sumas o promedios que "crecen" o "se mueven" a lo largo de una serie de tiempo o un orden definido (ej., promedio de ventas de los últimos 7 días).
- **Comparación de Filas:** Comparar el valor de una fila con el de la fila anterior o siguiente (`LAG`, `LEAD`).
- **Cálculos de Porcentaje:** Calcular el porcentaje de una fila con respecto al total o al subtotal de su partición.
- **Eliminación de Duplicados:** En combinación con `ROW_NUMBER()`, es una técnica común para identificar y eliminar duplicados manteniendo solo la primera ocurrencia.
