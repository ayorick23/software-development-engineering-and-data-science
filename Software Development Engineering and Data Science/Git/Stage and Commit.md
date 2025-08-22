# Stage & Commit

## Efectuar Cambios

En Git, el proceso de "_staging_" y "_committing_" es fundamental. El área de preparación (o **staging area**) es un espacio intermedio donde seleccionas y agrupas los cambios que deseas incluir en tu próxima "instantánea" o commit. Un **commit** es un registro permanente de los cambios en el historial de tu proyecto.

## Verificando el Estado

Antes de preparar tus cambios, es crucial saber en qué estado se encuentran tus archivos.

### `git status`

Muestra el estado del árbol de trabajo y del área de preparación. Te indica qué archivos han sido modificados, cuáles están listos para el "commit" y cuáles son nuevos y no están siendo rastreados.

```bash
git status
```

### `git diff`

Muestra las diferencias entre el árbol de trabajo y el área de preparación (staging area). Es útil para ver exactamente qué cambios has realizado en los archivos que aún no has agregado al "staging area".

```bash
git diff
```

## Preparando los Cambios

Una vez que has editado tus archivos, los preparas para el "commit" utilizando el comando `git add`.

### `git add`

```bash
git add <archivo>
```

Agrega el archivo especificado al área de preparación.

```bash
git add .
```

Agrega todos los archivos nuevos y modificados en el directorio actual y sus subdirectorios al área de preparación.

```bash
git add -A
```

Agrega todos los cambios, incluyendo archivos nuevos, modificados y eliminados, al área de preparación. Es similar a `git add .` pero también incluye los archivos eliminados.

### `git rm`
(ver [[Tracking Path Changes#`git rm`|git rm]])

Elimina un archivo del árbol de trabajo y del área de preparación. Este cambio debe ser luego confirmado con un "commit".

```bash
git rm <archivo>
```

## Creando la Instantánea

Una vez que todos los cambios que deseas están en el área de preparación, creas un "commit".

### `git commit`

```bash
git commit -m "<mensaje descriptivo>"
```

Registra los cambios preparados en el historial del repositorio con un mensaje que describe la naturaleza de los cambios.

```bash
git commit -am "<mensaje descriptivo>"
```

Una forma abreviada de agregar todos los archivos modificados y eliminados que ya estaban siendo rastreados por Git, y luego crear un "commit" con el mensaje proporcionado. No agrega archivos nuevos.

```bash
git commit --amend -m "<nuevo mensaje>"
```

Reescribe el último mensaje del "commit". Esto te permite corregir errores tipográficos o detalles olvidados en el mensaje. También te permite agregar archivos que olvidaste incluir en el "commit" anterior.

## Deshaciendo Cambios

A veces necesitas deshacer los cambios que has hecho. Git te da varias opciones para esto.

### `git restore`

```bash
git restore <archivo>
```

Descarta los cambios en el árbol de trabajo para el archivo especificado. Este comando es útil para revertir un archivo a su estado más reciente.

```bash
git restore --staged <archivo>
```

Deshace la preparación de un archivo. Mueve el archivo del área de preparación de vuelta al árbol de trabajo sin deshacer los cambios.

### `git reset`
(ver [[Rewrite History#`git reset`|git reset]])

```bash
git reset <archivo>
```

Un comando más antiguo que `git restore`. Deshace la preparación de un archivo. Es similar a `git restore --staged`, pero con una sintaxis diferente. Es más poderoso y a menudo utilizado para deshacer "commits".

## Comandos Avanzados

Estos comandos te permiten manipular el historial del repositorio de forma más compleja.

### `git log`
(ver [[Inspect and Compare#`git log`|git log]])

```bash
git log
```

Muestra el historial completo de "commits". Puedes ver quién hizo los cambios, cuándo se hicieron y el mensaje del "commit". Es esencial para entender el flujo del proyecto.

### `git rebase`
(ver [[Rewrite History#`git rebase`|git rebase]])

```bash
git rebase <rama>
```

Aplica los "commits" de la rama actual uno por uno sobre la rama especificada. Es una forma de mantener un historial de "commits" lineal y limpio, aunque es una operación avanzada que requiere cuidado.
