---
tags: [pytest, python, testing, mocking, cheat-sheet]
---

# 09 — Mocking y Monkeypatching

> Continúa de [[08 - Assertions Avanzadas]]. Cómo aislar el código bajo prueba de sus dependencias externas.

## Por qué se necesita mocking

Un test unitario debe probar **una sola unidad** de código de forma aislada — si esa unidad depende de una API externa, una base de datos, o el reloj del sistema, el test se vuelve lento, no reproducible, o directamente imposible de correr en CI sin credenciales reales. El **mocking** reemplaza esas dependencias por objetos falsos controlados durante el test.

## La fixture `monkeypatch` — la forma nativa de pytest

```python
def test_variable_de_entorno(monkeypatch):
    monkeypatch.setenv("API_KEY", "clave-de-prueba")
    assert obtener_api_key() == "clave-de-prueba"

def test_directorio_actual(monkeypatch, tmp_path):
    monkeypatch.chdir(tmp_path)
    assert Path.cwd() == tmp_path

def test_atributo_de_modulo(monkeypatch):
    monkeypatch.setattr("mi_modulo.URL_BASE", "http://test.local")
    assert obtener_url() == "http://test.local/endpoint"
```

`monkeypatch` es una fixture incluida en pytest (no requiere instalar nada extra) que reemplaza temporalmente variables de entorno, atributos, entradas de diccionario o el directorio de trabajo — y **revierte automáticamente** el cambio al terminar el test, sin necesitar teardown manual.

```python
def test_input_simulado(monkeypatch):
    monkeypatch.setattr("builtins.input", lambda _: "s")     # simula que el usuario escribe "s" en cualquier input()
    assert confirmar_accion() is True
```

## `unittest.mock` — la librería estándar de mocking

```python
from unittest.mock import Mock, MagicMock, patch

def test_con_mock_simple():
    servicio_falso = Mock()
    servicio_falso.obtener_precio.return_value = 100

    resultado = calcular_total(servicio_falso, cantidad=3)
    assert resultado == 300
    servicio_falso.obtener_precio.assert_called_once()      # verifica que el método SÍ fue llamado
```

Un `Mock()` acepta llamar **cualquier** atributo/método sin definirlo de antemano, devolviendo otro `Mock` automáticamente — útil para simular objetos complejos sin implementar una versión falsa completa.

## `patch()` — reemplazar temporalmente una dependencia real

```python
from unittest.mock import patch

@patch("mi_modulo.requests.get")
def test_llamada_api(mock_get):
    mock_get.return_value.json.return_value = {"precio": 100}
    resultado = obtener_precio_externo()
    assert resultado == 100
    mock_get.assert_called_once_with("https://api.ejemplo.com/precio")
```

```python
def test_con_context_manager():
    with patch("mi_modulo.requests.get") as mock_get:
        mock_get.return_value.status_code = 200
        assert verificar_servicio() is True
```

**Regla crítica de `patch()`:** se debe parchear el objeto **donde se usa**, no donde se define — `@patch("mi_modulo.requests.get")` funciona porque `mi_modulo` importó `requests` y lo referencia como `requests.get` dentro de su propio namespace; parchear `"requests.get"` directamente no afectaría la referencia ya importada dentro de `mi_modulo`.

## `pytest-mock` — la fixture `mocker`, un envoltorio más cómodo

```python
def test_con_mocker(mocker):
    mock_get = mocker.patch("mi_modulo.requests.get")
    mock_get.return_value.json.return_value = {"precio": 100}

    resultado = obtener_precio_externo()
    assert resultado == 100
```

`pytest-mock` (plugin externo, ver [[11 - Plugins del Ecosistema]]) expone `unittest.mock.patch` como la fixture `mocker`, con la ventaja de que **revierte automáticamente** cada patch al final del test sin necesitar el decorador o el `with` — más consistente con el estilo de fixtures del resto de pytest.

## Cuándo mockear y cuándo no

| Situación | ¿Mockear? |
|---|---|
| Llamada HTTP a una API externa real | Sí — el test no debe depender de red/disponibilidad de terceros |
| Base de datos de producción | Sí, o usar una de prueba real (ver [[14 - Testing de Bases de Datos e Infraestructura]]) |
| Lógica de negocio pura dentro del propio código | No — mockear demasiado interno hace que el test verifique la implementación, no el comportamiento |
| El reloj del sistema (`datetime.now()`) | Sí, si el resultado depende de la fecha/hora actual |

**Advertencia de sobre-mockeo:** un test que mockea casi todo lo que toca termina verificando "que se llamó a X con estos argumentos" en vez de "que el resultado es correcto" — un exceso de mocks hace que los tests pasen incluso cuando la lógica real está rota, porque nunca ejecutan código real. Ver más en [[17 - Buenas Prácticas, Errores Comunes y Comparativa]].

## Ver también

- [[08 - Assertions Avanzadas]]
- [[10 - conftest.py y Organización de Proyectos]]
- [[11 - Plugins del Ecosistema]]
- [[17 - Buenas Prácticas, Errores Comunes y Comparativa]]
