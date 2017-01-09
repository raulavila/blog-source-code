---
layout: post
title: Multithreading para dummies (3)
permalink: 2015/06/multithreading-3/
tags:
- Java
- desarrollo
- multithreading
comments: true
---

Seguimos con la serie de Multithreading. Nuestro juego de Ping Pong [lo dejamos](/2015/06/multithreading-2/) en un punto bastante óptimo, con los threads (jugadores) quedando bloqueados a la espera de su turno, y con la posibilidad de escalar al número de jugadores que quisiéramos. Sin embargo, había algo que no era demasiado elegante, y es que la clase `Game` se encargaba enteramente de la creación, inicio y finalización de threads, así como de sincronizarse con estos para no terminar antes de que los diferentes threads lo hubieran hecho:

<!--break-->

{% highlight java %}
//...
Thread thread2 = new Thread(player2);
thread2.start();
Thread thread1 = new Thread(player1);
thread1.start();

//Let the players play!
try {
   Thread.sleep(2);
} catch (InterruptedException e) {
   e.printStackTrace();
}

//Tell the players to stop
thread1.interrupt();
thread2.interrupt();

//Wait until players finish
try {
   thread1.join();
   thread2.join();
} catch (InterruptedException e) {
   e.printStackTrace();
}
//...
{% endhighlight %}

#### Gestión de threads / Thread pools

Además de engorroso, no encapsular la gestión de threads convenientemente nos lleva a código poco cohesionado, ya que estamos ligando la lógica del juego en sí a la gestión de la concurrencia. Como añadido, crear theads es algo costoso a nivel de rendimiento, y en aplicaciones más complejas conlleva una carga importante en el rendimiento final de nuestras aplicaciones.

La API Concurrency de Java exporta una serie de clases e interfaces que nos permiten precisamente encapsular la gestión de hilos con gran flexibilidad, el `Executor` framework. Sus tres elementos principales son:

* `Executor`: es una interfaz de un sólo método, `execute(Runnable)`. La idea con este framework es que ahora manejamos tareas (tasks) en lugar de hilos, por lo que le estamos pidiendo a la instancia de `Executor` que por favor ejecute la tarea (instancia de `Runnable`) cuando le sea posible
* `ExecutorService`: interfaz que extiende `Executor` y que publica una serie de métodos más avanzados, para controlar mejor el ciclo completo del trabajo a realizar (`shutdown`, `awaitTermination`), o para ejecutar tareas de tipo `Callable`, que a grandes rasgos son `Runnable` que devuelven un valor ([más información aquí](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Callable.html)). En la [documentación completa](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ExecutorService.html) quedan bastante claras las posibilidades de esta interfaz
* `Executors`: los dos anteriores componentes son interfaces, de las que nosotros podemos crear implementaciones si así lo deseamos. Sin embargo, la mayoría de casos de uso están implementados en el JDK, para utilizar estas implementaciones debemos solicitar una instancia utilizando los métodos factory estáticos que expone esta clase

En general se utiliza el nombre de Thread Pool para refererise las implementaciones de `Executor`/`ExecutorService` que utilicemos para gestionar nuestros threads. Los tipos más comunes que podemos obtener mediante la factoría `Executors` son:

* Single Thread Executor (`newSingleThreadExecutor`): contiene un solo thread que se encarga de ejecutar tareas. No es muy utilizado
* Fixed Thread Pool (`newFixedThreadPool`): mantiene un número constante de threads "vivos", esperando recibir tareas para ejecutar
* Cached Thread Pool (`newCachedThreadPool`): mantiene un pool de threads que puede crecer o decrecer según demanda
* Scheduled Thread Pool (`newScheduledThreadPool`): se utiliza para programar la ejecución de tareas. Devuelve una instancia de [ScheduledExecutorService](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/ScheduledExecutorService.html), ya que `ExecutorService` no expone métodos adecuados para programar tareas futuras, tan solo para ejecutarlas tan pronto como sea posible

### Ping Pong, Versión 5: Pool de threads

Sin necesidad de realizar modificaciones en la clase `Player` podemos adaptar nuestra clase `Game` para que utilice un pool de threads en lugar de encargarse ella de la engorrosa tarea de crear, arrancar y parar hilos. Veamos cómo quedaría:

{% highlight java %}
public class Game {

    public static void main(String[] args) {
        Lock lock = new ReentrantLock();

        Player player1 = new Player("ping", lock);
        Player player2 = new Player("pong", lock);

        player1.setNextPlayer(player2);
        player2.setNextPlayer(player1);

        System.out.println("Game starting...!");

        player1.setPlay(true);

        ExecutorService executor = Executors.newFixedThreadPool(2);

        executor.execute(player1);
        executor.execute(player2);

        sleep(2);

        executor.shutdownNow();

        System.out.println("Game finished!");
    }

