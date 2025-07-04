# Aggregate Functions

Las funciones de agregación (_o funciones de grupo_) operan sobre un conjunto de filas y devuelven un único valor de resumen. Son fundamentales para el análisis de datos, permitiéndote calcular promedios, sumas, recuentos, valores mínimos y máximos.

Estas funciones se usan comúnmente con la cláusula [[DQL (Data Query Language)#4. `GROUP BY` (_Agrupación de Filas_)|GROUP BY]], que las agrupa por una o más columnas, pero también pueden usarse sin GROUP BY para obtener un resumen de toda la tabla.

## `COUNT()` - Contar Filas

La función `COUNT()` se utiliza para contar el número de filas en un conjunto de resultados.

- `COUNT(*)`: Cuenta todas las filas, incluyendo aquellas con valores `NULL`.

  ```sql
  SELECT COUNT(*) FROM nombre_tabla;
  ```

- `COUNT(columna)`: Cuenta las filas donde la columna especificada no es `NULL`.

  ```sql
  SELECT COUNT(nombre_columna) FROM nombre_tabla;
  ```

- `COUNT(DISTINCT columna)`: Cuenta el número de valores únicos (_distintos_) no nulos en la columna especificada.

  ```sql
  SELECT COUNT(DISTINCT nombre_columna) FROM nombre_tabla;
  ```

## `SUM()` - Sumar Valores

La función `SUM()` calcula la suma total de los valores numéricos en una columna específica. Ignora los valores `NULL`.

```sql
SELECT SUM(columna) FROM nombre_tabla;
```

## `AVG()` - Calcular el Promedio

La función `AVG()` calcula el valor promedio (_media aritmética_) de los valores numéricos en una columna específica. Ignora los valores `NULL`.

```sql
SELECT AVG(nombre_columna) FROM nombre_tabla;
```

## `MIN()` - Encontrar el Valor Mínimo

La función `MIN()` encuentra el valor mínimo en una columna específica. Puede usarse con tipos de datos numéricos, de texto o de fecha/hora. Ignora los valores `NULL`.

```sql
SELECT MIN(nombre_columna) FROM nombre_tabla;
```

## `MAX()` - Encontrar el Valor Máximo

La función `MAX()` encuentra el valor máximo en una columna específica. Puede usarse con tipos de datos numéricos, de texto o de fecha/hora. Ignora los valores `NULL`.

```sql
SELECT MAX(nombre_columna) FROM nombre_tabla;
```

### IMPORTANTE

1. **`GROUP BY` y Funciones de Agregación:** Cuando usas una función de agregación con `GROUP BY`, la función se aplica a cada grupo individualmente. Las columnas que no están dentro de una función de agregación en la cláusula [[DML (Data Manipulation Language)#`SELECT`|SELECT]] deben estar en la cláusula `GROUP BY`.
2. **`NULL` Values:** Todas las funciones de agregación (`COUNT(columna)`, `SUM`, `AVG`, `MIN`, `MAX`) ignoran los valores `NULL` por defecto, excepto `COUNT(*)`, que cuenta todas las filas, independientemente de los valores `NULL`.
3. **`DISTINCT` Keyword:** Puedes usar `DISTINCT` dentro de `COUNT`, `SUM`, y `AVG` para que la agregación considere solo valores únicos.
