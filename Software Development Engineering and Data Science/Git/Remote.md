# Remote

## Repositorios Remotos

Los repositorios remotos son versiones de tu proyecto que se alojan en internet o en una red local. El comando `git remote` te permite gestionar estas conexiones, permitiéndote colaborar con otras personas o hacer una copia de seguridad de tu trabajo.

## Conexión y Gestión de Remotos

### `git remote`

```bash
git remote
```

Enumera todos los repositorios remotos que has configurado en tu proyecto. Si has clonado un repositorio, verás por defecto el alias `origin`, que es el nombre predeterminado para el servidor desde el que clonaste.

```bash
git remote -v
```

Muestra la misma lista que `git remote`, pero de forma más detallada, incluyendo las URL de cada repositorio remoto. La `-v` significa "verbose" (_verboso_).

```bash
git remote add <alias> <url>
```

Agrega una nueva conexión a un repositorio remoto. `<alias>` es el nombre corto que usarás para referirte a él (por ejemplo, `origin` o `upstream`), y `<url>` es la dirección del repositorio remoto.

## Manipulando Conexiones Remotas

```bash
git remote rm <nombre-repositorio-remoto>
```

Elimina una conexión remota de tu repositorio. Esto no elimina el repositorio remoto en sí, solo la referencia local a él (ver [[Tracking Path Changes#git rm|git rm]]).

```bash
git remote rename <nombre-antiguo> <nombre-nuevo>
```

Cambia el nombre de un repositorio remoto. Por ejemplo, puedes cambiar el nombre de ``origin`` a ``upstream`` si lo deseas.

```bash
git remote set-url <alias> <url-git>
```

Cambia la URL de un repositorio remoto existente. Esto es útil si la dirección del servidor ha cambiado o si necesitas usar un protocolo diferente (por ejemplo, de HTTPS a SSH).

## Sincronización con Remotos

Aunque no pertenecen directamente al comando ``git remote``, estos comandos son esenciales para trabajar con repositorios remotos.

```bash
git fetch <alias>
```

Descarga los datos más recientes del repositorio remoto, pero sin fusionarlos con tu rama local. Esto te permite ver qué cambios hay sin alterar tu trabajo actual (ver [[Share and Update#`git fetch`|git fetch]]).

```bash
git pull <alias> <rama>
```

Descarga los cambios de la rama remota especificada y los fusiona inmediatamente con tu rama local. Es una combinación de git fetch y git merge (ver [[Share and Update#`git pull`|git pull]]).

```bash
git push <alias> <rama>
```

Sube tus "[[Stage and Commit#Efectuar Cambios|commits]]" locales al repositorio remoto. Por ejemplo, [[Share and Update#`git push`|git push]] `origin main` enviaría los cambios de tu rama local `main` al servidor remoto `origin`.
