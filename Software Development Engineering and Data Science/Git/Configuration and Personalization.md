# Configuration and Personalization

## Configuración y Personalización

Git es altamente personalizable. Esta sección abarca los comandos para configurar tu entorno, optimizar la experiencia de la línea de comandos y gestionar archivos ignorados, lo que te permitirá adaptar Git a tu flujo de trabajo y mantener tu repositorio limpio.

## Configuración de Git
(ver [[Setup and Init]])

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu.correo@ejemplo.com"
```

Estos comandos, que ya has visto, son fundamentales. Establecen tu identidad globalmente para todos los [[Stage and Commit#Efectuar Cambios|commits]] que realices en tu máquina.

```bash
git config --global core.editor "nombre-del-editor"
```

Define el editor de texto que Git usará por defecto para escribir mensajes de commit, rebase interactivos y otras tareas. Puedes usar editores como `vim`, `nano` o `code --wait` si usas Visual Studio Code.

```bash
git config --global color.ui auto
```

Habilita el color en la salida de la terminal de Git. Esto hace que la información sea mucho más fácil de leer, diferenciando el estado de los archivos, los mensajes de `commit` y los [[Inspect and Compare#`git diff`|diffs]].

## Creación de Alias (Atajos)

```bash
git config --global alias.<alias> "<comando>"
```

Los alias te permiten crear atajos para comandos de Git largos o de uso frecuente. Por ejemplo, en lugar de escribir [[Stage and Commit#`git status`|git status]], puedes escribir `git st`. Esto acelera significativamente tu flujo de trabajo.

## Manejo de Archivos Ignorados

### Archivo `.gitignore`

El archivo `.gitignore` es crucial para mantener un repositorio limpio. Le indica a Git qué archivos y directorios debe ignorar y no incluir en el control de versiones. Esto es vital para evitar subir archivos de configuración, dependencias o compilaciones que son específicos de tu entorno local. Debes crear este archivo manualmente en la raíz de tu repositorio y listar los patrones de nombres de archivos y carpetas que deseas ignorar.
