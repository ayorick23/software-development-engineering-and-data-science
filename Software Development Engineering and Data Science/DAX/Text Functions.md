# Funciones de Texto
(misma lógica de [[Advanced Functions#1. String Functions (_Funciones de Texto_)|Funciones de Texto en SQL]])

Las **funciones de texto** en DAX te permiten trabajar con valores de texto para limpiar datos, extraer información o combinar cadenas. Son muy similares a sus contrapartes en Excel, lo que las hace intuitivas de usar para cualquier persona familiarizada con las hojas de cálculo.

## Transformación de Cadenas

Estas funciones cambian el formato o el contenido de una cadena de texto.

### LOWER() y UPPER()

- `LOWER()` convierte todas las letras de una cadena a minúsculas.
- `UPPER()` convierte todas las letras de una cadena a mayúsculas.

```dax
LOWER(<texto>)
UPPER(<texto>)
```

Ideal para estandarizar datos de texto, como nombres de productos o categorías, para asegurar que los filtros y las relaciones funcionen correctamente.

### TRIM()

Elimina los espacios en blanco iniciales, finales y duplicados en una cadena de texto.

```dax
TRIM(<texto>)
```

Esencial para la limpieza de datos.

### SUBSTITUTE()

Reemplaza todas las apariciones de un texto con otro. Es una de las funciones de manipulación más potentes.

```dax
SUBSTITUTE(<texto>, <texto_antiguo>, <texto_nuevo>[, <número_instancia>])
```

Limpiar códigos de producto, reemplazar abreviaciones, etc.

### REPLACE

Reemplaza una parte específica de una cadena, definida por su posición inicial y longitud.

```dax
REPLACE(<texto_antiguo>, <posición_inicio>, <número_caracteres>, <texto_nuevo>)
```

Modificar partes de códigos o números de serie.

## Extracción de Información

Estas funciones te permiten aislar y obtener partes específicas de una cadena.

### LEFT()

Devuelve el número de caracteres desde el inicio de una cadena.

```dax
LEFT(<texto>, <número_caracteres>)
```

### RIGHT()

Devuelve el número de caracteres desde el final de una cadena.

```dax
RIGHT(<texto>, <número_caracteres>)
```

### LEN()

Devuelve la longitud de una cadena de texto (el número de caracteres).

```dax
LEN(<texto>)
```

### FIND()

Devuelve la posición inicial de un texto dentro de otro. Es sensible a mayúsculas y minúsculas.

```dax
FIND(<texto_a_buscar>, <en_texto>[, <posición_inicial>])
```

A menudo se usa junto con `LEFT` o `RIGHT` para extraer texto dinámicamente.

## Lógica y Formato

Estas funciones te ayudan a comparar cadenas, unirlas y darles formato.

### EXACT()

Compara dos cadenas y devuelve `TRUE` si son idénticas (incluyendo mayúsculas y minúsculas).

```dax
EXACT(<texto1>, <texto2>)
```

Para validación de datos sensibles a mayúsculas y minúsculas.

### CONCATENATE()
(ver [[DAX Operators#Operadores de Texto y Concatenación|Operadores de Texto y Concatenación]])

Une dos cadenas de texto. El operador `&` realiza la misma función y es el método preferido por su simplicidad.

```dax
CONCATENATE(<texto1>, <texto2>)
```

Combinar nombres y apellidos, o códigos y descripciones.

### FORMAT()

Convierte un valor (número, fecha o moneda) en una cadena de texto con el formato especificado.

```dax
FORMAT(<valor>, <formato_código>)
```

Para mostrar valores en un formato legible en visualizaciones.
