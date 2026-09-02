---
tags: [pandas, python, data-science, indexing, cheat-sheet]
---

# 04 — Selección e Indexación

> Continúa de [[03 - Exploración e Inspección de Datos]].

## Selección de columnas

```python
df["precio"]                 # una columna -> Series
df[["precio", "stock"]]      # varias columnas -> DataFrame (nótese el doble corchete)
df.precio                     # acceso por atributo — evitar si el nombre choca con un método (ej. "count")
```

## `loc` — selección por ETIQUETA

```python
df.loc["fila_003"]                         # una fila por etiqueta de índice
df.loc[["fila_002", "fila_008"]]           # varias filas específicas
df.loc["fila_002":"fila_007"]              # rango — INCLUSIVO en ambos extremos (a diferencia de Python normal)
df.loc["fila_002":"fila_005", "nombre":"ciudad"]   # filas y columnas por etiqueta simultáneamente
df.loc[df["precio"] > 100, "producto"]     # loc + condición booleana — el patrón más común en la práctica
df.loc[df["precio"] > 100, "en_oferta"] = True   # asignación condicional — forma CORRECTA (ver 17)
```

## `iloc` — selección por POSICIÓN entera

```python
df.iloc[0]                     # primera fila (por posición, no por etiqueta)
df.iloc[[1, 3, 5]]              # filas específicas por posición
df.iloc[1:4]                    # rango — EXCLUSIVO en el extremo final (como slicing normal de Python)
df.iloc[0:5, 1:-1]              # primeras 5 filas, desde la 2da hasta la penúltima columna
df.iloc[-1]                      # última fila
```

| | `loc` | `iloc` |
|---|---|---|
| Selecciona por | Etiqueta (label) | Posición entera (int) |
| Rango `a:b` | Inclusivo en `b` | Exclusivo en `b` |
| Acepta booleanos | Sí | Sí (como máscara posicional) |
| Requiere índice ordenado para slices | No estrictamente, pero recomendado | N/A (siempre por posición) |

## `at` / `iat` — acceso a un único valor escalar

```python
df.at["fila_001", "nombre"]     # equivalente a .loc pero optimizado para UN valor -> más rápido
df.iat[0, -1]                    # equivalente a .iloc pero para UN valor
```

Usar `at`/`iat` en vez de `loc`/`iloc` cuando se necesita un solo escalar (por ejemplo dentro de un loop, aunque los loops fila-por-fila en sí deben evitarse — ver [[15 - Rendimiento y Optimización]]).

## Indexación booleana

```python
df[df["precio"] > 100]                       # filtro simple
df[(df["precio"] > 100) & (df["stock"] < 50)]  # AND — paréntesis obligatorios por precedencia de operadores
df[~(df["region"] == "Norte")]                # NOT
```

Ver el catálogo completo de operadores de filtrado (`isin`, `between`, `query`) en [[05 - Filtros y Condiciones Avanzadas]].

## `xs()` — cross-section, útil con `MultiIndex`

```python
df.xs("2026-01", level="mes")   # selecciona un nivel específico de un MultiIndex sin desarmar la jerarquía
```

Cobertura completa de índices jerárquicos en [[13 - MultiIndex y Datos Jerárquicos]].

## El error más común: indexación encadenada (chained indexing)

```python
# INCORRECTO — dos operaciones de indexación encadenadas, resultado ambiguo (vista o copia según el caso)
df[df["precio"] > 100]["en_oferta"] = True   # SettingWithCopyWarning, puede NO modificar df

# CORRECTO — una sola operación .loc con ambas condiciones
df.loc[df["precio"] > 100, "en_oferta"] = True
```

Explicación completa del porqué y de Copy-on-Write en [[17 - Buenas Prácticas, Errores Comunes y Comparativa]].

## Ver también

- [[03 - Exploración e Inspección de Datos]]
- [[05 - Filtros y Condiciones Avanzadas]]
- [[13 - MultiIndex y Datos Jerárquicos]]
- [[17 - Buenas Prácticas, Errores Comunes y Comparativa]]
