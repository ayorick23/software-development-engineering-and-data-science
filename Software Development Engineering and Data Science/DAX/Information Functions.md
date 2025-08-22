# Funciones de Información

Las **funciones de información** en DAX te proporcionan datos sobre el entorno de tu modelo, el contexto en el que se evalúan las fórmulas, o los tipos de datos en tus columnas. Son herramientas valiosas para el control de errores, la lógica condicional y la validación de datos.

## Inspección del Tipo de Dato y Errores

Estas funciones son fundamentales para la validación de datos y para crear lógica que maneje diferentes tipos de valores, incluyendo errores y espacios en blanco.

### ISBLANK() y ISERROR()

- `ISBLANK()` devuelve `TRUE` si el valor está vacío (`BLANK`).
- `ISERROR()` devuelve `TRUE` si el valor es un error.

```dax
ISBLANK(<valor>)
ISERROR(<valor>)
```

A menudo se usan con [[Logical Functions#IF|IF]] para reemplazar valores vacíos o errores con un valor más útil, como un cero o un texto.

### ISLOGICAL() y ISNUMBER()

- `ISLOGICAL()` devuelve `TRUE` si el valor es booleano (`TRUE` o `FALSE`).
- `ISNUMBER()` devuelve `TRUE` si el valor es un número.

```dax
ISLOGICAL(<valor>)
ISNUMBER(<valor>)
```

Para validar el tipo de dato de una columna antes de usarla en un cálculo.

## Inspección del Contexto de Filtro

Estas funciones te permiten entender cómo las relaciones y los filtros del usuario están afectando a tu cálculo.

### ISFILTERED()

Devuelve `TRUE` si la columna tiene un filtro directo aplicado (por ejemplo, desde un filtro de la página o una segmentación de datos).

```dax
ISFILTERED(<columna>)
```

Para crear medidas condicionales que se comportan de manera diferente si el usuario ha seleccionado un filtro.

### ISCROSSFILTERED()

Devuelve `TRUE` si la columna está siendo filtrada por otra tabla relacionada.

```dax
ISCROSSFILTERED(<columna>)
```

Útil para la depuración y para entender el flujo de contexto en un modelo complejo.

## Inspección del Modelo y del Usuario

Estas funciones proporcionan información sobre la estructura de tu modelo y el usuario que está interactuando con él.

### COLUMNSTATISTICS()

Esta función es menos común en las medidas, ya que se usa principalmente para la depuración y la optimización del modelo. Devuelve un conjunto de estadísticas sobre una columna, como el número de valores únicos.

```dax
COLUMNSTATISTICS(<columna>)
```

### NAMEOF()

Devuelve el nombre de una columna o [[DAX Statements#Medidas Cálculos Dinámicos|medida]] como una cadena de texto. A menudo se utiliza para la creación de expresiones dinámicas.

```dax
NAMEOF(<valor>)
```

### USERPRINCIPALNAME()

Devuelve la dirección de correo electrónico del usuario que ha iniciado sesión.

```dax
USERPRINCIPALNAME()
```

Esencial para la seguridad a nivel de fila (Row-Level Security o RLS), donde quieres mostrar a cada usuario solo los datos que le corresponden.
