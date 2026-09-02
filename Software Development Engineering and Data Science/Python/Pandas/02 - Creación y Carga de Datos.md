---
tags: [pandas, python, data-science, io, cheat-sheet]
---

# 02 — Creación y Carga de Datos

> Continúa de [[Python/Pandas/01 - Introducción y Arquitectura Interna]].

## Creación manual

```python
# Desde diccionario de listas (clave = columna)
df = pd.DataFrame({"a": [1, 2, 3], "b": [4, 5, 6]})

# Desde lista de diccionarios (cada dict = fila)
df = pd.DataFrame([{"a": 1, "b": 4}, {"a": 2, "b": 5}])

# Desde un arreglo de NumPy, con nombres de columna explícitos
df = pd.DataFrame(np.random.randn(4, 3), columns=["x", "y", "z"])

# Serie desde diccionario (claves -> índice)
s = pd.Series({"lun": 10, "mar": 20, "mie": 15})
```

## `read_csv` — el punto de entrada más común

```python
df = pd.read_csv(
    "ventas.csv",
    sep=",",                       # o "\t", ";" según el archivo
    encoding="utf-8",
    usecols=["fecha", "producto", "monto"],   # lee solo estas columnas (menos memoria)
    dtype={"producto": "category"},           # fuerza dtype en la carga (evita inferencia + astype después)
    parse_dates=["fecha"],                    # convierte directo a datetime64
    na_values=["N/A", "-", ""],               # strings adicionales a tratar como nulo
    nrows=1000,                               # solo para explorar archivos gigantes
    index_col=0,                              # usa la primera columna como índice
)
```

**Inferencia de tipos:** por defecto, Pandas infiere el dtype de cada columna leyendo el archivo completo. En archivos grandes esto es costoso — declarar `dtype=` explícito en la llamada es más rápido que cargar todo como default y hacer `.astype()` después.

### Lectura en trozos (`chunksize`) para archivos que no caben en memoria

```python
resultados = []
for chunk in pd.read_csv("ventas_enorme.csv", chunksize=100_000):
    resumen = chunk.groupby("producto")["monto"].sum()
    resultados.append(resumen)

total = pd.concat(resultados).groupby(level=0).sum()
```

`chunksize` convierte `read_csv` en un iterador de DataFrames — el patrón estándar para procesar archivos más grandes que la RAM disponible sin recurrir a Spark/Dask (ver [[15 - Rendimiento y Optimización]] y [[Python/Pandas/16 - Integración con el Ecosistema]] para cuándo sí conviene dar el salto).

## Otros formatos de entrada

```python
# Excel — requiere openpyxl (xlsx) o xlrd (xls antiguo)
df = pd.read_excel("reporte.xlsx", sheet_name="Ventas 2026", skiprows=2)
hojas = pd.read_excel("reporte.xlsx", sheet_name=None)   # dict {nombre_hoja: DataFrame}

# SQL — requiere SQLAlchemy
from sqlalchemy import create_engine
engine = create_engine("postgresql://user:pass@host:5432/db")
df = pd.read_sql("SELECT * FROM ventas WHERE fecha >= '2026-01-01'", engine)
df = pd.read_sql_table("ventas", engine)   # tabla completa, sin escribir SQL

# Parquet — formato columnar comprimido, preserva dtypes exactos (recomendado sobre CSV)
df = pd.read_parquet("ventas.parquet", engine="pyarrow", columns=["fecha", "monto"])

# JSON
df = pd.read_json("ventas.json", orient="records")

# Portapapeles — útil para pegar rápido una tabla copiada de Excel/web en un notebook
df = pd.read_clipboard()
```

**Parquet vs CSV:** Parquet es columnar, comprimido, tipado (guarda el dtype exacto, no strings a re-inferir) y muchísimo más rápido de leer/escribir en datasets medianos-grandes. Es el formato preferido para persistir resultados intermedios de un pipeline — ver también [[02 - Versionado de Datos - Comandos Fundamentales|DVC]] para versionar estos archivos.

## Motor `pyarrow` para lectura acelerada

```python
df = pd.read_csv("ventas.csv", engine="pyarrow", dtype_backend="pyarrow")
```

Desde pandas 2.x, `engine="pyarrow"` en `read_csv` es notablemente más rápido que el motor `"c"` default en archivos grandes, y `dtype_backend="pyarrow"` hace que las columnas usen tipos Arrow (con soporte nativo de nulos y mejor uso de memoria) en vez de NumPy. Detalle completo en [[15 - Rendimiento y Optimización]].

## Ver también

- [[Python/Pandas/01 - Introducción y Arquitectura Interna]]
- [[03 - Exploración e Inspección de Datos]]
- [[14 - Exportación de Datos]]
- [[15 - Rendimiento y Optimización]]
- [[Python/Pandas/16 - Integración con el Ecosistema]]
- [[02 - Versionado de Datos - Comandos Fundamentales|DVC]]
