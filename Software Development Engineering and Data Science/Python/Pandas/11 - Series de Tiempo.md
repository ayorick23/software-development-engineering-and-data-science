---
tags: [pandas, python, data-science, time-series, cheat-sheet]
---

# 11 — Series de Tiempo

> Continúa de [[10 - Reshaping y Pivoting]]. Uno de los dominios donde Pandas tiene la funcionalidad más profunda de todo el ecosistema Python.

## Crear y convertir fechas

```python
pd.to_datetime("2026-03-15")
pd.to_datetime(["2026-03-15", "2026-04-01"])
pd.to_datetime(df["fecha_str"], format="%d/%m/%Y")   # formato explícito — más rápido y sin ambigüedad
pd.date_range(start="2026-01-01", end="2026-01-31", freq="D")   # rango de fechas diarias
pd.date_range(start="2026-01-01", periods=12, freq="ME")         # 12 fines de mes consecutivos
```

| Código `freq` | Significado |
|---|---|
| `D` | Día calendario |
| `B` | Día hábil |
| `W` | Semana |
| `ME` | Fin de mes (antes `M`) |
| `QE` | Fin de trimestre |
| `YE` | Fin de año |
| `h` | Hora |
| `min` | Minuto |

## `DatetimeIndex` — usar fechas como índice

```python
df = df.set_index("fecha")
df.index   # DatetimeIndex — habilita slicing y resampling especializados

df.loc["2026-03"]                  # todas las filas de marzo 2026 — slicing parcial por string
df.loc["2026-01-01":"2026-03-31"]  # rango de fechas
```

## Componentes de fecha con el accessor `.dt`

```python
df["fecha"].dt.year
df["fecha"].dt.month
df["fecha"].dt.day_name()          # "Monday", "Tuesday"...
df["fecha"].dt.dayofweek           # 0 = lunes
df["fecha"].dt.quarter
df["fecha"].dt.is_month_end
df["fecha"].dt.to_period("M")      # convierte a Period (representa el MES completo, no un instante)
```

## `resample()` — cambiar la frecuencia de la serie

```python
df.resample("ME")["ventas"].sum()          # downsampling: de diario a mensual, sumando
df.resample("D")["ventas"].mean()           # de datos irregulares a diario, promediando
df.resample("h").ffill()                     # upsampling: de diario a horario, propagando el último valor conocido
```

`resample()` requiere un `DatetimeIndex` y es conceptualmente un `groupby` especializado en intervalos de tiempo — **downsampling** (agregar, ej. diario→mensual) necesita una función de agregación; **upsampling** (ej. diario→horario) necesita una estrategia de llenado (`ffill`, `interpolate`).

## `rolling()` — ventanas móviles

```python
df["media_movil_7d"] = df["ventas"].rolling(window=7).mean()      # promedio de los últimos 7 registros
df["std_movil_30d"] = df["ventas"].rolling(window=30).std()
df["ventas"].rolling(window=7, min_periods=1).mean()                # no exige ventana completa al inicio de la serie
df["ventas"].rolling("7D").mean()                                    # ventana basada en TIEMPO real, no en número de filas
```

## `expanding()` — ventana acumulativa desde el inicio

```python
df["ventas_acumuladas_promedio"] = df["ventas"].expanding().mean()   # promedio de TODO lo visto hasta cada fila
```

A diferencia de `rolling` (ventana de tamaño fijo que se desliza), `expanding` siempre incluye desde el primer registro hasta la fila actual — útil para métricas tipo "promedio a la fecha".

## `shift()`, `diff()`, `pct_change()` — comparar contra periodos anteriores

```python
df["ventas_mes_anterior"] = df["ventas"].shift(1)                 # desplaza la serie 1 posición hacia adelante
df["variacion_absoluta"] = df["ventas"].diff()                     # equivalente a ventas - ventas.shift(1)
df["variacion_pct"] = df["ventas"].pct_change()                    # variación porcentual periodo a periodo
df["ventas_mismo_mes_ano_anterior"] = df["ventas"].shift(12)       # para datos mensuales, comparación interanual
```

## Zonas horarias

```python
df["fecha"] = df["fecha"].dt.tz_localize("America/Mexico_City")   # asigna timezone a datos naive
df["fecha_utc"] = df["fecha"].dt.tz_convert("UTC")                  # convierte entre timezones (ya localizado)
```

**Regla práctica:** `tz_localize` se usa una sola vez para declarar en qué timezone están los datos originales (naive → aware); `tz_convert` se usa para pasar de una zona horaria aware a otra.

## Ver también

- [[10 - Reshaping y Pivoting]]
- [[12 - Texto y Datos Categóricos]]
- [[07 - Datos Nulos y Duplicados]] — `ffill`/`interpolate` en el contexto de series de tiempo
