# Introducción a Git

**Git** es un sistema de control de versiones distribuido, diseñado para manejar todo, desde pequeños a muy grandes proyectos con velocidad y eficiencia. Permite a múltiples desarrolladores trabajar juntos en un proyecto sin sobrescribir el trabajo de los demás, manteniendo un historial completo de cambios. Cada usuario tiene una copia local completa del repositorio, lo que significa que puedes trabajar sin conexión y sincronizar tus cambios más tarde.

## Configuración inicial (Setup)

Antes de empezar a usar Git, es fundamental configurar tu identidad. Esto es importante porque Git usa esta información para etiquetar los "[[Stage and Commit#Efectuar Cambios|commits]]" que haces. De esta manera, todo el mundo puede ver quién hizo qué.

## Comandos de Configuración e Inicio (Setup and Init)

### `git config`

El comando `git config` se utiliza para configurar variables que controlan la apariencia y el comportamiento de Git. Puedes configurar opciones a nivel de sistema (`--system`), a nivel de usuario (`--global`) o a nivel de repositorio (`--local`).

- **Configurar nombre de usuario:**

  ```bash
  git config --global user.name "Tu Nombre"
  ```

  Establece el nombre de usuario que se asociará con tus "commits" a nivel global (para todos tus repositorios).

- **Configurar correo electrónico:**

  ```bash
  git config --global user.email "tu.correo@ejemplo.com"
  ```

  Establece la dirección de correo electrónico que se asociará con tus "commits" a nivel global.

- **Verificar configuración:**

  ```bash
  git config --list
  ```

  Muestra todas las configuraciones de Git.

### `git init`

El comando `git init` es el primer paso para comenzar a usar Git en un proyecto. Crea un nuevo repositorio Git vacío.

- **Iniciar un nuevo repositorio:**

  ```bash
  git init
  ```

  Inicializa un nuevo repositorio Git en el directorio actual. Esto crea una subcarpeta oculta llamada ``.git`` que contiene todos los objetos y configuraciones de Git para tu repositorio.

### `git clone`

El comando `git clone` se usa para crear una copia local de un repositorio Git que ya existe, ya sea local o [[Remote#Repositorios Remotos|remoto]].

- **Clonar un repositorio remoto:**

  ```bash
  git clone <url-del-repositorio>
  ```

  Descarga un repositorio de una URL (por ejemplo, de GitHub o GitLab) a tu máquina local.

- **Clonar un repositorio y especificar el nombre del directorio local:**

  ```bash
  git clone <url-del-repositorio> <nombre-del-directorio>
  ```

  Clona un repositorio, pero lo guarda en un directorio con el nombre que especifiques.
