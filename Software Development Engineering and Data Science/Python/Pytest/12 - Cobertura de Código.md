---
tags: [pytest, python, testing, coverage, cheat-sheet]
---

# 12 — Cobertura de Código

> Continúa de [[11 - Plugins del Ecosistema]].

## `pytest-cov` — integración de `coverage.py` con pytest

```bash
pip install pytest-cov
pytest --cov=mi_paquete                    # mide cobertura del paquete 'mi_paquete' mientras corren los tests
pytest --cov=mi_paquete --cov-report=html    # genera un reporte HTML navegable en htmlcov/
pytest --cov=mi_paquete --cov-report=term-missing   # muestra en terminal las líneas EXACTAS no cubiertas
```

```
Name                    Stmts   Miss  Cover   Missing
-----------------------------------------------------
mi_paquete/calculo.py       45      3    93%   28-30
mi_paquete/validacion.py    20      0   100%
-----------------------------------------------------
TOTAL                       65      3    95%
```

`term-missing` es el reporte más útil durante desarrollo activo: señala exactamente qué números de línea nunca se ejecutaron durante la suite, permitiendo escribir un test dirigido a esa rama específica.

## Qué mide realmente la cobertura (y qué no)

Ver la advertencia conceptual ya establecida en [[Testing#Conceptos Fundamentales de Testing|Python/Testing]]: cobertura mide **qué líneas se ejecutaron**, no si el comportamiento fue verificado correctamente. Un test que llama a una función sin ningún `assert` sobre su resultado cuenta como "cobertura" completa de esa función, aunque no verifique absolutamente nada.

```python
# Esto cuenta como 100% de cobertura de la función, pero NO prueba nada realmente
def test_inutil():
    calcular_impuesto(100)     # se ejecuta, pero no hay ningún assert
```

## Cobertura de ramas (branch coverage) — más estricta que cobertura de líneas

```ini
# pytest.ini o .coveragerc
[coverage:run]
branch = True
```

```bash
pytest --cov=mi_paquete --cov-branch
```

Cobertura de **líneas** solo verifica que cada línea se ejecutó al menos una vez; cobertura de **ramas** verifica que cada posible camino de un `if`/`else` se ejecutó — una función con un `if` puede tener 100% de cobertura de líneas ejecutando solo la rama `if`, pero solo 50% de cobertura de ramas si la rama `else` nunca corrió en ningún test.

## Fijar un umbral mínimo — fallar la suite si la cobertura baja

```bash
pytest --cov=mi_paquete --cov-fail-under=80     # la suite FALLA si la cobertura total es menor a 80%
```

```toml
# pyproject.toml
[tool.coverage.report]
fail_under = 80
exclude_lines = [
    "pragma: no cover",
    "if TYPE_CHECKING:",
    "raise NotImplementedError",
]
```

Un umbral en CI (ver [[16 - Integración con CI-CD]]) evita que la cobertura se degrade silenciosamente con el tiempo a medida que se agrega código nuevo sin tests correspondientes — `exclude_lines` permite marcar explícitamente código que legítimamente no necesita cobertura (código solo para type-checking, ramas defensivas imposibles de alcanzar en la práctica).

## Excluir líneas/archivos específicos

```python
def funcion_dificil_de_testear():          # pragma: no cover
    ...
```

```toml
[tool.coverage.run]
omit = ["*/migrations/*", "*/tests/*", "manage.py"]
```

## El número de cobertura no es el objetivo — es un indicador

**Regla práctica:** perseguir un 100% de cobertura como meta en sí misma frecuentemente produce tests de baja calidad (como el ejemplo de arriba, que "cubre" código sin verificar nada) escritos solo para subir el número. Un umbral razonable (70-85% según el proyecto) combinado con revisión humana de qué se está probando realmente es más valioso que perseguir el 100% a cualquier costo — ver la discusión completa de calidad de tests en [[Machine Learning/13-Testing-en-Machine-Learning|Machine Learning/13 - Testing en Machine Learning]].

## Ver también

- [[11 - Plugins del Ecosistema]]
- [[16 - Integración con CI-CD]]
- [[Testing#Conceptos Fundamentales de Testing|Python/Testing]]
- [[Machine Learning/13-Testing-en-Machine-Learning|Machine Learning/13 - Testing en Machine Learning]]
