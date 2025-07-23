---
Fecha de creación: 2025-07-21T18:07:00
Materia:
  - Programación II
Fecha de clase: 2025-07-21
---

# Herencia

La herencia es uno de los pilares fundamentales de la programación orientada a objetos (POO) en Java. Es un mecanismo que permite crear nuevas clases basadas en clases existentes, promoviendo la reutilización de código y estableciendo relaciones jerárquicas entre clases.

## Conceptos Clave de Herencia

1. Clase Padre (Superclase): La clase cuyas características son heredadas.
2. Clase Hija (Subclase): La clase que hereda las características de otra clase.
3. `extends`: Palabra clave que se usa para indicar que una clase hereda de otra.

```java
// Clase Padre (SuperClase)
class Vehiculo {
    void mover() {
        System.out.println("El vehículo se está moviendo");
    }
}

// Clase Hija (SubClase)
class Coche extends Vehiculo {
    void acelerar() {
        System.out.println("El coche está acelerando");
    }
}

// Clase Main
public class Main {
    public static void main(String[] args) {
        Coche miCoche = new Coche();

        miCoche.mover();    // Método heredado de Vehiculo
        miCoche.acelerar(); // Método propio de Coche
    }
}
```

## Herencia Simple y Multinivel

- **Herencia simple:** Una clase solo puede heredar de una superclase (Java no soporta herencia múltiple de clases).
- **Herencia multinivel:** Una clase puede heredar de otra que a su vez hereda de una tercera.
- **Jerarquía de herencia:** Una superclase puede tener múltiples subclases.

## `super`

El uso de `super` en Java es fundamental en la herencia, permitiendo acceder a miembros (métodos y atributos) y constructores de la clase padre desde la clase hija.

### Casos principales de uso de `super`

- Llamar al constructor de la clase padre: `super(...)`
- Acceder a métodos de la clase padre: `super.metodo()`
- Acceder a atributos de la clase padre: `super.atributo`

```java
// Clase base Vehiculo
class Vehiculo {
    String marca;
    int año;

    // Constructor de Vehiculo
    Vehiculo(String marca, int año) {
        this.marca = marca;
        this.año = año;
    }

    // Método que será heredado y extendido
    void mostrarInfo() {
        System.out.println("Vehículo marca " + marca + ", año " + año);
    }
}

// Subclase Automovil
class Automovil extends Vehiculo {
    String modelo;
    int puertas;

    // Constructor de Automovil
    Automovil(String marca, int año, String modelo, int puertas) {
        super(marca, año);  // Llama al constructor de Vehículo
        this.modelo = modelo;
        this.puertas = puertas;
    }

    // Sobrescritura del método mostrarInfo()
    @Override
    void mostrarInfo() {
        super.mostrarInfo();  // Llama al método de la clase padre
        System.out.println("Modelo: " + modelo + ", Puertas: " + puertas);
    }
}

public class EjemploVehiculo {
    public static void main(String[] args) {
        // Creación de objeto Automóvil
        Automovil miAuto = new Automovil("Toyota", 2022, "Corolla", 4);

        // Llamada al método mostrarInfo() que combina funcionalidades
        miAuto.mostrarInfo();
        /* Output:
           Vehículo marca Toyota, año 2022
           Modelo: Corolla, Puertas: 4
        */
    }
}
```

# Clases Abstractas e Interfaces

## Clases Abstractas

Una clase abstracta es una clase que no se puede instanciar directamente y que puede contener **métodos abstractos** (sin implementación) y **métodos concretos** (con implementación). Se usa como base para otras clases.

### Características

- Puede tener métodos abstractos y concretos.
- Puede tener variables de instancia.
- Las subclases deben implementar todos los métodos abstractos.

## Interfaces

Una interfaz es una colección de métodos abstractos y constantes (`static final`). A partir de Java 8 pueden tener métodos `default` y `static` con implementación.

### Características

- Solo puede tener métodos abstractos (antes de Java 8).
- Puede tener constantes (`public static final`).
- Una clase puede implementar múltiples interfaces.
- Representa capacidades o comportamientos que una clase puede tener.

## `implements`

En Java, `implements` es una palabra clave que se usa para indicar que una clase está implementando una interfaz.

### ¿Qué significa esto?

- Una interfaz es como un contrato que define qué métodos debe tener una clase, pero no cómo se implementan.
- Cuando una clase implementa una interfaz, está obligada a proporcionar implementaciones concretas para todos los métodos abstractos de esa interfaz.

```java
// Interfaz (define qué debe hacer, no cómo)
interface Encendible {
    void encender();  // Método abstracto (sin implementación)
    void apagar();    // Método abstracto (sin implementación)
}

class Foco implements Encendible {
    // Implementamos los métodos de la interfaz
    @Override
    public void encender() {
        System.out.println("Foco encendido");
    }

    @Override
    public void apagar() {
        System.out.println("Foco apagado");
    }
}

public class Main {
    public static void main(String[] args) {
        Encendible miFoco = new Foco();

        miFoco.encender();
        miFoco.apagar();
     }
}
```

### ¿Qué demuestra este ejemplo?

- **La interfaz `Encendible`**

Solo declara qué debe hacer (`encender()`, `apagar()`), no cómo.
Es como un "contrato" que las clases deben cumplir.

- **La clase `Bombilla`**

Implementa la interfaz (usando `implements`).
Obligatoriamente define cómo funcionan `encender()` y `apagar()`.

- **Uso**

Puedes tratar `Foco` como un `Encendible` (útil para polimorfismo).
