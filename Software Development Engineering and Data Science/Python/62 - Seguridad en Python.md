# 62 - Seguridad en Python

> Continúa de [[61 - Empaquetado y Distribución|61]] y **cierra** todo el temario de profundización iniciado en [[26 - Modelo de Objetos y Memoria en CPython|26]]. Varios archivos anteriores mencionaron riesgos de seguridad de pasada ([[29 - Introspección y Metaprogramación|29]] con `eval`/`exec`, [[48 - Serialización Avanzada|48]] con `pickle`) — aquí se consolidan en un panorama completo.

## `eval()` y `exec()`: Ejecución de Código Arbitrario

```python
# NUNCA sobre entradas de un usuario o fuente externa
entrada_usuario = "__import__('os').system('rm -rf /')"
resultado = eval(entrada_usuario)  # ejecuta LITERALMENTE cualquier código Python
```

```python
# Alternativa segura para el caso común de "necesito parsear una expresión literal"
import ast

entrada = "[1, 2, 3]"
datos = ast.literal_eval(entrada)   # SOLO permite literales: números, strings, listas, dicts, tuplas
print(datos)  # [1, 2, 3]

# ast.literal_eval rechaza cualquier cosa que no sea un literal simple
# ast.literal_eval("__import__('os').system('ls')")  # ValueError: código no permitido
```

| | `eval()`/`exec()` | `ast.literal_eval()` | `json.loads()` |
|---|---|---|---|
| Ejecuta código arbitrario | Sí (peligroso) | No | No |
| Soporta literales Python (tuplas, sets) | Sí | Sí | No (solo tipos JSON) |
| Uso seguro con entrada externa | No | Sí | Sí |

## Deserialización Insegura: `pickle` y Formatos Similares

Ya cubierto en profundidad en [[48 - Serialización Avanzada|48]]: `pickle.loads()` sobre datos no confiables puede ejecutar código arbitrario mediante `__reduce__`. La regla es la misma que con `eval`: **nunca deserialices pickle de una fuente que no controlas por completo**. Lo mismo aplica, con menor frecuencia de uso, a `yaml.load()` sin el parámetro `Loader=yaml.SafeLoader` (ver [[14 - JSON and YAML|14]]).

```python
import yaml

# INSEGURO: yaml.load sin especificar Loader puede instanciar objetos arbitrarios
# datos = yaml.load(contenido_no_confiable)  # deprecado precisamente por esto

# SEGURO: SafeLoader solo construye tipos básicos de Python
datos = yaml.safe_load(contenido_no_confiable)
```

## Inyección SQL: el Riesgo Clásico

```python
# VULNERABLE: el input del usuario se concatena directamente en el SQL
usuario_id = input("ID de usuario: ")
cursor.execute(f"SELECT * FROM usuarios WHERE id = {usuario_id}")
# Si usuario_id es "1 OR 1=1", devuelve TODOS los usuarios
# Si es "1; DROP TABLE usuarios;--", podría borrar la tabla completa

# SEGURO: consultas parametrizadas -- el driver escapa el valor correctamente
cursor.execute("SELECT * FROM usuarios WHERE id = %s", (usuario_id,))
```

Ver [[24 - Databases|24 - Databases]] para el flujo completo de DB-API 2.0 — **toda** librería de base de datos en Python soporta parámetros (`%s`, `?`, o `:nombre` según el driver); la única razón para concatenar strings manualmente es no conocer esta API.

## Secretos: Nunca Hardcodeados

```python
# MAL: la clave queda en el historial de git para siempre, incluso si se borra después
API_KEY = "sk-abc123456789"

# BIEN: leer desde variables de entorno
import os
API_KEY = os.environ["API_KEY"]   # lanza KeyError explícito si falta, en vez de fallar silenciosamente

# BIEN (para desarrollo local): archivo .env NO versionado + python-dotenv
from dotenv import load_dotenv
load_dotenv()   # carga variables desde un archivo .env local
API_KEY = os.environ["API_KEY"]
```

```text
# .gitignore
.env
*.pem
secrets.json
```

> Si un secreto llega a commitearse a git, **rotarlo inmediatamente** (generar uno nuevo e invalidar el anterior) — borrarlo del código en un commit posterior **no** lo elimina del historial de git, que sigue siendo recuperable.

## El Módulo `secrets`: Aleatoriedad Criptográficamente Segura

```python
import random
import secrets

# random: NO es seguro para tokens, contraseñas o claves -- es predecible con suficientes muestras
token_inseguro = "".join(random.choices("abcdefghij0123456789", k=16))

# secrets: usa la fuente de aleatoriedad del sistema operativo, apto para uso criptográfico
token_seguro = secrets.token_hex(16)          # string hexadecimal
token_url = secrets.token_urlsafe(16)          # seguro para usar en URLs
password_temporal = secrets.token_urlsafe(12)

# Comparación segura contra timing attacks (compara en tiempo constante)
es_valido = secrets.compare_digest(token_recibido, token_esperado)
```

