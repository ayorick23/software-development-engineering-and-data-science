---
tags: [pytest, python, testing, plugins, cheat-sheet]
---

# 11 — Plugins del Ecosistema

> Continúa de [[10 - conftest.py y Organización de Proyectos]]. El ecosistema de plugins es una de las razones centrales por las que pytest domina sobre `unittest`.

## `pytest-xdist` — paralelizar la ejecución de tests

```bash
pip install pytest-xdist
pytest -n 4          # ejecuta los tests distribuidos en 4 procesos en paralelo
pytest -n auto          # usa automáticamente el número de núcleos disponibles
```

En suites grandes (cientos/miles de tests), correr en paralelo con `pytest-xdist` puede reducir el tiempo total drásticamente — el requisito es que los tests sean **independientes entre sí** (sin estado compartido implícito), ya que el orden de ejecución entre procesos no está garantizado.

## `pytest-timeout` — evitar que un test cuelgue la suite indefinidamente

```bash
pip install pytest-timeout
pytest --timeout=30          # cualquier test que tarde más de 30s se marca como fallido automáticamente
```

```python
@pytest.mark.timeout(5)     # timeout específico para un solo test
def test_operacion_rapida():
    ...
```

Sin esto, un test que se cuelga (por ejemplo, esperando una respuesta de red que nunca llega) puede bloquear la suite completa indefinidamente en CI, sin ninguna señal clara de qué test es el responsable.

## `pytest-randomly` — aleatorizar el orden de ejecución

```bash
pip install pytest-randomly
pytest     # se activa automáticamente al instalarse, reordena tests en cada corrida
```

Aleatorizar el orden es una forma efectiva de **detectar dependencias ocultas entre tests** (un test que solo pasa porque otro se ejecutó antes y dejó cierto estado global) — un problema de diseño que un orden siempre fijo puede esconder indefinidamente. El plugin imprime la semilla usada, permitiendo reproducir exactamente el mismo orden si se necesita depurar un fallo específico.

## `pytest-sugar` — salida visual mejorada

```bash
pip install pytest-sugar
```

Reemplaza la salida estándar de puntos/letras por una barra de progreso y colores más legibles — puramente cosmético, sin afectar el comportamiento de los tests, pero mejora la experiencia de desarrollo diario.

## `pytest-rerunfailures` — reintentar tests inestables (flaky)

```bash
pip install pytest-rerunfailures
pytest --reruns 3 --reruns-delay 2     # reintenta hasta 3 veces, esperando 2s entre intentos
```

Útil como mitigación temporal para tests genuinamente inestables (dependientes de timing, red, servicios externos intermitentes) — **no** es una solución al problema de fondo, solo reduce el ruido de falsos negativos mientras se investiga la causa raíz de la inestabilidad.

## `pytest-env` — variables de entorno declarativas para tests

```ini
# pytest.ini
[pytest]
env =
    API_KEY=clave-de-prueba
    ENTORNO=testing
```

Alternativa declarativa a usar `monkeypatch.setenv()` (ver [[09 - Mocking y Monkeypatching#La fixture monkeypatch — la forma nativa de pytest|Mocking]]) en cada test individual, cuando las mismas variables de entorno deben aplicar a toda la suite.

## Tabla resumen: cuándo instalar cada plugin

| Necesito... | Plugin |
|---|---|
| Ejecutar la suite más rápido, en paralelo | `pytest-xdist` |
| Medir/exigir cobertura de código | `pytest-cov` (ver [[12 - Cobertura de Código]]) |
| Testear código async | `pytest-asyncio` (ver [[13 - Testing Asíncrono]]) |
| Mocking con sintaxis de fixture | `pytest-mock` (ver [[09 - Mocking y Monkeypatching]]) |
| Evitar que un test cuelgue la suite | `pytest-timeout` |
| Detectar dependencias ocultas entre tests | `pytest-randomly` |
| Mitigar tests inestables temporalmente | `pytest-rerunfailures` |
| Property-based testing | `hypothesis` (ver [[15 - Property-Based Testing con Hypothesis]]) |

## Ver también

- [[10 - conftest.py y Organización de Proyectos]]
- [[12 - Cobertura de Código]]
- [[13 - Testing Asíncrono]]
