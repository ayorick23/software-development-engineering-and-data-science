# Branch & Merge

## Cambios Grupales

Git es una herramienta poderosa para la colaboración, y las ramas son la clave de esto. Una **rama** es esencialmente un puntero movible a un [[Stage and Commit#Efectuar Cambios|commit]], permitiéndote trabajar en nuevas funcionalidades o arreglos de errores de forma aislada, sin afectar la rama principal de tu proyecto. El **merge** o fusión es la forma en que combinas el trabajo de diferentes ramas en una sola.

## Gestión de Ramas

Los siguientes comandos te permiten ver, crear y moverte entre las diferentes ramas de tu proyecto.

### `git branch`

```bash
git branch
```

Muestra una lista de todas las ramas locales en tu repositorio. La rama en la que te encuentras actualmente (la **rama activa**) se resalta con un asterisco (`*`).

```bash
git branch -av
```

Enumera todas las ramas locales y también las ramas remotas, mostrando el último commit de cada una. La opción `-a` muestra todas las ramas y `-v` proporciona detalles adicionales.

```bash
git branch <nombre-rama>
```

Crea una nueva rama local con el nombre especificado. La rama se crea como una copia exacta del commit actual, pero no te mueve a ella.

```bash
git branch -d <nombre-rama>
```

Elimina la rama especificada. La opción `-d` solo funciona si la rama ya ha sido completamente fusionada en otra rama.

```bash
git branch -D <nombre-rama>
```

Elimina la rama especificada sin importar si ha sido fusionada. Utiliza esta opción con precaución.

```bash
git branch -m <nuevo-nombre-rama>
```

Renombra la rama actual a un nuevo nombre.

## Moviéndose entre Ramas

### `git checkout`

```bash
git checkout <nombre-rama>
```

Te cambia a la rama especificada. Esto actualiza tu directorio de trabajo para que coincida con los archivos y el historial de esa rama.

```bash
git checkout -b <nombre-rama>
```

Un comando abreviado que combina dos acciones: primero crea una nueva rama con el nombre especificado y luego te cambia a ella (checkout). Es ideal para iniciar el trabajo en una nueva funcionalidad.

### `git switch`

```bash
git switch <nombre-rama>
```

Un comando más moderno y seguro que `git checkout` para cambiar de ramas. Es la opción recomendada para moverte entre ramas.

```bash
git switch -c <nombre-rama>
```

Es el equivalente moderno de `git checkout -b` para crear y cambiar a una nueva rama.

## Combinando y Eliminando Ramas

### `git merge`

```bash
git merge <rama>
```

Combina el historial de la rama especificada con la rama en la que te encuentras actualmente. Git intentará fusionar los cambios de forma automática, pero si hay conflictos, tendrás que resolverlos manualmente.

## Etiquetas

Las etiquetas son una forma de marcar commits específicos en el historial como importantes. Se usan comúnmente para marcar puntos de lanzamiento (versiones) del software.

### `git tag`

```bash
git tag <nombre-tag>
```

Agrega una etiqueta ligera al commit actual.

```bash
git tag -a <nombre-tag> -m "<mensaje>"
```

Crea una etiqueta anotada, que almacena más información (autor, fecha y mensaje) y es recomendable para las versiones oficiales.

```bash
git tag -d <nombre-tag>
```

Elimina una etiqueta.

```bash
git push <alias-remoto> <nombre-tag>
```

Sube una etiqueta al [[Remote#Repositorios Remotos|repositorio remoto]]. Por defecto, [[Share and Update#`git push`|git push]] no sube etiquetas, por lo que debes hacerlo explícitamente.
