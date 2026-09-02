---
tags: [pytest, python, testing, database, cheat-sheet]
---

# 14 — Testing de Bases de Datos e Infraestructura

> Continúa de [[13 - Testing Asíncrono]].

## El dilema: mockear la BD vs usar una real

Mockear completamente una base de datos (ver [[09 - Mocking y Monkeypatching]]) hace los tests rápidos, pero no detecta errores reales de SQL, constraints, o comportamiento específico del motor — usar una base de datos **real** (aunque sea de prueba) detecta esos problemas, a costa de tests más lentos y con más infraestructura necesaria. La estrategia común es una combinación: mocks para tests unitarios rápidos, BD real para un conjunto más pequeño de tests de integración.

## Fixture de sesión de base de datos con rollback transaccional

```python
@pytest.fixture(scope="session")
def motor_db():
    motor = create_engine("postgresql://test_user:test_pass@localhost/test_db")
    yield motor
    motor.dispose()

@pytest.fixture
def sesion_db(motor_db):
    conexion = motor_db.connect()
    transaccion = conexion.begin()
    Sesion = sessionmaker(bind=conexion)
    sesion = Sesion()

    yield sesion

    sesion.close()
    transaccion.rollback()       # DESHACE todos los cambios del test — la BD queda exactamente como estaba
    conexion.close()
```

Este patrón (fixture de conexión con `scope="session"` + fixture de sesión con `scope="function"` que envuelve cada test en una transacción con **rollback** al final) es el estándar para testing con bases de datos relacionales: cada test opera contra una BD real, pero ningún cambio persiste entre tests — evita tanto la lentitud de recrear la BD completa en cada test como la contaminación de estado entre tests.

```python
def test_crear_usuario(sesion_db):
    usuario = Usuario(nombre="Ana")
    sesion_db.add(usuario)
    sesion_db.commit()

    resultado = sesion_db.query(Usuario).filter_by(nombre="Ana").first()
    assert resultado is not None
    # al terminar el test, el rollback de la fixture deshace esta inserción automáticamente
```

## SQLite en memoria — para tests rápidos sin infraestructura externa

```python
@pytest.fixture
def motor_db_memoria():
    motor = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(motor)
    yield motor
    Base.metadata.drop_all(motor)
```

Una base de datos SQLite en memoria arranca instantáneamente y no requiere ningún servicio externo corriendo — buena opción para tests rápidos de la capa de acceso a datos, aunque **no** detecta diferencias de comportamiento específicas del motor real de producción (ej. PostgreSQL) si el proyecto depende de features específicas de ese motor.

## `testcontainers` — infraestructura real, efímera, por test

```python
from testcontainers.postgres import PostgresContainer

@pytest.fixture(scope="session")
def postgres_container():
    with PostgresContainer("postgres:16") as postgres:
        yield postgres.get_connection_url()
```

`testcontainers` levanta un contenedor Docker real (Postgres, Redis, Kafka, lo que sea necesario) automáticamente al inicio de la suite y lo destruye al final — da la fidelidad de probar contra el motor real de producción sin necesitar una instancia compartida y persistente de infraestructura de pruebas, ni el riesgo de que un test contamine el estado de otro proyecto que use el mismo servicio compartido.

## Fixtures de infraestructura no-BD: archivos temporales, caché, colas

```python
def test_procesar_archivo(tmp_path):          # 'tmp_path' es una fixture incluida en pytest — un directorio temporal único por test
    archivo = tmp_path / "datos.csv"
    archivo.write_text("a,b\n1,2\n")

    resultado = procesar_csv(archivo)
    assert resultado == [{"a": "1", "b": "2"}]
```

`tmp_path` (y su variante `tmp_path_factory` con scope más amplio) es una fixture nativa de pytest para trabajar con archivos temporales sin ensuciar el sistema de archivos real ni tener que limpiar manualmente — se elimina automáticamente después de cierto número de ejecuciones.

## Ver también

- [[13 - Testing Asíncrono]]
- [[09 - Mocking y Monkeypatching]]
- [[Docker/Docker for Data Science|Docker]]
