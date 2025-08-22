# Share & Update

## Compartir y Actualizar

La colaboración en Git se basa en la capacidad de compartir tu trabajo y obtener las actualizaciones de otros. Los comandos de esta sección te permiten sincronizar tu repositorio local con uno o varios [[Remote#Repositorios Remotos|repositorios remotos]], asegurando que todos los colaboradores trabajen con la misma versión del proyecto.

## Conexión con Repositorios Remotos

```bash
git remote add <alias> <url>
```

Conecta un repositorio local a un repositorio remoto. El `<alias>` (por ejemplo, `origin`) es el nombre corto que se usará para referirse a la `<url>` del repositorio remoto. Esto es crucial para poder subir y descargar cambios.

## Descargar y Fusionar Cambios

### `git fetch`

```bash
git fetch <alias>
```

Descarga los datos más recientes de todas las [[Branch and Merge#Cambios Grupales|ramas]] del repositorio remoto (especificado por el `<alias>`). Los cambios se guardan localmente, pero no se fusionan con tu rama actual. Esto te permite revisar las actualizaciones antes de aplicarlas.

### `git merge`

```bash
git merge <alias>/<rama>
```

Fusiona la rama remota especificada con tu rama local actual. Este comando a menudo se usa después de un `git fetch` para aplicar los cambios descargados.

### `git pull`

```bash
git pull <alias> <rama>
```

Es un atajo que combina dos comandos: `git fetch` y `git merge`. Descarga los cambios de la rama remota y los fusiona inmediatamente con tu rama local. Es una forma rápida de actualizar tu proyecto. Si el repositorio remoto es el único configurado, puedes usar solo `git pull`.

## Subir y Compartir Cambios

### `git push`

```bash
git push <alias> <rama>
```

Sube todos los [[Stage and Commit#Efectuar Cambios|commits]] de tu rama local a la rama del repositorio remoto especificado. Es la forma de compartir tu trabajo con el resto del equipo.

```bash
git push --set-upstream <alias> <rama>
```

Sube la rama local por primera vez y la "enlaza" con la rama remota. Esto te permite usar `git pull` y `git push` sin especificar el `<alias>` y la `<rama>` en el futuro. Es común usar el alias `-u` en lugar de `--set-upstream`.

```bash
git push <alias> --delete <rama>
```

Elimina la rama especificada del repositorio remoto.

## Transferencia de Commits entre Ramas

### `git cherry-pick`

```bash
git cherry-pick <id_commit>
```

Aplica un commit específico de una rama a tu rama actual. Esto es útil para transferir un solo cambio sin tener que fusionar toda una rama. Por ejemplo, si un arreglo de un error urgente está en una rama de desarrollo, puedes usar `cherry-pick` para llevar ese cambio directamente a la rama principal.