> **Nunca uses `random` para nada relacionado con seguridad** (tokens de sesión, reseteo de contraseña, claves de API) — `random` está diseñado para simulaciones y muestreo estadístico, no para impredecibilidad criptográfica. `random.seed()` incluso permite reproducir la secuencia completa si se conoce la semilla.

## Hashing de Contraseñas: Nunca Texto Plano, Nunca Hash Simple

```python
import hashlib

# MAL: MD5/SHA256 simples son rápidos de calcular -> vulnerables a fuerza bruta con GPUs
password_mal_hasheado = hashlib.sha256(b"mi_contraseña").hexdigest()

# BIEN: usar un algoritmo diseñado específicamente para contraseñas (lento a propósito)
import bcrypt

password = b"mi_contraseña_segura"
hash_guardado = bcrypt.hashpw(password, bcrypt.gensalt())

# Verificación
es_correcta = bcrypt.checkpw(b"intento_del_usuario", hash_guardado)
```

| | MD5/SHA256 directo | bcrypt / argon2 / scrypt |
|---|---|---|
| Diseñado para contraseñas | No (son hashes de propósito general, rápidos) | Sí (deliberadamente lentos y con costo ajustable) |
| Incluye "salt" automáticamente | No (hay que agregarlo manualmente) | Sí, integrado |
| Resistente a ataques de fuerza bruta con GPU/ASIC | No | Sí |

## Deserialización de Datos de Formularios/APIs: Validar Siempre

```python
# Sin validar: cualquier campo extra o tipo incorrecto pasa silenciosamente
def crear_usuario(datos: dict):
    usuario = Usuario(nombre=datos["nombre"], es_admin=datos.get("es_admin", False))
    # Si el cliente envía {"nombre": "x", "es_admin": true} sin autorización, se acepta

# Con Pydantic (ver 40): se valida ESTRUCTURA y TIPOS explícitamente
from pydantic import BaseModel

class CrearUsuarioRequest(BaseModel):
    nombre: str
    email: str
    # es_admin deliberadamente NO incluido -> el cliente no puede inyectarlo

def crear_usuario_seguro(datos: CrearUsuarioRequest):
    usuario = Usuario(nombre=datos.nombre, es_admin=False)  # decisión explícita del servidor
```

Este patrón — nunca confiar en campos que el cliente **podría** enviar de más — es la defensa central contra la llamada _mass assignment vulnerability_, y es exactamente lo que `Python/FastAPI/03 - Request Body y Modelos Pydantic.md` y `Python/FastAPI/09 - Autenticación y Seguridad.md` cubren en el contexto de una API real.

## SSRF y Validación de URLs Externas

```python
import requests
from urllib.parse import urlparse

def descargar_recurso(url_proporcionada_por_usuario):
    parsed = urlparse(url_proporcionada_por_usuario)
    # Validar el host ANTES de hacer la petición, para evitar que el servidor
    # sea usado como proxy hacia direcciones internas (169.254.169.254, localhost, etc.)
    if parsed.hostname in ("localhost", "127.0.0.1") or parsed.hostname.startswith("169.254"):
        raise ValueError("URL no permitida")
    return requests.get(url_proporcionada_por_usuario, timeout=5)
```

Si tu aplicación descarga contenido desde una URL que el usuario controla (miniaturas, webhooks, integraciones), un atacante puede intentar usarla para acceder a metadatos internos de la nube (`169.254.169.254`, común en AWS/GCP) o a servicios internos no expuestos — validar el destino antes de hacer la petición es la mitigación básica.

## Dependencias de Terceros: la Superficie de Ataque Invisible

```bash
pip install pip-audit
pip-audit                 # escanea las dependencias instaladas contra bases de datos de CVEs conocidos
```

```yaml
# GitHub Actions: escaneo automático de dependencias en cada PR
- name: Audit dependencies
  run: |
    pip install pip-audit
    pip-audit
```

Un proyecto con decenas de dependencias transitivas hereda automáticamente cualquier vulnerabilidad de cualquiera de ellas — `pip-audit` (o el Dependabot integrado de GitHub) automatiza detectar cuándo una dependencia instalada tiene una CVE conocida.

## Checklist Final de Seguridad

- ¿Alguna entrada de usuario llega a `eval`/`exec`/`pickle.loads` sin pasar por una fuente confiable?
- ¿Todas las consultas SQL usan parámetros, nunca f-strings/concatenación con datos externos?
- ¿Los secretos viven en variables de entorno, nunca en el código fuente versionado?
- ¿Las contraseñas se hashean con bcrypt/argon2, nunca con MD5/SHA256 directo ni en texto plano?
- ¿Los modelos de entrada (Pydantic/validación) excluyen explícitamente campos sensibles como `es_admin`?
- ¿Las dependencias se auditan periódicamente con `pip-audit`?

## Ver también

- [[29 - Introspección y Metaprogramación]] — el riesgo de `eval`/`exec` en contexto de metaprogramación.
- [[48 - Serialización Avanzada]] — el riesgo detallado de `pickle.loads` sobre datos no confiables.
- [[24 - Databases]] — consultas parametrizadas como prevención de inyección SQL.
- `Python/FastAPI/09 - Autenticación y Seguridad.md` — estos principios aplicados a una API real con OAuth2/JWT.
