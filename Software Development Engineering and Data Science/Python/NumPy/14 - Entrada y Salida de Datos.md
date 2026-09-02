---
tags: [numpy, python, data-science, io, cheat-sheet]
---

# 14 — Entrada y Salida de Datos

> Continúa de [[13 - Números Aleatorios (Generator API)]].

## Formato binario nativo: `.npy` y `.npz`

```python
arr = np.array([1, 2, 3, 4, 5])

np.save("datos.npy", arr)             # guarda UN array
cargado = np.load("datos.npy")          # lo recupera exactamente igual (mismo dtype, mismo shape)

np.savez("datos.npz", a=arr, b=arr*2)     # guarda VARIOS arrays con nombre, en un solo archivo
datos = np.load("datos.npz")
datos["a"], datos["b"]                       # acceso por nombre, como un diccionario

np.savez_compressed("datos_comprimidos.npz", a=arr, b=arr*2)   # igual que savez, pero comprimido (más lento de escribir, menos espacio)
```

El formato `.npy`/`.npz` preserva el `dtype` y `shape` exactos — a diferencia de un CSV, no hay ninguna inferencia de tipos al releer, por lo que es preferible sobre texto plano para guardar resultados intermedios puramente numéricos que se van a re-cargar con NumPy.

## Texto plano: `genfromtxt()` y `loadtxt()`

```python
np.loadtxt("datos.csv", delimiter=",")                          # rápido, pero NO tolera valores faltantes
np.genfromtxt("datos.csv", delimiter=",", skip_header=1, filling_values=0)   # más lento, tolera nulos/columnas faltantes

np.savetxt("salida.csv", arr, delimiter=",", fmt="%.2f")           # exportar a texto plano
```

| | `loadtxt()` | `genfromtxt()` |
|---|---|---|
| Velocidad | Más rápida | Más lenta |
| Valores faltantes | Falla si encuentra huecos | Los rellena con `filling_values` |
| Uso típico | Archivos numéricos limpios y completos | Archivos reales con datos faltantes/ruido |

Para datos tabulares reales con columnas de texto, fechas y nulos, [[Python/Pandas/02 - Creación y Carga de Datos|`pd.read_csv()`]] es casi siempre preferible a ambas — estas funciones de NumPy son para arrays puramente numéricos.

## `memmap` — trabajar con archivos más grandes que la RAM

```python
mm = np.memmap("archivo_enorme.dat", dtype="float32", mode="r", shape=(1_000_000, 100))
mm[0:10]          # solo lee del disco la porción accedida, no el archivo completo
```

Un `memmap` mapea un archivo en disco como si fuera un array en memoria, cargando páginas bajo demanda — permite indexar/operar sobre secciones de un archivo mucho más grande que la RAM disponible sin cargarlo completo. Ver el patrón equivalente por lotes en Pandas (`chunksize`) en [[Python/Pandas/02 - Creación y Carga de Datos#Lectura en trozos (chunksize) para archivos que no caben en memoria|Pandas]].

## Serialización con `pickle` (uso ocasional)

```python
import pickle
with open("modelo_datos.pkl", "wb") as f:
    pickle.dump(arr, f)
```

`pickle` funciona con arrays de NumPy, pero **no** es portable entre versiones de Python/NumPy de forma tan confiable como `.npy` — usar `.npy`/`.npz` para persistencia pura de arrays, y reservar `pickle` para estructuras mixtas más complejas (ver también persistencia de modelos completos en [[13 - Persistencia de Modelos|Scikit-learn]]).

## Ver también

- [[13 - Números Aleatorios (Generator API)]]
- [[Python/Pandas/02 - Creación y Carga de Datos|Python/Pandas]]
- [[Python/Pandas/14 - Exportación de Datos|Python/Pandas — Exportación]]
