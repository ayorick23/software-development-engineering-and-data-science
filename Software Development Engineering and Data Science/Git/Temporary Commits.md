# Temporary Commits

## Commits Temporales

A veces, estás trabajando en una función y necesitas cambiar de rama para corregir algo urgente, pero no quieres hacer un [[Stage and Commit#Efectuar Cambios|commit]] de tu trabajo a medio terminar. El comando `git stash` es la solución perfecta. Te permite guardar temporalmente tus cambios sin tener que confirmarlos, para que puedas restaurarlos más tarde.

## Guardando y Restaurando Cambios

### `git stash`

```bash
git stash
```

Guarda de forma temporal todos los archivos modificados y con seguimiento (`tracked files`) de tu directorio de trabajo y del área de preparación (staging area). Es como "esconder" tus cambios para poder trabajar en otra cosa.

```bash
git stash push -m "<mensaje>"
```

Una forma más avanzada de `git stash` que te permite agregar un mensaje descriptivo a tu "stash", lo que facilita recordar para qué era cada conjunto de cambios.

```bash
git stash list
```

Muestra una lista de todos los conjuntos de cambios que has guardado temporalmente. Cada "stash" tiene un índice (`stash@{0}`, `stash@{1}`, etc.).

```bash
git stash apply
```

Restaura los cambios del "stash" más reciente en tu directorio de trabajo, pero mantiene el "stash" en la lista. Esto es útil si quieres aplicar los mismos cambios a varias [[Branch and Merge#Cambios Grupales|ramas]].

```bash
git stash pop
```

Restaura los cambios del "stash" más reciente y, a diferencia de `git stash apply`, elimina ese "stash" de la lista. Es la forma más común de aplicar y deshacerse del "stash" de una sola vez.

## Manipulando la Lista de Stashes

```bash
git stash drop
```

Elimina el "stash" más reciente de tu lista.

```bash
git stash drop <stash@{índice}>
```

Elimina un "stash" específico de la lista, utilizando su índice. Por ejemplo, `git stash drop stash@{1}`.

```bash
git stash clear
```

Elimina todos los "stashes" de la lista. Úsalo con precaución, ya que no se pueden recuperar.
