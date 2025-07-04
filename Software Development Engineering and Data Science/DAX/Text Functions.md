# Funciones de Texto

DAX incluye un conjunto de funciones de texto basadas en la biblioteca de funciones de cadena en Excel, pero que se han modificado para trabajar con tablas y columnas en modelos tabulares. En esta sección se describen las funciones de texto disponibles en el idioma DAX. Similares a las [[Advanced Functions#1. String Functions (_Funciones de Texto_)|funciones de texto de SQL]].

## Funciones Principales

- `EXACT(<text_1>, <text_2>)`: Comprueba si dos cadenas son idénticas (`XACT()` distingue entre mayúsculas y minúsculas).
- `FIND(<text_tofind>, <in_text>)`: Devuelve la posición inicial de un texto dentro de otro texto (`FIND()` distingue entre mayúsculas y minúsculas).
- `FORMAT(<value>, <format>)`: Convierte un valor en un texto en el formato de número especificado.
- `LEFT(<text>, <num_chars>)`: Devuelve el número de caracteres desde el inicio de una cadena.
- `RIGHT(<text>, <num_chars>)`: Devuelve el número de caracteres desde el final de una cadena.
- `LEN(<text>)`: Devuelve el número de caracteres en una cadena de texto.
- `LOWER(<text>)`: Convierte todas las letras de una cadena a minúsculas.
- `UPPER(<text>)`: Convierte todas las letras de una cadena a mayúsculas.
- `TRIM(<text>)`: Elimina todos los espacios de una cadena de texto.
- `CONCATENATE(<text_1>, <text_2>)`: Une dos textos para formar una sola.
- `SUBSTITUTE(<text>, <old_text>, <new_text>, <instance_num>)`: Reemplaza el texto existente con texto nuevo en una cadena.
- `REPLACE(<old_text>, <start_position>, <num_chars>, <new_text>)`: Reemplaza parte de una cadena con una nueva cadena.
