---
tags: [pytest, python, testing, best-practices, cheat-sheet]
---

# 17 — Buenas Prácticas, Errores Comunes y Comparativa

> Cierra la serie iniciada en [[01 - Introducción y Arquitectura]].

## Tests interdependientes (el error más insidioso)

```python
# MAL — test_02 asume que test_01 ya se ejecutó y dejó cierto estado
usuarios_creados = []

def test_01_crear_usuario():
    usuarios_creados.append(crear_usuario("Ana"))

def test_02_contar_usuarios():
    assert len(usuarios_creados) == 1     # falla si se ejecuta AISLADO, o en otro orden
```

Cada test debe poder ejecutarse **de forma aislada e independiente**, en cualquier orden — depender de efectos secundarios de otro test es frágil (se rompe con `pytest-xdist`, con `pytest-randomly`, o simplemente al ejecutar un solo test con `pytest test_02_contar_usuarios`). La solución es usar fixtures para el estado que cada test necesita, en vez de variables globales compartidas — ver [[03 - Fixtures - Fundamentos]].

## Sobre-mockeo: tests que no prueban nada real

```python
# Este test pasa incluso si calcular_total() está completamente roto
def test_calcular_total(mocker):
    mocker.patch("mi_modulo.calcular_total", return_value=100)
    assert mi_modulo.calcular_total() == 100     # solo verifica que el mock devuelve lo que se le dijo que devolviera
```

Mockear la función que en teoría se está probando (en vez de sus dependencias externas) hace que el test sea circular — verifica que el mock funciona, no que el código real funciona. Ver la guía completa de cuándo mockear en [[09 - Mocking y Monkeypatching#Cuándo mockear y cuándo no|Mocking]].

## Fixtures con scope demasiado amplio y estado compartido mutable

```python
@pytest.fixture(scope="session")
def lista_compartida():
    return []          # PELIGRO: la MISMA lista se comparte entre TODOS los tests de la sesión

def test_agregar_item(lista_compartida):
    lista_compartida.append(1)
    assert len(lista_compartida) == 1     # pasa solo si es el PRIMER test en usar la fixture
```

Un scope amplio (`session`, `module`) es apropiado para recursos costosos de crear (conexiones, servidores), pero **no** para objetos mutables que un test modifica y otro luego lee — eso reintroduce el problema de tests interdependientes a través de la fixture compartida. Ver el detalle de scopes en [[03 - Fixtures - Fundamentos#Scope — cuántas veces se ejecuta una fixture|Fixtures]].

## Asserts múltiples sin contexto de cuál falló

```python
# Si el segundo assert falla, nunca se sabe si el primero habría pasado
def test_usuario_completo():
    usuario = crear_usuario()
    assert usuario.nombre == "Ana"
    assert usuario.edad == 28
    assert usuario.activo is True
```

No es necesariamente incorrecto (pytest muestra claramente cuál línea falló), pero cuando los tres `assert` verifican conceptos genuinamente independientes, es preferible dividir en tres tests separados — así un fallo no oculta si los otros dos también habrían fallado, y los tres se ejecutan y reportan independientemente.

## Nombres de test que no explican qué se está verificando

```python
# MAL — no comunica qué comportamiento se espera
def test_1():
    assert calcular_descuento(100, 0.1) == 90

# BIEN — el nombre ES la documentación de la propiedad verificada
def test_descuento_del_10_por_ciento_reduce_precio_correctamente():
    assert calcular_descuento(100, 0.1) == 90
```

Un nombre de test descriptivo sirve como documentación viva — cuando falla en CI, el nombre solo ya comunica qué se rompió, sin necesitar abrir el código del test para entenderlo.

## Checklist antes de dar una suite de tests por "lista"

- [ ] ¿Cada test puede correr de forma aislada, en cualquier orden (`pytest-randomly` no revela dependencias ocultas)?
- [ ] ¿Los mocks cubren dependencias externas reales, no la lógica que se está probando?
- [ ] ¿Los nombres de test describen el comportamiento verificado, no solo un número secuencial?
- [ ] ¿La cobertura mide algo significativo, no solo líneas ejecutadas sin aserciones reales? Ver [[12 - Cobertura de Código]].
- [ ] ¿Los tests lentos/de integración están marcados y separados de los rápidos en CI? Ver [[16 - Integración con CI-CD]].

## Comparativa: pytest vs unittest vs nose vs doctest

| | pytest | unittest | nose(2) | doctest |
|---|---|---|---|---|
| Incluido en la librería estándar | No | **Sí** | No | **Sí** |
| Sintaxis de aserción | `assert` plano | Métodos `self.assertX()` | `assert` plano (heredado de pytest-style) | Comparación de salida de `>>>` literal |
| Requiere clases | No (funciones sueltas o clases opcionales) | Sí, heredar de `TestCase` | No | No |
| Sistema de fixtures | **Muy potente** (scopes, composición, parametrización) | `setUp`/`tearDown` básico | Similar a unittest | No aplica |
| Parametrización nativa | **Sí**, `@pytest.mark.parametrize` | No nativa (requiere librerías extra o loops manuales) | No nativa | No aplica |
| Ecosistema de plugins | **Enorme** y activo | Ninguno propio | Discontinuado, ya no se mantiene activamente | No aplica |
| Estado del proyecto | Activo, estándar de facto | Activo (parte de la stdlib) | **Discontinuado** — sucesor espiritual es pytest | Activo, pero de uso muy acotado |
| Mejor para | La gran mayoría de proyectos nuevos | Librerías que deben minimizar dependencias externas, o equipos ya invertidos en unittest | Ya no recomendado para proyectos nuevos | Ejemplos de documentación que deben mantenerse correctos |

**Regla práctica:** pytest es la elección por default para cualquier proyecto nuevo — su sintaxis más simple, sistema de fixtures superior, y ecosistema de plugins lo hacen strictamente más capaz que `unittest` para la gran mayoría de casos, sin perder la capacidad de ejecutar tests escritos en estilo `unittest` (pytest los descubre y corre igual, ver [[Testing#Unittest (PyUnit) El Framework Estándar|Python/Testing]]). `unittest` sigue siendo razonable cuando el proyecto es una librería que busca minimizar dependencias externas (al venir en la librería estándar). `nose`/`nose2` no se recomienda para proyectos nuevos. `doctest` es un complemento, no un sustituto, útil específicamente para mantener ejemplos de documentación verificados automáticamente.

## Ver también

- [[01 - Introducción y Arquitectura]]
- [[03 - Fixtures - Fundamentos]]
- [[09 - Mocking y Monkeypatching]]
- [[Testing#¿Cuál Usar?|Python/Testing]]
- [[Machine Learning/25-Testing-Avanzado|Machine Learning/25 - Testing Avanzado]]
