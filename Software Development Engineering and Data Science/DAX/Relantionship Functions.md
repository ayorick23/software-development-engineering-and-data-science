# Funciones de Relación

En DAX, las funciones de relación son cruciales para trabajar con datos interconectados en modelos tabulares. Estas funciones permiten acceder a datos de tablas relacionadas, especificar qué relación utilizar en un cálculo y controlar la dirección del filtrado cruzado.

## Funciones Principales

- `CROSSFILTER(<left_column>, <right_column>, <crossfiltertype>)`: Especifica la dirección de filtrado cruzado que se utilizará en un cálculo.
- `RELATED(<column>)`: Devuelve un valor relacionado de otra tabla.
