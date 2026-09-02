---
tags: [pandas, python, data-science, io, export, cheat-sheet]
---

# 14 — Exportación de Datos

> Continúa de [[13 - MultiIndex y Datos Jerárquicos]]. El complemento simétrico de [[02 - Creación y Carga de Datos]].

## `to_csv()`

```python
df.to_csv("salida.csv", index=False)                      # index=False evita escribir el índice como columna extra
df.to_csv("salida.csv", sep=";", encoding="utf-8-sig")     # ; para Excel en configuración regional ES, BOM para acentos
df.to_csv("salida.csv.gz", compression="gzip")              # comprimido directamente
```

`index=False` es, en la práctica, casi siempre lo que se quiere al exportar para consumo externo — sin él, Pandas agrega una columna sin nombre con el índice numérico, que rara vez es útil para quien recibe el archivo.

## `to_excel()` — con múltiples hojas

```python
df.to_excel("reporte.xlsx", sheet_name="Ventas", index=False)

with pd.ExcelWriter("reporte_completo.xlsx", engine="openpyxl") as writer:
    df_ventas.to_excel(writer, sheet_name="Ventas", index=False)
    df_clientes.to_excel(writer, sheet_name="Clientes", index=False)
```

`ExcelWriter` como context manager es necesario para escribir **varias hojas en un mismo archivo** — llamar `to_excel()` directamente varias veces sobre la misma ruta sobrescribe el archivo en cada llamada.

## `to_parquet()` — formato recomendado para datos intermedios

```python
df.to_parquet("datos.parquet", engine="pyarrow", compression="snappy", index=False)
```

Preferir Parquet sobre CSV para guardar resultados intermedios de un pipeline: preserva dtypes exactos (sin re-inferencia al releer), comprime mejor, y es más rápido de leer/escribir. Ver también [[02 - Versionado de Datos - Comandos Fundamentales|DVC]] para versionar estos archivos junto con el código que los generó.

## `to_sql()`

```python
from sqlalchemy import create_engine
engine = create_engine("postgresql://user:pass@host:5432/db")

df.to_sql("ventas", engine, if_exists="replace", index=False)   # reemplaza la tabla completa
df.to_sql("ventas", engine, if_exists="append", index=False, chunksize=10_000)   # inserta en lotes
```

`if_exists="replace"` **elimina y recrea** la tabla — usar con cuidado en tablas de producción; `chunksize` evita construir un único `INSERT` gigantesco que puede agotar memoria o timeout en tablas grandes.

## `to_json()`

```python
df.to_json("datos.json", orient="records", date_format="iso", indent=2)
```

| `orient` | Forma del JSON resultante |
|---|---|
| `"records"` | Lista de objetos, uno por fila — el más común para APIs |
| `"columns"` | `{columna: {indice: valor}}` |
| `"split"` | `{"columns": [...], "index": [...], "data": [...]}` — reconstruye el DataFrame exacto |
| `"table"` | Incluye schema JSON explícito (dtypes) |

Ver también [[04 - Response Models y Serialización|FastAPI]] para servir un DataFrame como respuesta JSON de una API en producción (típicamente vía `df.to_dict(orient="records")` dentro de un modelo Pydantic, no `to_json` directo).

## `Styler` — formato condicional exportado a HTML/Excel

```python
(
    df.style
    .background_gradient(subset=["monto"], cmap="Greens")
    .format({"monto": "${:,.2f}"})
    .to_html("reporte_estilizado.html")
)

df.style.applymap(lambda v: "color: red" if v < 0 else "color: green", subset=["variacion"]).to_excel("reporte.xlsx")
```

`Styler` permite exportar tablas con formato visual (colores condicionales, formato de número) directamente a HTML o Excel — útil para reportes que se comparten fuera de un notebook. Ver también cómo mostrar DataFrames con formato en [[05 - Mostrar Datos - DataFrames, Tablas y Métricas|Streamlit]].

## Ver también

- [[02 - Creación y Carga de Datos]]
- [[15 - Rendimiento y Optimización]]
- [[02 - Versionado de Datos - Comandos Fundamentales|DVC]]
- [[04 - Response Models y Serialización|FastAPI]]
- [[05 - Mostrar Datos - DataFrames, Tablas y Métricas|Streamlit]]
