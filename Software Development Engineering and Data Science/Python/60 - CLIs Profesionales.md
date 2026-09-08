# 60 - CLIs Profesionales

> Continúa de [[59 - Python Moderno por Versión|59]]. Muchas herramientas internas (scripts de automatización, utilidades de datos, herramientas de DevOps) se exponen mejor como una **interfaz de línea de comandos** bien diseñada que como un script que se edita a mano cada vez que cambian los parámetros.

## El Problema: Scripts con Parámetros "Hardcodeados"

```python
# mal_script.py -- cualquier cambio de comportamiento requiere EDITAR el archivo
ARCHIVO_ENTRADA = "datos.csv"
FORMATO_SALIDA = "json"
VERBOSE = True

def main():
    ...

main()
```

```python
# Con argparse: los parámetros se pasan desde la terminal, sin tocar el código
import argparse

def main():
    parser = argparse.ArgumentParser(description="Convierte archivos entre formatos")
    parser.add_argument("entrada", help="Archivo de entrada")
    parser.add_argument("--formato", choices=["json", "yaml", "csv"], default="json")
    parser.add_argument("--verbose", action="store_true", help="Salida detallada")
    args = parser.parse_args()

    if args.verbose:
        print(f"Procesando {args.entrada} hacia formato {args.formato}")

if __name__ == "__main__":
    main()
```

```bash
python script.py datos.csv --formato yaml --verbose
```

## `argparse`: la Librería Estándar, a Fondo

```python
import argparse

parser = argparse.ArgumentParser(
    prog="miherramienta",
    description="Herramienta de procesamiento de datos",
)

# Argumento posicional (obligatorio, sin prefijo --)
parser.add_argument("archivo", type=str, help="Ruta del archivo a procesar")

# Argumentos opcionales con valor
parser.add_argument("-o", "--output", default="salida.txt", help="Archivo de salida")
parser.add_argument("-n", "--num-lineas", type=int, default=10)

# Flags booleanas (no toman valor, solo presencia/ausencia)
parser.add_argument("-v", "--verbose", action="store_true")

# Opciones restringidas a un conjunto fijo de valores
parser.add_argument("--nivel", choices=["debug", "info", "warning"], default="info")

# Múltiples valores (nargs)
parser.add_argument("--tags", nargs="+", help="Uno o más tags")

args = parser.parse_args()
print(args)  # Namespace(archivo='...', output='salida.txt', ...)
```

### Subcomandos (estilo `git commit`, `docker run`)

```python
import argparse

parser = argparse.ArgumentParser(prog="gestor")
subparsers = parser.add_subparsers(dest="comando", required=True)

parser_crear = subparsers.add_parser("crear", help="Crea un nuevo recurso")
parser_crear.add_argument("nombre")

parser_eliminar = subparsers.add_parser("eliminar", help="Elimina un recurso")
parser_eliminar.add_argument("nombre")
parser_eliminar.add_argument("--forzar", action="store_true")

args = parser.parse_args()

if args.comando == "crear":
    print(f"Creando {args.nombre}")
elif args.comando == "eliminar":
    print(f"Eliminando {args.nombre} (forzado={args.forzar})")
```

```bash
python gestor.py crear mi_recurso
python gestor.py eliminar mi_recurso --forzar
```

## Las Limitaciones de `argparse` en la Práctica

- Verboso para CLIs con muchos comandos y validaciones de tipo complejas.
- No integra automáticamente con [[42 - Type Hints y el Módulo Typing|type hints]] — todo se declara imperativamente con `add_argument`.
- La ayuda generada (`--help`) es funcional pero visualmente básica, sin color ni formato enriquecido.

## Click: Declarativo y Basado en Decoradores

```python
import click

@click.command()
@click.argument("archivo")
@click.option("--formato", type=click.Choice(["json", "yaml", "csv"]), default="json")
@click.option("--verbose", is_flag=True, help="Salida detallada")
def convertir(archivo, formato, verbose):
    """Convierte ARCHIVO al formato especificado."""
    if verbose:
        click.echo(f"Procesando {archivo} hacia {formato}")

if __name__ == "__main__":
    convertir()
```

