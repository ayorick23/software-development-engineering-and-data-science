---
tags: [scipy, procesamiento-de-senales, cheat-sheet]
---

# 08 — Procesamiento de Señales (`scipy.signal`)

## Convolución y correlación

```python
import numpy as np
from scipy import signal

senal = np.array([1, 2, 3, 4, 5])
kernel = np.array([0.25, 0.5, 0.25])   # suavizado simple

signal.convolve(senal, kernel, mode='same')     # 'full', 'same', 'valid' controlan el tamaño de salida
signal.correlate(senal, kernel, mode='same')    # correlación cruzada — útil para detectar dónde aparece un patrón
```

## Filtros digitales

Diseñar y aplicar un filtro pasa-bajos (elimina ruido de alta frecuencia, común en series de tiempo de sensores):

```python
from scipy.signal import butter, filtfilt

# Filtro Butterworth de orden 4, frecuencia de corte 0.1 (normalizada respecto a Nyquist)
b, a = butter(N=4, Wn=0.1, btype='low')

senal_filtrada = filtfilt(b, a, senal_ruidosa)   # filtfilt: sin desfase temporal (filtra hacia adelante y atrás)
```

| Tipo de filtro | `btype` | Uso típico |
|---|---|---|
| Pasa-bajos | `'low'` | Eliminar ruido de alta frecuencia |
| Pasa-altos | `'high'` | Eliminar tendencia/deriva lenta |
| Pasa-banda | `'bandpass'` | Aislar una banda de frecuencia específica |

## Suavizado con Savitzky-Golay

Alternativa a un filtro pasa-bajos que preserva mejor los picos y la forma de la señal original:

```python
from scipy.signal import savgol_filter

suavizada = savgol_filter(senal_ruidosa, window_length=11, polyorder=3)
```

## Detección de picos

Común en análisis de señales biomédicas (ECG), espectros, o series de tiempo con eventos discretos:

```python
from scipy.signal import find_peaks

picos, propiedades = find_peaks(senal, height=0, distance=5)
# height: altura mínima; distance: separación mínima entre picos consecutivos
```

## Análisis espectral: densidad de potencia

Identificar las frecuencias dominantes de una señal (complementa la [[11 - Transformada de Fourier (scipy.fft)|FFT]], pero con estimación estadística más estable frente a ruido):

```python
from scipy.signal import welch

frecuencias, potencia = welch(senal, fs=100)   # fs: frecuencia de muestreo en Hz
```

## Remuestreo

Cambiar la frecuencia de muestreo de una señal (por ejemplo, de 1000 Hz a 100 Hz):

```python
from scipy.signal import resample

senal_remuestreada = resample(senal, num=len(senal) // 10)
```

## Ver también

- [[11 - Transformada de Fourier (scipy.fft)]]
- [[07 - Integración y Ecuaciones Diferenciales (scipy.integrate)]]
