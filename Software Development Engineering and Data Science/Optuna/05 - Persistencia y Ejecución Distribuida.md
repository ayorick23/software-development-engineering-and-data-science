---
tags: [optuna, hiperparametros, storage, distribuido, cheat-sheet]
---

# 05 — Persistencia y Ejecución Distribuida

> Continúa de [[04 - Pruners en Profundidad]].

Por defecto, un `study` de Optuna vive **solo en memoria** — si el proceso termina, se pierde. El parámetro `storage` conecta el estudio a un backend persistente, lo que además habilita paralelización real entre procesos y máquinas.

## Sin storage — solo en memoria (el caso por defecto)

```python
study = optuna.create_study(direction="minimize")
study.optimize(objective, n_trials=100)
# Si el script termina o crashea, todo el progreso desaparece
```

## `RDBStorage` — persistencia en base de datos relacional

```python
study = optuna.create_study(
    study_name="claro-rd-demand-tuning",
    storage="sqlite:///optuna_studies.db",   # SQLite local, el caso más simple
    direction="minimize",
    load_if_exists=True,   # si el estudio ya existe con ese nombre, lo reanuda en vez de fallar
)
study.optimize(objective, n_trials=100)
```

Con Postgres o MySQL (necesario para paralelización real entre múltiples procesos/máquinas):

```python
study = optuna.create_study(
    study_name="claro-rd-demand-tuning",
    storage="postgresql://usuario:password@host:5432/optuna_db",
    direction="minimize",
    load_if_exists=True,
)
```

```bash
pip install psycopg2-binary   # driver necesario para Postgres
```

## Reanudar un estudio existente

```python
# En una sesión posterior, sin recrear el estudio desde cero:
study = optuna.load_study(
    study_name="claro-rd-demand-tuning",
    storage="postgresql://usuario:password@host:5432/optuna_db",
)
study.optimize(objective, n_trials=50)   # continúa desde donde quedó, con historial completo
print(len(study.trials))   # incluye TODOS los trials de sesiones anteriores
```

## Paralelización dentro de una sola máquina — `n_jobs`

```python
study.optimize(objective, n_trials=200, n_jobs=4)   # 4 trials evaluándose en paralelo (multi-hilo)
```

> **Advertencia práctica**: `n_jobs` usa hilos (threads), no procesos — solo aporta paralelismo real si la función objetivo libera el GIL durante el trabajo pesado (por ejemplo, si internamente llama a librerías en C como XGBoost/LightGBM/NumPy, que sí lo liberan). Para modelos puramente en Python puro, `n_jobs` no acelera gran cosa; en ese caso conviene paralelizar a nivel de proceso (ver siguiente sección).

## Paralelización distribuida — múltiples procesos o máquinas

El patrón real para escalar: correr el **mismo script** en varios procesos/máquinas simultáneamente, todos apuntando al mismo `storage` compartido. Optuna coordina automáticamente para que no se dupliquen trials.

```python
# worker.py — este mismo script se ejecuta en N procesos/máquinas distintas
import optuna

def objective(trial):
    ...

study = optuna.load_study(
    study_name="claro-rd-demand-tuning",
    storage="postgresql://usuario:password@db-host:5432/optuna_db",
)
study.optimize(objective, n_trials=50)   # cada worker corre su propia porción de trials
```

```bash
# Lanzar 4 workers en paralelo (ej. en distintos contenedores/nodos):
python worker.py &
python worker.py &
python worker.py &
python worker.py &
wait
```

Cada worker consulta el storage compartido antes de proponer un nuevo trial, evitando colisiones — es la forma estándar de escalar una búsqueda de cientos de trials en un cluster o en varios contenedores de Kubernetes.

## `JournalStorage` — alternativa sin base de datos relacional

Para entornos donde levantar Postgres es una fricción innecesaria, Optuna soporta almacenamiento basado en archivos con locking (útil en sistemas de archivos compartidos tipo NFS, o en almacenamiento de objetos):

```python
from optuna.storages import JournalStorage, JournalFileBackend

storage = JournalStorage(JournalFileBackend("./optuna_journal.log"))
study = optuna.create_study(storage=storage, study_name="claro-rd-tuning", load_if_exists=True)
```

## Inspeccionar y administrar estudios persistidos

```python
# Listar todos los estudios guardados en un storage:
summaries = optuna.study.get_all_study_summaries(storage="postgresql://usuario:password@host:5432/optuna_db")
for s in summaries:
    print(s.study_name, s.n_trials, s.best_trial.value if s.best_trial else None)

# Eliminar un estudio:
optuna.delete_study(study_name="estudio-viejo", storage="postgresql://...")

# Copiar un estudio (ej. de SQLite local a Postgres compartido):
optuna.copy_study(
    from_study_name="tuning-local",
    from_storage="sqlite:///local.db",
    to_study_name="tuning-compartido",
    to_storage="postgresql://usuario:password@host:5432/optuna_db",
)
```

## Recuperación ante fallos — reanudar tras un crash

Como cada trial se persiste al storage inmediatamente después de completarse (no solo al final del estudio), un crash a mitad de una búsqueda de 500 trials no pierde el progreso — basta con volver a llamar `load_study` + `optimize` apuntando al mismo `storage`/`study_name`, y Optuna continúa desde el último trial persistido.

```python
def ejecutar_busqueda_robusta(n_trials_totales=500):
    study = optuna.create_study(
        study_name="tuning-robusto",
        storage="postgresql://...",
        direction="minimize",
        load_if_exists=True,
    )
    trials_restantes = n_trials_totales - len(study.trials)
    if trials_restantes > 0:
        study.optimize(objective, n_trials=trials_restantes)
    return study
```

## Ver también

- [[03 - Samplers en Profundidad]]
- [[04 - Pruners en Profundidad]]
- `MLflow/03 - Tracking - Servidor, Backend Store y Artifact Store.md`
- `Docker/Orchestration and Scalability.md`
