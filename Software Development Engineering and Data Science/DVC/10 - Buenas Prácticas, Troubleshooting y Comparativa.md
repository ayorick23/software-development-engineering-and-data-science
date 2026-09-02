---
tags: [dvc, mlops, buenas-practicas, troubleshooting, comparativa, cheat-sheet]
---

# 10 — Buenas Prácticas, Troubleshooting y Comparativa

> Cierre del cheat-sheet. Se apoya en todos los archivos anteriores.

## `.dvcignore` — evitar versionar basura dentro de carpetas trackeadas

```
# .dvcignore
*.tmp
.ipynb_checkpoints/
__pycache__/
.DS_Store
```

Al versionar una carpeta completa (`dvc add data/raw/`), sin un `.dvcignore` bien configurado es fácil terminar versionando archivos temporales de herramientas (checkpoints de Jupyter, cachés de sistema operativo) que no forman parte real del dataset — inflando el cache sin necesidad.

## `dvc doctor` — primer paso ante cualquier comportamiento extraño

```bash
dvc doctor
```

Reporta versión de DVC, backends de remote disponibles, y configuración activa — la mayoría de errores de "no encuentra el remote" o "no puede autenticar" se diagnostican más rápido revisando esta salida que adivinando la causa.

## Cache corrupto o desincronizado

```bash
dvc cache dir   # confirma dónde vive el cache actual

dvc checkout --relink   # regenera los links del workspace desde el cache, sin re-descargar del remote
dvc fsck                  # verifica integridad del cache — reporta archivos corruptos/faltantes
```

Un síntoma típico de cache corrupto: `dvc status` reporta cambios en archivos que nadie modificó intencionalmente, o `dvc checkout` falla silenciosamente. `dvc fsck` es el diagnóstico correcto antes de asumir que hay que re-descargar todo desde el remote.

## Migrar el cache a otra ubicación (ej. disco más rápido, o compartido entre proyectos)

```bash
dvc cache dir /ruta/a/otro/disco
dvc config cache.type reflink,hardlink,copy   # orden de preferencia de estrategia de link
```

`reflink` (copy-on-write, disponible en sistemas de archivos como Btrfs/APFS/ZFS) es la opción más eficiente cuando está disponible; `hardlink` es el fallback estándar en la mayoría de sistemas; `copy` es el más lento pero siempre funciona, usado como último recurso.

## Archivo `.dvc` con conflicto de merge en Git

```bash
git status   # muestra el conflicto en el archivo .dvc, igual que cualquier conflicto de texto

# El .dvc es YAML plano — resolver el conflicto significa elegir qué hash conservar (o generar uno nuevo)
git checkout --theirs data/demanda_historica.parquet.dvc   # o --ours, según cuál versión de datos se quiere conservar
dvc checkout   # trae el contenido real correspondiente al hash elegido
```

A diferencia de un archivo binario versionado directamente en Git (donde un conflicto de merge es prácticamente irresoluble sin herramientas externas), un conflicto en un archivo `.dvc` es un conflicto de **texto YAML simple** — se resuelve como cualquier conflicto de Git, eligiendo qué hash de datos prevalece.

## Repositorio clonado sin acceso al remote — modo de solo lectura del código

```bash
git clone https://repo-url.git
cd repo
# Sin credenciales del remote configuradas, dvc pull fallará — pero el código y dvc.yaml siguen siendo legibles
dvc status   # reporta qué archivos faltan sin poder descargarlos
```

Útil recordar: clonar el repo Git siempre funciona (es solo código + punteros `.dvc`); lo que requiere credenciales es específicamente `dvc pull`/`push` contra el remote — un colaborador puede revisar la lógica del pipeline sin necesitar acceso a los datos reales.

## Comparativa final — DVC vs. Git-LFS vs. lakeFS vs. Delta Lake

| | DVC | Git-LFS | lakeFS | Delta Lake |
|---|---|---|---|---|
| **Qué versiona** | Archivos + pipelines completos | Solo archivos grandes | Buckets de object storage completos | Tablas dentro de un data lake |
| **Modelo mental** | "Git para datos y pipelines de ML" | "Git para archivos grandes" | "Git para un data lake completo" | "Control de versiones tipo base de datos sobre Parquet" |
| **Reproducibilidad de pipelines** | Sí, nativo (`dvc.yaml`) | No | No directamente | No directamente |
| **Requiere Git** | Sí | Sí | No — opera a nivel de object storage | No — opera sobre Spark/tablas |
| **Caso de uso típico** | Proyectos de ML con datasets versionados por snapshot y pipelines reproducibles | Cualquier repo con archivos grandes sueltos (no necesariamente ML) | Equipos de datos que necesitan versionar TODO un data lake (branches, commits sobre buckets S3) | Pipelines de datos con actualizaciones incrementales sobre tablas (time travel, ACID) |

### Cuándo DVC no es la herramienta correcta

- Si los datos de entrenamiento **siempre** se leen dinámicamente desde una base de datos/data warehouse (ej. "los últimos 90 días desde SQL Server"), sin snapshots explícitos y curados, el concepto de "versión fija de datos" de DVC aporta menos valor directo — ver la discusión de fondo en `Machine Learning/46-Reproducibilidad-con-DVC.md`.
- Si el equipo necesita versionar un **data lake completo** (múltiples tablas, actualizado continuamente por muchos pipelines), lakeFS o Delta Lake operan a una escala y con garantías (ACID, time travel a nivel de tabla) que DVC no está diseñado para ofrecer.
- Si solo se necesita resolver "archivos grandes en Git" sin ningún requisito de pipelines reproducibles ni tracking de experimentos, Git-LFS es una solución más simple y con menos superficie de aprendizaje.

## Checklist antes de considerar un proyecto "DVC-ready"

1. ¿`dvc init` corrió sobre un repo Git ya existente, con `.dvc/` versionado en Git?
2. ¿El remote está configurado y las credenciales usan `--local` (nunca hardcodeadas en `.dvc/config`)?
3. ¿El pipeline completo (no solo los datos) está declarado en `dvc.yaml`, con `deps`/`outs`/`params` correctos por etapa?
4. ¿`dvc.lock` se versiona en Git junto a `dvc.yaml`?
5. ¿Las métricas/plots usan `cache: false` para que Git las versione directamente y sean diffables?
6. ¿El CI/CD hace `dvc pull` antes de cualquier `dvc repro`, con credenciales inyectadas como secretos?
7. ¿Se usa `dvc exp` para exploración descartable, reservando commits de Git solo para configuraciones adoptadas?

## Ver también

- [[01 - Introducción y Arquitectura Interna]]
- [[04 - DVC Pipelines - dvc.yaml y Reproducibilidad]]
- [[08 - Integración con CI-CD y CML]]
- `Machine Learning/46-Reproducibilidad-con-DVC.md`
