---
tags: [pytest, python, testing, marks, cheat-sheet]
---

# 06 — Marks y Selección de Tests

> Continúa de [[05 - Parametrización de Tests]].

## `@pytest.mark.skip` — saltar un test incondicionalmente

```python
@pytest.mark.skip(reason="funcionalidad todavía no implementada")
def test_feature_futura():
    ...
```

El test aparece en la salida como `SKIPPED`, con la razón visible — no cuenta como fallo, pero deja constancia explícita de que existe y por qué no corre.

## `@pytest.mark.skipif` — saltar condicionalmente

```python
import sys

@pytest.mark.skipif(sys.version_info < (3, 10), reason="requiere pattern matching de Python 3.10+")
def test_con_match_statement():
    ...

@pytest.mark.skipif(not tiene_gpu_disponible(), reason="requiere GPU")
def test_entrenamiento_en_gpu():
    ...
```

`skipif` evalúa una condición **en el momento de la colección** de tests — común para tests que dependen de la versión de Python, el sistema operativo, o la disponibilidad de un recurso externo (GPU, una API con clave configurada).

## `@pytest.mark.xfail` — se espera que falle

```python
@pytest.mark.xfail(reason="bug conocido, ver ticket JIRA-1234")
def test_con_bug_conocido():
    assert funcion_con_bug() == resultado_correcto     # FALLA, pero se reporta como XFAIL, no como FAILED
```

`xfail` documenta un fallo **conocido y aceptado** (a diferencia de `skip`, que ni siquiera ejecuta el test) — si el test *pasa* inesperadamente, pytest lo reporta como `XPASS`, señal de que el bug pudo haberse arreglado y el `xfail` ya no es necesario.

```python
@pytest.mark.xfail(strict=True)     # con strict=True, un XPASS se convierte en un FALLO real de la suite
def test_estricto():
    ...
```

`strict=True` es útil para forzar que alguien actualice el `xfail` cuando el bug subyacente se corrige, en vez de dejar un `XPASS` silencioso acumulándose indefinidamente.

## Marks personalizados — categorizar tests propios

```python
@pytest.mark.lento
def test_procesamiento_completo():
    ...

@pytest.mark.integracion
def test_conexion_real_a_bd():
    ...
```

```toml
# pyproject.toml — registrar marks propios evita warnings de "mark desconocido"
[tool.pytest.ini_options]
markers = [
    "lento: tests que tardan más de unos segundos",
    "integracion: tests que requieren servicios externos reales",
]
```

Registrar los marks personalizados en la configuración (ver [[02 - Escritura y Descubrimiento de Tests#Configuración pytest.ini pyproject.toml|Configuración]]) no es obligatorio para que funcionen, pero evita un `PytestUnknownMarkWarning` y sirve como documentación de qué categorías de test existen en el proyecto.

## Seleccionar tests por mark o por nombre

```bash
pytest -m lento                          # solo tests marcados @pytest.mark.lento
pytest -m "not lento"                      # todos EXCEPTO los marcados como lentos — común en CI para un feedback rápido
pytest -m "lento or integracion"             # combinar marks con operadores booleanos
pytest -k "suma"                               # por NOMBRE del test (no por mark) — ver 02
```

El patrón `pytest -m "not lento"` en un pipeline de CI (ejecutar rápido en cada commit) combinado con `pytest -m lento` en un job nocturno separado (ejecutar la suite completa, incluyendo lo costoso) es un patrón común de organización de suites grandes — ver [[16 - Integración con CI-CD]].

## `pytest.importorskip` — saltar si una dependencia opcional no está instalada

```python
numpy = pytest.importorskip("numpy")     # salta TODO el módulo de test si numpy no está instalado

def test_con_numpy():
    assert numpy.array([1, 2, 3]).sum() == 6
```

## Ver también

- [[05 - Parametrización de Tests]]
- [[07 - Manejo de Excepciones y Warnings]]
- [[16 - Integración con CI-CD]]
