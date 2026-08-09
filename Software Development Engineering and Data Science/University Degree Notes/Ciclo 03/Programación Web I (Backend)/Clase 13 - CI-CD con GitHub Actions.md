---
Fecha de creación: 2026-04-20T18:00:00
Materia:
  - Programación Web I (Backend)
Fecha de clase: 2026-04-20
---
[[Clase 12 - Despliegue de APIs con Docker|← Clase anterior]] | [[Clase 14 - Documentación de APIs con Swagger|Clase siguiente →]]

# CI/CD con GitHub Actions
(ver [[48-CICD-con-GitHub-Actions|CI/CD con GitHub Actions]] para la versión aplicada a proyectos de ML; automatiza la construcción de la imagen [[Clase 12 - Despliegue de APIs con Docker|Docker]] de la Clase 12 y la ejecución de las [[Clase 05 - Pruebas Manuales y Automatizadas|pruebas automatizadas]] de la Clase 05 en cada cambio de código)

## ¿Qué es CI/CD?

- **CI (Integración Continua):** cada vez que alguien sube cambios al repositorio, se ejecutan automáticamente pasos de verificación (compilar, correr pruebas) para detectar errores lo antes posible, en lugar de descubrirlos manualmente días después.
- **CD (Despliegue/Entrega Continua):** si esos pasos pasan, el sistema puede desplegar automáticamente la nueva versión, sin intervención manual.

Sin CI/CD, cada actualización de la API depende de que alguien recuerde ejecutar las pruebas y hacer el despliegue a mano — un proceso lento y propenso a errores humanos, justo lo que la [[Clase 05 - Pruebas Manuales y Automatizadas|Clase 05]] ya buscaba evitar a nivel de pruebas individuales.

## GitHub Actions

**GitHub Actions** es el sistema de CI/CD integrado directamente en GitHub. Se configura mediante archivos YAML dentro de la carpeta `.github/workflows/` del repositorio (ver [[Clase 08 - Git y el Control de Versiones|Git y el Control de Versiones]] de Programación II para el flujo de trabajo con Git que dispara estos workflows).

### Workflow Básico: Compilar y Probar

```yaml
# .github/workflows/ci.yml
name: CI - Build y Test

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - name: Descargar el código
        uses: actions/checkout@v4

      - name: Configurar .NET 8
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Restaurar dependencias
        run: dotnet restore

      - name: Compilar
        run: dotnet build --no-restore --configuration Release

      - name: Ejecutar pruebas
        run: dotnet test --no-build --configuration Release
```

Este workflow se dispara automáticamente con cada `push` o `pull request` a la rama `main`: si las pruebas de xUnit vistas en la [[Clase 05 - Pruebas Manuales y Automatizadas|Clase 05]] fallan, GitHub marca el commit o el pull request como fallido, evitando que código roto llegue a producción.

### Extender el Workflow: Construir y Publicar la Imagen Docker

```yaml
  build-docker:
    needs: build-and-test  # solo corre si el job anterior fue exitoso
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Iniciar sesión en Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Construir y subir la imagen
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: usuario/mi-api-dotnet:latest
```

`needs: build-and-test` asegura que la imagen Docker de la [[Clase 12 - Despliegue de APIs con Docker|Clase 12]] solo se construya y publique si el código compiló y pasó todas las pruebas — el pipeline completo queda encadenado: **compilar → probar → contenerizar → publicar**.

## Secrets: Credenciales Seguras en el Pipeline

Nótese `${{ secrets.DOCKERHUB_TOKEN }}` en el ejemplo anterior: nunca se escriben credenciales directamente en el archivo YAML (que es público si el repo lo es). GitHub permite guardar **secrets** cifrados a nivel de repositorio, accesibles solo dentro de los workflows — la misma lógica de "nunca hardcodear secretos en el código" vista para las claves JWT en la [[Clase 08 - Cifrado, Hashing, Salting y Arquitectura Limpia#Buenas prácticas de manejo seguro|Clase 08]].

## Beneficios en el Contexto de esta API

- **Calidad garantizada:** ningún cambio llega a `main` sin pasar las pruebas automatizadas.
- **Despliegues consistentes:** la imagen Docker que se prueba en CI es exactamente la misma que se publica y despliega — elimina el "en mi máquina sí funciona".
- **Retroalimentación rápida:** un desarrollador se entera de un error minutos después de subir su cambio, no días después en producción.
