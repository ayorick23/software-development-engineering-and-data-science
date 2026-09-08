# 41 - Patrones de Diseño Pythonicos

> Continúa de [[40 - Dataclasses, NamedTuple y Comparativa con Attrs-Pydantic|40]] y cierra el bloque de POO avanzada. Los **patrones de diseño** clásicos (del libro _Design Patterns_ de la "Gang of Four", pensado originalmente para C++/Java) siguen siendo útiles en Python, pero varios se simplifican radicalmente gracias a que funciones son de primera clase y el lenguaje es dinámico.

## Singleton

Garantiza que una clase tenga **una única instancia** en todo el programa.

```python
class Singleton:
    _instancia = None

    def __new__(cls, *args, **kwargs):
        if cls._instancia is None:
            cls._instancia = super().__new__(cls)
        return cls._instancia

class Configuracion(Singleton):
    def __init__(self):
        self.valores = {}

c1 = Configuracion()
c2 = Configuracion()
print(c1 is c2)  # True -> misma instancia
```

> **Alternativa más pythónica:** en la mayoría de los casos, un **módulo** ya es un singleton natural (Python solo importa un módulo una vez, cacheándolo en `sys.modules`). En vez de una clase Singleton, simplemente define variables y funciones a nivel de módulo:

```python
# config.py
valores = {}

def obtener(clave):
    return valores.get(clave)

# en cualquier otro archivo: import config -> siempre el MISMO módulo, ya cacheado
```

## Factory (Fábrica)

Centraliza la lógica de creación de objetos, desacoplando al código cliente de las clases concretas.

```python
from abc import ABC, abstractmethod

class Notificador(ABC):
    @abstractmethod
    def enviar(self, mensaje: str): ...

class NotificadorEmail(Notificador):
    def enviar(self, mensaje):
        print(f"Email: {mensaje}")

class NotificadorSMS(Notificador):
    def enviar(self, mensaje):
        print(f"SMS: {mensaje}")

def crear_notificador(tipo: str) -> Notificador:
    """Factory function: decide qué clase concreta instanciar."""
    notificadores = {"email": NotificadorEmail, "sms": NotificadorSMS}
    clase = notificadores.get(tipo)
    if clase is None:
        raise ValueError(f"Tipo de notificador desconocido: {tipo}")
    return clase()

notificador = crear_notificador("email")
notificador.enviar("Hola!")
```

En Python, una **función factory simple** (como `crear_notificador` arriba) suele ser suficiente — no siempre se necesita una jerarquía completa de clases `Factory`/`AbstractFactory` como en Java.

## Strategy (Estrategia)

Permite intercambiar el algoritmo usado por un objeto en tiempo de ejecución.

```python
# Versión clásica orientada a objetos
class EstrategiaDescuento(ABC):
    @abstractmethod
    def calcular(self, total: float) -> float: ...

class SinDescuento(EstrategiaDescuento):
    def calcular(self, total):
        return total

class DescuentoPorcentaje(EstrategiaDescuento):
    def __init__(self, porcentaje):
        self.porcentaje = porcentaje

    def calcular(self, total):
        return total * (1 - self.porcentaje / 100)

class Carrito:
    def __init__(self, estrategia: EstrategiaDescuento):
        self.estrategia = estrategia

    def total_final(self, subtotal):
        return self.estrategia.calcular(subtotal)

carrito = Carrito(DescuentoPorcentaje(20))
print(carrito.total_final(100))  # 80.0
```

```python
# Versión "pythónica": una función simple hace lo mismo, sin jerarquía de clases
def sin_descuento(total): return total
def descuento_porcentaje(porcentaje):
    return lambda total: total * (1 - porcentaje / 100)

estrategia = descuento_porcentaje(20)
print(estrategia(100))  # 80.0
```

Dado que las funciones son de primera clase (ver [[31 - Funciones de Alto Orden y Programación Funcional|31]]), Strategy en Python muchas veces se reduce a "pasar una función como parámetro" — sin necesidad de una interfaz `EstrategiaDescuento` completa, salvo que el algoritmo necesite mantener estado propio complejo.

