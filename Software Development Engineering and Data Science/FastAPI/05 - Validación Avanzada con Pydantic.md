---
tags: [fastapi, apis, pydantic, python, cheat-sheet]
---

# 05 — Validación Avanzada con Pydantic

> Continúa de [[04 - Response Models y Serialización]]. Esta nota cubre validación de Pydantic en el contexto específico de request/response de FastAPI — para una referencia exhaustiva de Pydantic como librería independiente (más allá de su uso en APIs), ver la carpeta `Pydantic/` cuando esté desarrollada.

## `field_validator` — validación custom por campo (Pydantic v2)

```python
from pydantic import BaseModel, field_validator

class Usuario(BaseModel):
    nombre: str
    email: str
    edad: int

    @field_validator("email")
    @classmethod
    def email_debe_tener_arroba(cls, valor: str) -> str:
        if "@" not in valor:
            raise ValueError("email debe contener @")
        return valor.lower()   # el validador también puede transformar el valor

    @field_validator("edad")
    @classmethod
    def edad_razonable(cls, valor: int) -> int:
        if not 0 < valor < 120:
            raise ValueError("edad fuera de rango razonable")
        return valor
```

Cuando un `field_validator` lanza `ValueError`, FastAPI lo captura y lo convierte automáticamente en un **422** con el mensaje incluido — el mismo flujo que la validación de tipos básica, solo que con lógica custom.

## `model_validator` — validación que cruza varios campos

```python
from pydantic import BaseModel, model_validator

class RangoFechas(BaseModel):
    fecha_inicio: str
    fecha_fin: str

    @model_validator(mode="after")
    def fin_despues_de_inicio(self):
        if self.fecha_fin < self.fecha_inicio:
            raise ValueError("fecha_fin no puede ser anterior a fecha_inicio")
        return self
```

`field_validator` valida un campo de forma aislada; `model_validator` se usa cuando la regla depende de la relación **entre** varios campos, como aquí.

## Tipos especiales listos para usar

```python
from pydantic import BaseModel, EmailStr, HttpUrl, PositiveInt, PositiveFloat

class Suscripcion(BaseModel):
    email: EmailStr           # requiere: pip install "pydantic[email]"
    sitio_web: HttpUrl
    precio: PositiveFloat
    intentos_max: PositiveInt
```

`EmailStr` valida formato de correo real (no solo "contiene @"), `HttpUrl` valida que sea una URL bien formada y la parsea en un objeto navegable — evitan tener que escribir regex a mano para casos comunes.

## `Enum` y `Literal` — restringir a un conjunto fijo de valores

```python
from enum import Enum
from typing import Literal

class Rol(str, Enum):
    admin = "admin"
    editor = "editor"
    lector = "lector"

class Usuario(BaseModel):
    rol: Rol                                   # opción 1: Enum, reutilizable en varios modelos
    estado: Literal["activo", "inactivo", "suspendido"]  # opción 2: Literal, más rápido para un solo uso
```

`Enum` es preferible cuando el mismo conjunto de valores se reutiliza en múltiples modelos o endpoints (y aparece como dropdown con nombre propio en Swagger); `Literal` es más directo para restricciones puntuales de un solo campo.

## `Field()` — restricciones declarativas sin lógica custom

```python
from pydantic import BaseModel, Field

class Producto(BaseModel):
    nombre: str = Field(min_length=1, max_length=100)
    precio: float = Field(gt=0, description="Precio en USD, debe ser positivo")
    sku: str = Field(pattern=r"^[A-Z]{3}-\d{4}$")   # regex
    cantidad: int = Field(ge=0, le=10_000, default=0)
```

Para restricciones simples (rangos numéricos, longitud, patrones), `Field()` es preferible a un `field_validator` completo — es más legible y se refleja directamente en el esquema de OpenAPI (Swagger muestra el rango permitido sin necesitar leer el código).

## Validación de listas y colecciones

```python
class Pedido(BaseModel):
    items: list[str] = Field(min_length=1, max_length=50)   # entre 1 y 50 elementos
    cantidades: list[PositiveInt]                              # cada elemento debe ser positivo

    @field_validator("items")
    @classmethod
    def sin_duplicados(cls, valor: list[str]) -> list[str]:
        if len(valor) != len(set(valor)):
            raise ValueError("no se permiten items duplicados")
        return valor
```

## Mensajes de error personalizados en la respuesta

Por defecto, un error de validación se ve así en la respuesta 422:

```json
{
  "detail": [
    {
      "type": "value_error",
      "loc": ["body", "edad"],
      "msg": "Value error, edad fuera de rango razonable",
      "input": 200
    }
  ]
}
```

`loc` indica exactamente dónde ocurrió el error (`body -> edad`), útil para que el cliente de la API resalte el campo correcto en su propio formulario. Para personalizar completamente este formato de respuesta, ver el manejador de `RequestValidationError` en [[07 - Manejo de Errores y Excepciones]].

## Ver también

- [[03 - Request Body y Modelos Pydantic]]
- [[04 - Response Models y Serialización]]
- [[07 - Manejo de Errores y Excepciones]]
