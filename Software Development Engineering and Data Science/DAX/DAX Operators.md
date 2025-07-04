# Operadores en DAX

## Operadores de Comparación

En DAX, los operadores de comparación se utilizan para comparar valores y devolver un resultado booleano (verdadero o falso). Estos operadores son fundamentales para construir [[Logical Functions|expresiones lógicas]] y [[Filter Functions|filtrar datos]] en tus modelos de datos.

| Operador de comparación | Significado           |
| ----------------------- | --------------------- |
| `=`                     | Igual a               |
| `==`                    | Estrictamente igual a |
| `>`                     | Mayor que             |
| `<`                     | Menor que             |
| `>=`                    | Mayor o igual que     |
| `<=`                    | Menor o igual que     |
| `<>`                    | No es igual           |

### Consideraciones Importantes

- **[[Information Functions|BLANK]]:** Todos los operadores de comparación, excepto `==`, tratan el valor `BLANK` como igual a `0`, cadena vacía (`""`), `DATE(1899, 12, 30)` o `FALSE`.
- **`==` (Igual a estricto):** Este operador compara dos valores y solo devuelve TRUE si son estrictamente iguales, incluso en el caso de `BLANK`.
- **Uso con cadenas:** Al comparar cadenas, es importante considerar si la comparación es sensible a mayúsculas y minúsculas, ya que algunas funciones de DAX lo son y otras no.

## Operadores de Texto

En DAX, los operadores de texto sirven para manipular y combinar cadenas de texto. Los principales operadores son `&` para concatenar texto, y funciones como [[Text Functions|LEFT, RIGHT, MID, CONCATENATE, UPPER, LOWER, TRIM, FIND y SEARCH]] para extraer, reemplazar o transformar texto de diversas maneras.

| Operador de texto | Significado                | Ejemplo                  |
| ----------------- | -------------------------- | ------------------------ |
| `&`               | Concatena valores de texto | `[Ciudad]&”, “&[Estado]` |

## Operadores Lógicos

En DAX, los operadores lógicos se utilizan para combinar o evaluar expresiones booleanas, resultando en un valor booleano (verdadero o falso). Los operadores lógicos principales son [[Logical Functions|AND, OR y NOT]], así como el operador `IN` para verificar la pertenencia a una lista.

| Operador lógico | Significado                   | Ejemplo                                            |
| --------------- | ----------------------------- | -------------------------------------------------- |
| `&&`            | Condición `AND`               | `([Ciudad] = “SaS”) && ([Ubicación] = “Centro”)`   |
| `\|\|`          | Condición `OR`                | `([Ciudad] = “SaS”) \|\| ([Ubicación] = “Centro”)` |
| `IN {}`         | Condición `OR` para cada fila | `Producto[Color] IN {”Azul”, “Rojo”, “Gris”}`      |
