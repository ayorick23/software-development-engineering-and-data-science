---
Fecha de creación: 2025-08-25T18:06:00
Materia:
  - Programación II
Fecha de clase: 2025-08-25
---
# Concurrencia en Java

La **concurrencia** es utilizada por las principales empresas de desarrollo de Java y se refiere a la capacidad de un programa para ejecutar múltiples tareas simultáneamente. Permite el uso eficiente de los recursos del sistema y puede mejorar el rendimiento general y la capacidad de respuesta de la aplicación.

Los conceptos, clases e interfaces de concurrencia de Java utilizados para multihilo, como `Thread`, `Runnable`, `Callable`, `Future`, `ExecutorService` y las clases de `java.util.concurrent`, forman parte de las bibliotecas estándar de Java, por lo que no debería haber demasiadas diferencias entre los distintos frameworks de Java.
## Multiproceso en Java

El subproceso múltiple en Java es una característica que permite que un programa ejecute varios subprocesos al mismo tiempo, para que las tareas puedan trabajar en paralelo y utilizar la CPU de manera más eficiente. Un hilo es simplemente una unidad más pequeña y liviana de un programa que se ejecuta de forma independiente pero dentro del mismo proceso.

## Heredando de la clase Thread

Una forma de crear un hilo es heredándolo de la clase del hilo. Luego, solo hay que sobrescribir el método ``run()`` del objeto del hilo. Este método se invocará al iniciar el hilo.

## ``Thread`` vs ``Runnable``

### Clase ``Thread``

Representa un hilo de ejecución independiente. Cada objeto Thread tiene su propio stack de ejecución y puede ejecutarse concurrentemente con otros hilos.

### Interfaz ``Runnable``

Es una interfaz funcional que define un único método `run()`. Representa una tarea que puede ser ejecutada por un hilo.

```java
public class MiThread extends Thread {  
    public void run() {  
        // Lógica del hilo  
    }  
}  
  
public class MiRunnable implements Runnable {  
    public void run() {  
        // Lógica ejecutable  
    }  
}
```

## Implementación de ``Runnable``

### Ventajas de ``Runnable``

Al implementar ``Runnable``, separamos la tarea del mecanismo de ejecución, permitiendo mayor flexibilidad y evitando problemas de herencia múltiple.

```java
// Implementación de Runnable  
public class TareaCalculo implements Runnable {  
    private int numero;  
  
    public TareaCalculo(int numero) {  
        this.numero = numero;  
    }  
  
    public void run() {  
        // Realizar cálculo  
        System.out.println(Thread.currentThread().getName() + ": " + (numero * 2));  
    }  
}
```

## ``ExecutorService``

### Gestión de Hilos

``ExecutorService`` proporciona un framework para gestionar hilos de manera eficiente, controlando la creación, ciclo de vida y asignación de recursos.

```java
// Crear un ExecutorService con pool de hilos  
ExecutorService executor = Executors.newFixedThreadPool(5);  
  
// Ejecutar tareas  
for (int i = 0; i < 10; i++) {  
    Runnable tarea = new TareaCalculo(i);  
    executor.execute(tarea);  
}  
  
// Finalizar ExecutorService  
executor.shutdown();
```

## ``Callable`` y ``Future``

### Tareas con Retorno

``Callable`` es similar a ``Runnable`` pero puede devolver un resultado y lanzar excepciones. ``Future`` representa el resultado de una computación asíncrona.

```java
// Implementación de Callable  
public class TareaConResultado implements Callable<Integer> {  
    public Integer call() throws Exception {  
        // Realizar cálculo y retornar resultado  
        return 42;  
    }  
}  
  
// Uso con ExecutorService  
Future<Integer> futuro = executor.submit(new TareaConResultado());  
Integer resultado = futuro.get(); // Bloquea hasta obtener el resultado
```

## Sincronización con ``Synchronized``

### Control de Acceso

La palabra clave ``synchronized`` se utiliza para controlar el acceso a recursos compartidos entre múltiples hilos, previniendo condiciones de carrera.

```java
// Método sincronizado  
public class Contador {  
    private int cuenta = 0;  
  
    public synchronized void incrementar() {  
        cuenta++;  
    }  
  
    public synchronized int getCuenta() {  
        return cuenta;  
    }  
}  
  
// Bloque sincronizado  
public void realizarOperacion() {  
    synchronized(this) {  
        // Código crítico  
    }  
}
```

## ``Thread`` vs ``Runnable``

| Aspecto        | Thread                   | Runnable                    |
| -------------- | ------------------------ | --------------------------- |
| Herencia       | Usa herencia (`extends`) | Usa interfaz (`implements`) |
| Flexibilidad   | Menos flexible           | Más flexible                |
| Reutilización  | Limitada                 | Alta                        |
| Acoplamiento   | Alto acoplamiento        | Bajo acoplamiento           |
| Uso de memoria | Mayor overhead           | Menor overhead              |