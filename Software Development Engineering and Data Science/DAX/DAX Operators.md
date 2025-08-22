# Operadores en DAX

Los **operadores en DAX** son los símbolos que le dicen al motor DAX qué tipo de cálculo o comparación realizar. Son los bloques de construcción más básicos de cualquier fórmula y se dividen en tres categorías principales.

## Operadores de Comparación
(misma lógica de [[Introduction to Python#Operadores Relacionales|Operadores Relacionales en Python]])

Estos operadores se utilizan para comparar dos valores y devolver un resultado booleano (`TRUE` o `FALSE`). Son la base de las expresiones lógicas y de los filtros.

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

- **`BLANK`:** Todos los operadores de comparación, excepto `==`, tratan el valor `BLANK` como igual a `0`, cadena vacía (`""`), `DATE(1899, 12, 30)` o `FALSE`.
- **`==` (Igual a estricto):** Este operador compara dos valores y solo devuelve `TRUE` si son estrictamente iguales, incluso en el caso de `BLANK`.
- **Uso con cadenas:** Al comparar cadenas, es importante considerar si la comparación es sensible a mayúsculas y minúsculas, ya que algunas funciones de DAX lo son y otras no.

**Ejemplo de la diferencia:**

- `= BLANK() = 0` (Esto devuelve `TRUE`)
- `= BLANK() == 0` (Esto devuelve `FALSE`)

## Operadores Lógicos

Los **operadores lógicos** se utilizan para combinar o evaluar expresiones booleanas, resultando en un valor booleano (verdadero o falso). Los operadores lógicos principales son [[Logical Functions#AND|AND]], [[Logical Functions#OR|OR]] y [[Logical Functions#NOT|NOT]], así como el operador `IN` para verificar la pertenencia a una lista.

| Operador  | Nombre      | Equivalente en DAX Function | Ejemplo                                                |
| --------- | ----------- | --------------------------- | ------------------------------------------------------ |
| `&&`      | AND         | `AND()`                     | `Ventas[Región] = "Norte" && Ventas[Producto] = "A"`   |
| `\|\|`    | OR          | `OR()`                      | `Ventas[Región] = "Norte" \|\| Ventas[Región] = "Sur"` |
| `IN {}`   | IN          | N/A                         | `Producto[Color] IN {"Rojo", "Azul", "Gris"}`          |

- A menudo se usan en el primer argumento de una función [[Logical Functions#IF|IF]] o como filtros en la función [[Filter Functions#CALCULATE El motor de DAX|CALCULATE]].
- `IN {<lista>}`: Es una sintaxis más limpia y eficiente que anidar múltiples operadores OR. Se lee como "el valor está en la lista de valores".

## Operadores de Texto y Concatenación
(ver [[Text Functions]])

Los **operadores de texto** sirven para manipular y combinar cadenas de texto. Los principales operadores son `&` para concatenar texto, y funciones como `LEFT`, `RIGHT`, `MID`, `CONCATENATE`, `UPPER`, `LOWER`, `TRIM`, `FIND`, y `SEARCH` para extraer, reemplazar o transformar texto de diversas maneras.

| Operador de texto | Significado                | Ejemplo                  |
| ----------------- | -------------------------- | ------------------------ |
| `&`               | Concatena valores de texto | `[Ciudad]&”, “&[Estado]` |

El operador & es el más utilizado para unir textos. Es la alternativa preferida a la función [[Text Functions#CONCATENATE()|CONCATENATE]], ya que es más simple y flexible.
