# Inspect & Compare

## Repasar Historial

Git no solo te permite guardar cambios, sino también explorar y comparar el historial de tu proyecto. Los comandos de esta sección te ayudan a navegar a través de los "[[Stage and Commit#Efectuar Cambios|commits]]", ver las diferencias entre [[Branch and Merge#Cambios Grupales|ramas]] y entender la evolución de tus archivos.

## Inspeccionando el Historial

### `git log`

El comando `git log` es la herramienta principal para ver el historial de "commits" del proyecto.

```bash
git log
```

Muestra una lista cronológica de todos los "commits" en la rama actual. Cada entrada de "log" incluye el hash del "commit", el autor, la fecha y el mensaje del "commit".

```bash
git log --oneline
```

Muestra el historial en un formato conciso de una sola línea por "commit", que incluye el hash acortado y el mensaje del "commit".

```bash
git log --graph --oneline
```

Además del formato de una sola línea, muestra un gráfico ASCII que representa la estructura de las ramas y las fusiones del historial.

```bash
git log <primera-rama>..<segunda-rama>
```

Muestra los "commits" que están en la `<segunda-rama>` pero no en la `<primera-rama>`. Es una forma útil de ver qué tan adelantada o atrasada está una rama respecto a otra.

```bash
git log --follow <archivo>
```

Muestra el historial de "commits" para el archivo especificado, incluyendo un historial de renombramientos. Esto te permite ver la historia completa del archivo, incluso si su nombre ha cambiado.

## Comparando Cambios

Estos comandos te permiten ver las diferencias exactas de contenido entre varias versiones de tu proyecto.

### `git diff`

```bash
git diff <primera-rama> <segunda-rama>
```

Muestra las diferencias de contenido (qué líneas se añadieron y se eliminaron) entre las dos ramas especificadas. Te ayuda a entender los cambios entre versiones completas de tu proyecto.

```bash
git diff <commit> <archivo>
```

Muestra los cambios que se realizaron en un archivo específico en relación con el estado del proyecto en un "commit" determinado.

### `git show`

```bash
git show <commit>
```

Muestra los metadatos y los cambios de contenido de un "commit" específico. Es una forma de inspeccionar un "commit" en particular en detalle, incluyendo los archivos modificados, las adiciones y las eliminaciones de líneas.

### `git blame`

```bash
git blame <archivo>
```

Muestra el historial de un archivo línea por línea. Te indica el "commit" y el autor que realizó el último cambio en cada línea del archivo, lo cual es muy útil para depurar y entender por qué una línea de código se ve como lo hace.
