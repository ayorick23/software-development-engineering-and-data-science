---
tags: [scipy, buenas-practicas, cheat-sheet]
---

# 13 — Buenas Prácticas, Errores Comunes y Comparativa

## Errores comunes

- **Olvidar importar el submódulo específico.** `import scipy` no da acceso a `scipy.stats` ni a ningún otro submódulo — hay que importarlo explícitamente (`from scipy import stats`).
- **Confundir los parámetros de `scipy.stats.uniform`.** `uniform(loc, scale)` es origen + ancho, no mínimo + máximo. `uniform(loc=10, scale=5)` es $U(10, 15)$, no $U(10, 5)$.
- **No revisar `result.success` en `scipy.optimize.minimize`.** Un optimizador puede terminar sin converger y devolver un `result.x` que no es el mínimo real.
- **Usar `1 - cdf(x)` en vez de `sf(x)` para colas extremas.** Cuando `cdf(x)` es muy cercano a 1 (por ejemplo 0.999999998), restarlo de 1 pierde precisión por redondeo de punto flotante; `sf(x)` (survival function) calcula la cola directamente y es numéricamente más estable.
- **Densificar una matriz dispersa grande sin necesidad.** Llamar `.toarray()` sobre una matriz `scipy.sparse` de millones de columnas puede agotar la memoria disponible — solo densificar si el resultado es genuinamente pequeño.
- **Aplicar pruebas paramétricas (t-test, ANOVA) sin verificar sus supuestos.** Ver [[05 - Estadística II - Pruebas de Hipótesis (scipy.stats)]] y [[Estadística Aplicada/08 - Supuestos Estadísticos Detrás de los Modelos de ML]].

## SciPy vs NumPy — cuándo usar cuál

| | NumPy | SciPy |
|---|---|---|
| Alcance | Arrays, aritmética, álgebra lineal básica | Algoritmos científicos especializados |
| `linalg` | Suficiente para uso básico | Más completo, siempre usa BLAS/LAPOACK completo |
| Estadística | Casi nada (`np.mean`, `np.std`) | Distribuciones completas, pruebas de hipótesis |
| Dependencia | Básicamente universal en el ecosistema Python científico | Depende de NumPy; no siempre necesaria |

Regla práctica: empezar con NumPy; recurrir a SciPy en cuanto se necesite algo que NumPy no ofrece (una distribución, un optimizador, una descomposición avanzada).

## SciPy vs statsmodels — para estadística

| | `scipy.stats` | `statsmodels` |
|---|---|---|
| Enfoque | Funciones puntuales: pruebas, distribuciones | Modelos completos con resumen (`summary()`) |
| Salida típica | `(estadístico, p-valor)` | Objeto con coeficientes, errores estándar, IC, diagnósticos |
| Series de tiempo | No | Sí (ARIMA, SARIMAX, descomposición) |
| Uso típico | "¿Es significativa esta diferencia?" | "¿Cuál es el efecto de X sobre Y, con qué incertidumbre?" |

## SciPy vs scikit-learn

No son sustitutos: scikit-learn resuelve problemas de aprendizaje automático (entrenar con datos etiquetados y predecir sobre datos nuevos); SciPy provee las herramientas matemáticas de propósito general que scikit-learn usa internamente. Un flujo de trabajo típico usa ambos: SciPy para explorar/ajustar/validar estadísticamente, scikit-learn para el modelo predictivo final.

## Checklist antes de usar una prueba estadística de `scipy.stats`

1. ¿Se verificó el supuesto de normalidad si la prueba lo requiere (`shapiro`)?
2. ¿Se usó `equal_var=False` en `ttest_ind` salvo que haya razón fuerte para asumir varianzas iguales?
3. ¿Se interpretó el p-valor correctamente (no como "probabilidad de que H0 sea cierta")?
4. Si se hicieron múltiples pruebas sobre los mismos datos, ¿se corrigió por comparaciones múltiples? (ver [[Estadística Aplicada/06 - A-B Testing de Principio a Fin]])

## Ver también

- [[01 - Introducción y Arquitectura]]
- [[12 - Integración con el Ecosistema]]
- [[Estadística Aplicada/10 - Integración con el Ecosistema y Comparativa]]
