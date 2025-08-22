# Rewrite History

## Rehacer Commits

Los comandos de reescritura de historial son herramientas poderosas para limpiar y organizar tus [[Stage and Commit#Efectuar Cambios|commits]]. Te permiten deshacer errores, fusionar múltiples commits en uno solo y reorganizar el historial. Sin embargo, deben usarse con extrema precaución, especialmente en ramas que ya has compartido con otros. **Nunca reescribas el historial de una rama que otros han clonado o utilizado.**

## Reorganizando el Historial con git rebase

### `git rebase`

```bash
git rebase <rama>
```

Mueve o "reaplica" los commits de tu [[Branch and Merge#Cambios Grupales|rama]] actual sobre el último commit de la rama especificada. Esto crea un historial de commits más lineal y limpio, evitando los "commits" de fusión que crea [[Branch and Merge#`git merge`|git merge]]. Es muy útil para mantener la rama principal ordenada antes de fusionar tus cambios.

```bash
git rebase -i <commit>
```

Inicia un "rebase" interactivo. Te permite editar, combinar, reordenar o eliminar commits de forma precisa en el historial. Es una herramienta muy poderosa para limpiar tu historial antes de compartirlo.

## Deshaciendo Cambios

### `git reset`

El comando `git reset` te permite retroceder en el historial de commits. Es fundamental entender las diferentes opciones que tiene.

```bash
git reset <commit> (modo --mixed)
```

Deshace todos los commits después del commit especificado. Los cambios que estaban en esos commits se mueven a tu directorio de trabajo, pero no están en el área de preparación. Es el modo por defecto.

```bash
git reset --hard <commit>
```

**¡Cuidado!** Este comando deshace todos los commits posteriores al especificado y también elimina todos los cambios en tu directorio de trabajo y área de preparación. Es una acción destructiva que borra tu historial de forma permanente.

```bash
git reset --soft <commit>
```

Deshace todos los commits posteriores al especificado, pero mantiene los cambios en el área de preparación. Esto te permite volver a crear un commit con todos los cambios que estaban en los commits deshechos.

### `git revert`

```bash
git revert <commit>
```

A diferencia de `git reset`, `git revert` no reescribe el historial. En su lugar, crea un nuevo commit que revierte los cambios de un commit anterior. Esto es seguro de usar en ramas compartidas, ya que no altera el historial.
