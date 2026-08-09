---
Fecha de creación: 2026-02-13T18:28:00
Materia:
  - Probabilidad y Estadística
Fecha de clase: 2026-02-13
---
[[Clase 03 - Tipos de Gráficos Estadísticos|← Clase anterior]] | [[Clase 05 - Laboratorio 1|Clase siguiente →]]

# Medidas de Tendencia Central

Las **medidas de tendencia central** resumen un conjunto de datos mediante un valor representativo que indica dónde se concentran los datos.

Las principales son:

- Media
- Mediana
- Moda

## Media (Promedio)

Es el promedio calculado sumando todos los datos y dividiendo entre el total de observaciones. Es sensible a valores extremos.

### Para datos no agrupados

$$
\bar{x} = \frac{\sum x_i}{n}
$$

Donde:

- $x_i$ = cada dato
- $n$ = número total de datos

### Para datos agrupados

Cuando los datos están en intervalos:

$$
\bar{x} = \frac{\sum f_im_i}{n}
$$

Donde:

- $f_i$ = frecuencia absoluta
- $m_i$ = marca de clase
- $n$ = $\sum f_i$

## Mediana

Es el valor que divide los datos ordenados en dos partes iguales.

### Para datos no agrupados

Si $n$ es impar:

$$
Me = x_{\frac{n+1}{2}}
$$

Si $n$ es par:

$$
Me = \frac{x_{n/2}+x_{x/2+1}}{2}
$$

### Para datos agrupados

$$
Me = L_i + (\frac{\frac{n}{2} - F_{\text{anterior}}}{f_i})A
$$

Donde:

- $L_i$ = límite inferior de la clase mediana
- $n$ = total de datos
- $F_{\text{anterior}}$ = frecuencia acumulada anterior
- $f_i$ = frecuencia de la clase mediana
- $A$ = amplitud

## Moda

Valor que más se repite.

### Para datos agrupados

$$
Mo = L_i + (\frac{f_i - f_{\text{anterior}}}{(f_i - f_{\text{anterior}}) + (f_i - f_{\text{siguiente}})})A
$$

## Percentiles

Dividen los datos en 100 partes iguales.

Ejemplos:

- P50 = mediana
- P25 = primer cuartil
- P75 = tercer cuartil

### Para datos agrupados

$$
P_k = L_i + (\frac{\frac{k}{100}n-F_{anterior}}{f_i})A
$$

### Ejemplo de Aplicación

Supongamos los tiempos de atención (minutos) de 20 clientes:

```
4, 7, 10, 6, 9, 8, 5, 12, 15, 6,
9, 11, 7, 8, 10, 14, 13, 9, 6, 8
```

#### Paso 1: Calcular Rango

$$
R = 15 - 4 = 11
$$

#### Paso 2: Número de intervalos (Sturges)

$$
k = 1 + 3.322\log(20) \approx 5
$$

#### Paso 3: Amplitud

$$
A = \frac{11}{5} \approx 3
$$

#### Paso 4: Tabla de Frecuencia

| Intervalo | Marca ($mi$) | $f_i$ | $F_i$ | $h_i$ |
| --------- | ------------ | ----- | ----- | ----- |
| 4–6       | 5            | 5     | 5     | 0.25  |
| 7–9       | 8            | 7     | 12    | 0.35  |
| 10–12     | 11           | 5     | 17    | 0.25  |
| 13–15     | 14           | 3     | 20    | 0.15  |

#### Media Agrupada

$$
 = \frac{(5\times5) + (7\times8) + (5\times11) + (3\times14)}{20}
$$
$$
= \frac{25+56+55+42}{20}
$$
$$
= \frac{178}{20} = 8.9
$$
#### Moda Agrupada

$$
Mo = 7+(\frac{7-5}{(7-5)+(7-5)})3
$$
$$
Mo = 7 + \frac{2}{4}\times3
$$
$$
Mo = 7 + 1.5 = 8.5
$$

#### Percentil 75

$$
\frac{75}{100}(20) = 15
$$

Cae en la clase 10-12.

$$
P_{75} = 10 + (\frac{15-12}{5})3
$$
$$
P_{75} = 10 + 1.8 = 11.8
$$

### Ejemplo recreado en Excel

#### Paso 1: Datos originales

Coloca los 20 datos en `A1:A20`.

#### Media

```excel
=AVERAGE(A1:A20)
```

#### Mediana

```excel
=MEDIAN(A1:A20)
```

#### Moda

```excel
=MODE.SNGL(A1:A20)
```

#### Percentil 75

```excel
=PERCENTILE.EXC(A1:A20,0.75)
```

>[!IMPORTANT] `.EXC` excluye extremos.

También existe:

```excel
=PERCENTILE.INC(A1:A20,0.75)
```

### Para datos agrupados

Debes crear columnas:

- Intervalo
- Marca de clase → `(Límite inferior + superior)/2`
- fi → usar `COUNTIFS`
- Fi → suma acumulada
- fi * mi
- Suma total

Media agrupada:

```excel
=SUMPRODUCT(rango_fi,rango_mi)/SUM(rango_fi)
```

# Interpretación Final

- Media ≈ 8.9
- Mediana ≈ 9.14
- Moda ≈ 8.5
- P75 ≈ 11.8

 >[!IMPORTANT] El 75% de los clientes fue atendido en menos de 11.8 minutos.