    public static void sleep(long ms) {
        try {
            Thread.sleep(ms);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

Utilizamos un pool de dos threads (uno por jugador), y le enviamos las tareas a ejecutar. Nos dormimos 2ms para dejarles pelotear un poco, e invocamos a `shutdownNow()`, que viene a ser el equivalente a interrumpir los threads según hacíamos en las anteriores versiones, pero encapsulado en el pool. Es necesario invocar `shutdownNow` en lugar de `shutdown`, ya que este último deja terminar las tareas en ejecución y devuelve la lista de tareas pendientes. Cómo nuestros jugadores juegan infinitamente hasta que son interrumpidos, si intentamos terminar con `shutdown` la aplicación no acabaría nunca.

Bien, si probamos varias veces la aplicación, veremos que muchas veces funciona como es debido, mientras que otras la salida se presenta tal que así:

{% highlight text %}
Game starting...!
ping
pong
//...
Game finished!
pong
{% endhighlight %}

¿Qué ha ocurrido? Tras solicitar a los threads su interrupción es posible que el hilo principal se adelante a esa finalización, y por eso el texto "Game finished!" aparece antes que la última jugada de "pong". Explorando la API `ExecutorService`, vemos que tiene un método llamado `awaitTermination`. Este método bloquea el thread que lo invoca hasta que todas las tareas del pool han terminado o expira un timeout que le proporcionamos por parámetro:

{% highlight java %}
//...
ExecutorService executor = Executors.newFixedThreadPool(2);

executor.execute(player1);
executor.execute(player2);

sleep(2);

executor.shutdownNow();

try {
    executor.awaitTermination(5, TimeUnit.SECONDS);
} catch (InterruptedException e) {
    System.out.println("Main thread interrupted while waiting for players to finish");
}

System.out.println("Game finished!");
//...
{% endhighlight %}

Ahora sí conseguimos la salida deseada, y el juego se comporta como queremos con una clase principal mucho más limpia y legible. ¿Hemos terminado? Aún no :)

#### Barreras

Las barreras de entrada / salida, son mecanismos de sincronización que facilitan la ejecución simultánea de un grupo de threads (barrera de entrada), o la espera hasta finalizar la ejecución de (otra vez) otro pool de threads.

La idea de la barrera de salida (exit barrier) la hemos visto en el punto anterior con `awaitTermination`. No obstante, aunque este método nos posibilita crear una barrera de salida también nos obliga a establecer un timeout (que aunque puede ser de horas no deja de ser un timeout). Nosotros querríamos crear una barrera de salida sin timeout alguno.

Para entender lo que es una barrera de entrada vamos a añadir una instrucción a `Game`:

{% highlight java %}
//...
executor.execute(player1);
sleep(1000);
executor.execute(player2);
//...
{% endhighlight %}

Dormimos el hilo principal durante un segundo antes de iniciar la ejecución del segundo jugador. Aunque el resultado es difícil de reproducir aquí, porque está relacionado con el timing de la aplicación, ocurre algo así:

{% highlight java %}
Game starting...!
ping
// Waiting 1 second
pong
{% endhighlight %}

Es decir, el jugador "ping" pelotea, ¡pero durante un segundo no tiene a nadie al otro lado! Por lo que el juego queda "suspendido" un segundo, que podrían ser minutos (el tiempo que el hilo principal tarde en lanzar la ejecución del segundo jugador). Esta situación no es ideal, porque estamos arrancando el funcionamiento de un proceso concurrente que requiere la presencia de varios threads antes de que todos estén listos. Para evitar esto necesitamos utilizar una barrera de entrada (entry barrier).

Existen varias clases en la API concurrency que pueden utilizarse con fines de barrera, pero la más sencilla, y la que utilizaremos en ambos (barrera de entrada y salida) es `CountdownLatch`. El uso de esta clase puede resumirse en tres puntos:

1. Creamos una barrera con un contador inicializado a **N**
2. Los hilos que dependan de la barrera para continuar invocarán `await()`, y quedarán bloqueados hasta que el contador llegue a cero. También existe un método `await()` con timeout
3. Los actores que pueden influir en la apertura de la barrera invocarán `countDown` cuando se cumplan las condiciones adecuadas para liberarla. En general deben cumplirse **N** condiciones para que la apertura tenga lugar

### Versión 6: Barreras de entrada / salida

En esta nueva versión deberemos modificar tanto `Game` como `Player`. Veamos como quedarían:

{% highlight java %}
public class Player implements Runnable {

    private final String text;

    private final Lock lock;
    private final Condition myTurn;

    private final CountDownLatch entryBarrier;
    private final CountDownLatch exitBarrier;

    private Condition nextTurn;

    private Player nextPlayer;

    private volatile boolean play = false;

    public Player(String text,
                  Lock lock,
                  CountDownLatch entryBarrier,
                  CountDownLatch exitBarrier) {
        this.text = text;
        this.lock = lock;
        this.myTurn = lock.newCondition();

        this.entryBarrier = entryBarrier;
        this.exitBarrier = exitBarrier;
    }

    @Override
    public void run() {
        if(entryBarrierOpen())
            play();
    }

    public boolean entryBarrierOpen() {
        try {
            entryBarrier.await();
            return true;
        } catch (InterruptedException e) {
            System.out.println("Player "+text+
                                " was interrupted before starting Game!");
            return false;
        }
    }

    private void play() {
        while (!Thread.interrupted()) {
            lock.lock();

            try {
                while (!play)
                    myTurn.awaitUninterruptibly();

                System.out.println(text);

                this.play = false;
                nextPlayer.play = true;

                nextTurn.signal();
            } finally {
                lock.unlock();
            }
        }

        exitBarrier.countDown();
    }

    public void setNextPlayer(Player nextPlayer) {
        this.nextPlayer = nextPlayer;
        this.nextTurn = nextPlayer.myTurn;
    }

    public void setPlay(boolean play) {
        this.play = play;
    }
}
{% endhighlight %}

La clase no comienza a jugar hasta que la barrera de entrada (`entryBarrier`) esté abierta, y cuando es interrumpido para terminar invoca a `countDown` sobre la barrera de salida (`exitBarrier`), que será la forma de que `Game` sepa que ambos jugadores han terminado.

Pensad por un segundo a qué valores iniciaremos los contadores de `entryBarrier` y `exitBarrier` antes de seguir leyendo...

{% highlight java %}
public class Game {

    public static void main(String[] args) {
        CountDownLatch entryBarrier = new CountDownLatch(1);
        CountDownLatch exitBarrier = new CountDownLatch(2);

        Lock lock = new ReentrantLock();

        Player player1 = new Player("ping", lock, entryBarrier, exitBarrier);
        Player player2 = new Player("pong", lock, entryBarrier, exitBarrier);

        player1.setNextPlayer(player2);
        player2.setNextPlayer(player1);

        System.out.println("Game starting...!");

        player1.setPlay(true);

        ExecutorService executor = Executors.newFixedThreadPool(2);

        executor.execute(player1);

        sleep(1000);

        executor.execute(player2);

        entryBarrier.countDown();

        sleep(2);

        executor.shutdownNow();

        try {
            exitBarrier.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Game finished!");
    }

    public static void sleep(long ms) {
        try {
            Thread.sleep(ms);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
{% endhighlight %}

En efecto, la barrera de entrada tiene un contador de 1, porque se abrirá tan pronto como el hilo principal haya pasado todas las tareas al pool de threads, mientras que la barrera de salida, que aquí se utiliza como alternativa a `awaitTermination`, tiene un contador de 2, que es el número de actores que debe finalizar su ejecución antes de que el hilo principal pueda proseguir.

De esta forma el timing de la aplicación es el deseado, aunque para ello hayamos tenido que complicar un poco el código. El tema es que la concurrencia de por sí es compleja, por lo que es difícil encapsular perfectamente todos los mecanismos utilizados.

Antes de terminar el post, mencionar que la barrera de salida ha sido añadida a esta versión a efectos didácticos. El mejor mecanismo para esperar la finalización de un grupo de threads en un pool es la espera mediante `awaitTermination`, introduciendo un timeout razonable, de forma que si alcanzamos el timeout sea porque algún fallo está ocurriendo en las tareas de las que esperamos su terminación. [En mi repositorio de GitHub](https://github.com/raulavila/blog-examples/tree/master/src/main/java/com/raulavila/pingpong) he añadido una versión 7 donde se utiliza la barrera de entrada y `awaitTermination` como barrera de salida, pudiéndose considerar ésta la versión óptima de la aplicación.

### Conclusiones

He disfrutado bastante la preparación y escritura de esta serie de tres posts sobre concurrencia, por lo que es bastante posible que la retome en algún momento en el futuro. Es por ello que no añado la coletilla "y 3" al título. El Multithreading es un tema muy complejo, y siempre hay cosas nuevas que aprender, lo cual es genial.
