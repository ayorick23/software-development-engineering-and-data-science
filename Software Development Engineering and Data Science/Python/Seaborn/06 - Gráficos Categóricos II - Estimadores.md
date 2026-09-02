---
tags: [seaborn, python, data-science, visualization, categorical, cheat-sheet]
---

# 06 — Gráficos Categóricos II: Estimadores

> Continúa de [[05 - Gráficos Categóricos I - Dispersión y Conteo]].

## `boxplot()` — diagrama de caja con sintaxis simplificada

```python
sns.boxplot(data=df, x="region", y="ventas")
sns.boxplot(data=df, x="region", y="ventas", hue="canal")     # cajas agrupadas por segunda categoría, automático
sns.boxplot(data=df, x="region", y="ventas", showfliers=False)  # oculta outliers individuales
```

Comparado con `ax.boxplot()` de Matplotlib puro (ver [[Python/Matplotlib/06 - Gráficos Estadísticos|Python/Matplotlib]]), la ventaja de `sns.boxplot` es que `hue=` agrupa automáticamente sub-cajas dentro de cada categoría del eje X, sin necesitar calcular posiciones manualmente.

## `violinplot()` — distribución completa por categoría

```python
sns.violinplot(data=df, x="region", y="ventas")
sns.violinplot(data=df, x="region", y="ventas", hue="canal", split=True)   # divide cada violín a la mitad, una mitad por valor de hue
```

`split=True` con exactamente dos niveles de `hue` dibuja cada mitad del violín para una de las dos categorías — permite comparar dos grupos directamente lado a lado dentro del mismo violín en vez de dos violines separados, aprovechando mejor el espacio horizontal.

## `boxenplot()` — para datasets grandes con colas largas

```python
sns.boxenplot(data=df, x="region", y="ventas")
```

También llamado "letter-value plot" — dibuja más niveles de cajas anidadas que un boxplot tradicional (que solo muestra cuartiles), mostrando más detalle sobre la forma de las colas de la distribución. Diseñado específicamente para datasets con miles/millones de observaciones, donde un boxplot tradicional marcaría una cantidad excesiva de "outliers" individuales.

## `barplot()` — barras con estimador e intervalo de confianza

```python
sns.barplot(data=df, x="region", y="ventas")                          # altura = MEDIA por default, con barra de error = IC 95%
sns.barplot(data=df, x="region", y="ventas", estimator="median")        # cambiar el estimador
sns.barplot(data=df, x="region", y="ventas", errorbar="sd")               # barra de error = desviación estándar en vez de IC
sns.barplot(data=df, x="region", y="ventas", hue="canal")
```

**Diferencia clave con `ax.bar()` de Matplotlib:** `sns.barplot()` recibe los datos **crudos** (una fila por observación) y calcula el estimador internamente; `ax.bar()` de Matplotlib espera que el valor a graficar ya venga pre-agregado. Esto es lo mismo que la distinción figure-level/axes-level pero aplicada a agregación de datos — ver [[13 - Estadística Integrada]] para el detalle del cálculo del intervalo de confianza.

## `pointplot()` — estimadores conectados por línea

```python
sns.pointplot(data=df, x="mes", y="ventas", hue="region")
```

Igual que `barplot()` en el cálculo (estimador + intervalo de confianza), pero representado como puntos conectados por una línea en vez de barras — más legible que `barplot` cuando hay muchas categorías en el eje X, porque las líneas conectando puntos comunican tendencia de forma más directa que comparar alturas de barras adyacentes.

## Tabla resumen: todos los gráficos categóricos

| Función | Muestra | Mejor para |
|---|---|---|
| `stripplot` | Cada observación individual (con jitter) | Ver la muestra completa, datasets pequeños-medianos |
| `swarmplot` | Cada observación, sin superposición | Igual, con mejor lectura de densidad, datasets pequeños |
| `boxplot` | Cuartiles, mediana, outliers | Comparación rápida de resumen estadístico |
| `violinplot` | Densidad completa (KDE) | Cuando la forma/multimodalidad de la distribución importa |
| `boxenplot` | Múltiples niveles de cuantiles | Datasets grandes con colas largas |
| `barplot` | Estimador (media/mediana) + IC | Comparar un solo número resumen entre categorías |
| `pointplot` | Estimador + IC, conectado por línea | Tendencia a través de muchas categorías ordenadas |
| `countplot` | Frecuencia/conteo | Cuántas observaciones hay por categoría |

## Ver también

- [[05 - Gráficos Categóricos I - Dispersión y Conteo]]
- [[07 - Gráficos de Regresión]]
- [[13 - Estadística Integrada]]
- [[Python/Matplotlib/06 - Gráficos Estadísticos|Python/Matplotlib]]
