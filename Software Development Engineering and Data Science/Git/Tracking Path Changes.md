# Tracking Path Changes

## Nombres de Archivo y Rutas

A medida que un proyecto evoluciona, es común mover, renombrar o eliminar archivos. Git ofrece comandos específicos para manejar estos cambios, asegurando que el historial de versiones se mantenga limpio y preciso. En lugar de simplemente eliminar y añadir un archivo, estos comandos le dicen a Git que un archivo se ha movido o eliminado intencionalmente, manteniendo un rastro de su historial.

## Moviendo y Eliminando Archivos

### `git rm`

```bash
git rm <archivo>
```

Elimina un archivo tanto del sistema de archivos local como del control de versiones de Git. El comando automáticamente agrega este cambio al área de preparación (staging area), listo para ser confirmado con un [[Stage and Commit#Efectuar Cambios|commit]].

```bash
git rm --cached <archivo>
```

Elimina el archivo del control de versiones de Git, pero lo mantiene en tu directorio de trabajo. Esto es útil si accidentalmente agregaste un archivo que no debería estar en el repositorio y quieres ignorarlo en el futuro sin borrarlo de tu máquina.

### `git mv`

```bash
git mv <archivo-original> <archivo-renombrado>
```

Renombra un archivo. Git lo interpreta como un solo evento de movimiento en lugar de un `rm` (eliminación) y un [[Stage and Commit#`git add`|add]] (adición). Esto preserva el historial del archivo a través de su cambio de nombre, facilitando su seguimiento en el historial del repositorio.

## Inspeccionando Cambios de Ruta
(ver [[Inspect and Compare#`git log`|git log]])

```bash
git log --stat
```

Muestra el historial de commits junto con un resumen de los cambios (`stat`). Para cada `commit`, verás una lista de los archivos que fueron modificados y un resumen de las líneas agregadas y eliminadas.

```bash
git log --follow <archivo>
```

Muestra el historial de un archivo específico y lo sigue a través de cualquier cambio de nombre que haya tenido. Esto te permite ver la historia completa del archivo, incluso si su ruta o nombre ha cambiado.

```bash
git log --stat -M
```

Muestra el historial de commits e incluye los archivos que fueron renombrados o movidos con un alto nivel de confianza. La opción `-M` le indica a Git que detecte los renombramientos, incluso si no se usó el comando `git mv`.
