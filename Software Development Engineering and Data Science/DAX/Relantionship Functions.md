# Funciones de Relación

Las **funciones de relación** en DAX te permiten trabajar con modelos de datos interconectados. Su propósito es acceder a datos de tablas relacionadas y manipular el comportamiento de las relaciones. Si bien la lista de funciones es corta, su correcto uso es vital para construir [[DAX Statements#Medidas Cálculos Dinámicos|medidas]] y [[DAX Statements#Columnas Calculadas Nuevos Atributos en el Modelo|columnas calculadas]] que extraen datos de múltiples tablas.

## RELATED()

La función `RELATED` es una de las más usadas. Te permite seguir una relación de muchos a uno (muchos productos en una tabla de ventas a un solo producto en la tabla de productos) para acceder a un valor de la tabla en el lado "uno" de la relación.

```dax
RELATED(<columna>)
```

Ideal para crear columnas calculadas en la tabla de muchos (por ejemplo, la tabla de ventas) que traen información de la tabla de uno (la tabla de productos o clientes).

- **Regla clave:** `RELATED` solo funciona si hay una relación activa de muchos a uno desde la tabla donde se usa la fórmula hacia la tabla a la que se hace referencia.

## RELATEDTABLE()

A diferencia de `RELATED`, `RELATEDTABLE` es una función de iteración que te permite seguir una relación de uno a muchos para acceder a una tabla completa de datos relacionados. A menudo se usa con funciones de agregación como [[DAX/Aggregate Functions#SUMX()|SUMX]] o [[DAX/Aggregate Functions#COUNTX()|COUNTX]].

```dax
RELATEDTABLE(<table>)
```

Te permite realizar cálculos que resumen los datos en la tabla del lado "muchos" de la relación.

## CROSSFILTER()

Esta función te da un control explícito sobre la dirección del filtro cruzado en una relación. Es una de las funciones más avanzadas y se utiliza como un filtro dentro de [[Filter Functions#CALCULATE El motor de DAX|CALCULATE]].

```dax
CROSSFILTER(<columna_izquierda>, <columna_derecha>, <tipo_de_filtro_cruzado>)
```

### Parámetros Principales

- `<columna_izquierda>`: La columna del lado "muchos" de la relación.
- `<columna_derecha>`: La columna del lado "uno" de la relación.
- `<tipo_de_filtro_cruzado>`:
  - `Both`: Permite el filtro en ambas direcciones.
  - `OneWay`: Permite el filtro solo desde la tabla del lado "uno" a la del lado "muchos" (comportamiento predeterminado).
  - `None`: Desactiva el filtro.

Comúnmente se usa para activar temporalmente relaciones inactivas o para anular el comportamiento predeterminado del modelo.
