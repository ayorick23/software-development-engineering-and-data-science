---
tags: [scipy, fourier, cheat-sheet]
---

# 11 — Transformada de Fourier (`scipy.fft`)

## El concepto

La Transformada de Fourier descompone una señal (una serie de valores en el tiempo) en las frecuencias que la componen — convierte del **dominio del tiempo** al **dominio de la frecuencia**. Permite responder "¿qué tan periódica es esta señal, y con qué período?" sin inspeccionarla visualmente.

## FFT básica

```python
import numpy as np
from scipy.fft import fft, fftfreq, ifft

fs = 500          # frecuencia de muestreo (Hz)
t = np.linspace(0, 1, fs, endpoint=False)
senal = np.sin(2 * np.pi * 50 * t) + 0.5 * np.sin(2 * np.pi * 120 * t)   # dos frecuencias: 50 Hz y 120 Hz

espectro = fft(senal)
frecuencias = fftfreq(len(senal), d=1/fs)   # eje de frecuencias correspondiente

magnitud = np.abs(espectro)   # la FFT devuelve números complejos; la magnitud es lo interpretable
```

Al graficar `frecuencias` vs `magnitud` (solo la mitad positiva, por simetría), deberían aparecer picos claros en 50 Hz y 120 Hz — exactamente las frecuencias inyectadas en la señal de ejemplo.

## Transformada inversa

```python
senal_reconstruida = ifft(espectro)   # debe recuperar (aprox.) la señal original
```

## Filtrado en el dominio de la frecuencia

Una forma directa de eliminar ruido: pasar al dominio de frecuencia, poner a cero las frecuencias no deseadas, y volver:

```python
espectro_filtrado = espectro.copy()
espectro_filtrado[np.abs(frecuencias) > 100] = 0   # elimina todo por encima de 100 Hz

senal_filtrada = ifft(espectro_filtrado).real   # .real: descarta la parte imaginaria residual (ruido numérico)
```

Para filtrado más controlado (con transición suave entre bandas), preferir los filtros diseñados de [[08 - Procesamiento de Señales (scipy.signal)|scipy.signal]] (`butter` + `filtfilt`) en vez de este "corte duro" en frecuencia.

## FFT en 2D (imágenes)

```python
from scipy.fft import fft2, ifft2

imagen = np.random.rand(100, 100)
espectro_2d = fft2(imagen)
```

Base de técnicas de compresión y filtrado de imágenes (el mismo principio detrás de la compresión JPEG, simplificando mucho).

## Ver también

- [[08 - Procesamiento de Señales (scipy.signal)]]