```python
# Grupos de comandos (subcomandos), más ergonómico que argparse
@click.group()
def cli():
    pass

@cli.command()
@click.argument("nombre")
def crear(nombre):
    click.echo(f"Creando {nombre}")

@cli.command()
@click.argument("nombre")
@click.option("--forzar", is_flag=True)
def eliminar(nombre, forzar):
    click.echo(f"Eliminando {nombre} (forzado={forzar})")

if __name__ == "__main__":
    cli()
```

`click` valida tipos automáticamente, genera mensajes de error consistentes, y su sintaxis basada en decoradores reduce notablemente el boilerplate comparado con `argparse` para CLIs con múltiples subcomandos.

## Typer: Click + Type Hints (la Opción Más Moderna)

`typer` (creado por el mismo autor de FastAPI, ver `Python/FastAPI/`) construye sobre `click`, pero deriva la interfaz de línea de comandos **directamente de las anotaciones de tipo** de una función normal de Python — la misma filosofía que FastAPI aplica a APIs web.

```python
import typer

app = typer.Typer()

@app.command()
def convertir(
    archivo: str,
    formato: str = "json",
    verbose: bool = False,
):
    """Convierte ARCHIVO al formato especificado."""
    if verbose:
        typer.echo(f"Procesando {archivo} hacia {formato}")

@app.command()
def crear(nombre: str):
    typer.echo(f"Creando {nombre}")

if __name__ == "__main__":
    app()
```

```python
# typer también aprovecha typing.Annotated para metadata rica (ayuda, validación)
from typing import Annotated

@app.command()
def saludar(
    nombre: Annotated[str, typer.Argument(help="Nombre a saludar")],
    veces: Annotated[int, typer.Option(help="Cuántas veces repetir")] = 1,
):
    for _ in range(veces):
        typer.echo(f"Hola, {nombre}!")
```

Con `typer`, no se importa ni se llama a nada especial de argparse/click para declarar tipos — la firma de la función **es** la especificación de la CLI, con autocompletado y chequeo de mypy/pyright funcionando de forma nativa (ver [[42 - Type Hints y el Módulo Typing|42]] y [[43 - Análisis Estático con Mypy y Pyright|43]]).

## Comparativa

| | `argparse` | `click` | `typer` |
|---|---|---|---|
| Parte del stdlib | Sí | No | No |
| Estilo | Imperativo (`add_argument`) | Declarativo (decoradores) | Declarativo (type hints) |
| Integración con type hints | Ninguna | Parcial | Nativa y central |
| Subcomandos anidados | Soportado, verboso | Ergonómico | Muy ergonómico |
| Autocompletado de shell (bash/zsh) | Manual | Integrado | Integrado (heredado de click) |
| Curva de aprendizaje | Baja (ya en stdlib) | Baja-media | Baja para quien ya usa type hints/FastAPI |
| Cuándo usarlo | Scripts simples, sin dependencias externas | CLIs medianas/complejas | CLIs nuevas, especialmente si el equipo ya usa FastAPI/Pydantic |

## Buenas Prácticas para CLIs Robustas

```python
import sys

def main():
    try:
        # lógica principal
        pass
    except KeyboardInterrupt:
        print("\nOperación cancelada por el usuario", file=sys.stderr)
        sys.exit(130)   # 130 es el código de salida convencional para Ctrl+C
    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    main()
```

- **Códigos de salida significativos:** `0` para éxito, distinto de cero para error (`sys.exit(1)`) — scripts y pipelines de CI dependen de esto para saber si algo falló.
- **Mensajes de error a `stderr`, no `stdout`:** permite que `stdout` se use limpiamente para el resultado real (útil cuando la salida se redirige con `|` a otro programa).
- **`--help` siempre generado automáticamente** — todas las librerías mencionadas lo hacen por defecto.

## Ver también

- [[42 - Type Hints y el Módulo Typing]] — la base que hace posible el enfoque declarativo de `typer`.
- `Python/FastAPI/` — la misma filosofía de "la firma de la función es la especificación", aplicada a APIs web.
- [[61 - Empaquetado y Distribución]] — cómo convertir un script CLI en un comando instalable (`entry_points`).
- [[45 - Estilo Pythonico y Buenas Prácticas]] — principios de diseño de API aplicables también al diseño de una CLI.