## Observer (Observador)

Permite que varios objetos ("observadores") sean notificados automáticamente cuando cambia el estado de otro objeto ("sujeto").

```python
class Sujeto:
    def __init__(self):
        self._observadores = []

    def suscribir(self, observador):
        self._observadores.append(observador)

    def notificar(self, evento):
        for observador in self._observadores:
            observador(evento)   # los observadores son simplemente funciones invocables

class EstacionMeteorologica(Sujeto):
    def actualizar_temperatura(self, grados):
        self.notificar({"temperatura": grados})

def log_consola(evento):
    print(f"[LOG] Nuevo evento: {evento}")

def alerta_frio(evento):
    if evento["temperatura"] < 0:
        print("¡Alerta! Temperatura bajo cero")

estacion = EstacionMeteorologica()
estacion.suscribir(log_consola)
estacion.suscribir(alerta_frio)
estacion.actualizar_temperatura(-5)
```

Este es, conceptualmente, el mismo patrón detrás de los **event listeners** de frameworks web y de librerías de UI — y la base de sistemas de señales como Django Signals.

## Decorator Pattern vs `@decorador` de Python

Es fácil confundir el **patrón de diseño Decorator** (GoF) con la sintaxis `@decorador` de Python — están relacionados pero no son lo mismo.

```python
# Patrón Decorator (GoF): envolver un objeto con la MISMA interfaz, añadiendo comportamiento
class Cafe:
    def costo(self):
        return 2.0

class ConLeche:
    def __init__(self, cafe):
        self._cafe = cafe

    def costo(self):
        return self._cafe.costo() + 0.5

pedido = ConLeche(Cafe())
print(pedido.costo())  # 2.5

# El decorador @sintáctico de Python (ver 16 - Decorators) resuelve el MISMO problema,
# pero para FUNCIONES, con azúcar sintáctico integrado en el lenguaje:
def con_leche(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs) + 0.5
    return wrapper

@con_leche
def costo_cafe():
    return 2.0

print(costo_cafe())  # 2.5
```

| | Patrón Decorator (GoF) | `@decorador` de Python |
|---|---|---|
| Aplica a | Objetos (composición de interfaces) | Funciones/clases |
| Requiere una interfaz común | Sí (ambos implementan `costo()`) | No |
| Sintaxis dedicada del lenguaje | No | Sí (`@nombre`) |
| Casos de uso típicos | Envolver objetos con capas de comportamiento | Logging, cacheo, validación, registro de rutas |

## Context Manager como Patrón de Diseño

El protocolo `with` (ver [[15 - Context Manager|15]] y [[50 - Context Managers Avanzados|50]]) es, en esencia, el patrón **RAII** (_Resource Acquisition Is Initialization_) de C++ adaptado a Python — otro ejemplo de un patrón clásico convertido en sintaxis nativa del lenguaje en vez de requerir una implementación manual repetida en cada clase.

## Cuándo NO Usar un Patrón de Diseño

Python favorece la simplicidad (`import this` — el Zen de Python, ver [[45 - Estilo Pythonico y Buenas Prácticas|45]]). Antes de implementar un patrón formal, pregúntate:

- ¿Una función simple resolvería esto igual de bien? (Strategy, Factory simples)
- ¿Ya existe un mecanismo del lenguaje que cubre esto? (Singleton → módulo; Decorator → `@decorador`; RAII → `with`)
- ¿Estoy agregando una capa de abstracción porque el problema la necesita, o porque "así se hace en Java"?

## Ver también

- [[16 - Decorators]] — la sintaxis `@decorador` que Python ofrece en vez del patrón Decorator clásico.
- [[31 - Funciones de Alto Orden y Programación Funcional]] — por qué Strategy/Factory se simplifican con funciones de primera clase.
- [[38 - Metaclases y type()]] — Singleton implementado como metaclase (una alternativa más a `__new__`).
- [[45 - Estilo Pythonico y Buenas Prácticas]] — el principio de simplicidad que guía cuándo (no) usar estos patrones.
