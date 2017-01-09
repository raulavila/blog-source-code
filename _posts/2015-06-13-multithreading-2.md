---
layout: post
title: Multithreading para dummies (2)
permalink: 2015/06/multithreading-2/
tags:
- Java
- desarrollo
- multithreading
comments: true
---

En [mi anterior post](/2015/05/multithreading-1/) vimos algunos conceptos básicos de multithreading desde un punto de vista eminentemente práctico. Con el desarrollo de un juego de Ping Pong en mente, continuaremos introduciendo mejoras progresivas que nos servirán para explicar conceptos que todos deberíamos conocer a la hora de implementar aplicaciones concurrentes en Java.

<!--break-->

La base de este post será una de las últimas versiones comentadas en la primera parte:

{% highlight java %}
public class Player implements Runnable {

    private final String text;

    private Player nextPlayer;

    private volatile boolean mustPlay = false;

    public Player(String text) {
        this.text = text;
    }

    @Override
    public void run() {
        while(!Thread.interrupted()) {
            while (!mustPlay);

            System.out.println(text);

            this.mustPlay = false;
            nextPlayer.mustPlay = true;

        }
    }

    public void setNextPlayer(Player nextPlayer) {
        this.nextPlayer = nextPlayer;
    }

    public void setMustPlay(boolean mustPlay) {
        this.mustPlay = mustPlay;
    }
}
{% endhighlight %}

Versión bastante horrible, por otro lado. En ningún caso está justificado que una aplicación seria haga algo como:

{% highlight java %}
while(!mustPlay);
{% endhighlight %}

##### Espera activa

Esta instrucción es un ejemplo de "espera activa" o ["Busy Waiting"](https://en.wikipedia.org/wiki/Busy_waiting), y no es más que la comprobación infinita de una condición, evitando el progreso de la aplicación hasta que sea cierta. El problema de este enfoque es que nuestro hilo sobrecarga de forma excesiva a la CPU, ya que para el [Thread Scheduler](http://www.javatpoint.com/thread-scheduler-in-java) no hay nada que le impida progresar, por lo que siempre que existen recursos lo mantiene en su estado "Running" ([aquí tenéis un buen diagrama de estados de los threads en Java](http://www.uml-diagrams.org/examples/java-6-thread-state-machine-diagram-example.html)). El resultado es un uso de recursos excesivo e injustificado.

Os voy a contar una historia curiosa para ilustrar esto que estoy explicando. Cuando desarrollé los ejemplos para la primera parte de este post, dejé mi IDE abierto con la aplicación funcionando (y la espera activa). El resultado es que mi batería, que normalmente dura una 6-8 horas se consumió en menos de dos. Pensemos en las consecuencias de un diseño tan defectuoso en aplicaciones corporativas serias.

#### Locking

La forma más fácil de deshacernos de la espera activa es mediante el uso de Locks. En pocas palabras, locking es un mecanismo que permite establecer políticas de exclusión en aplicaciones concurrentes cuando existen instancias cuyo estado puede ser compartido y modificado por diferentes threads.

Este estado susceptible de ser modificado por más de un thread debe protegrese mediante el uso de una sección crítica ([critical section](https://en.wikipedia.org/wiki/Critical_section)). Java ofrece diferentes mecanismos parar implementar secciones críticas, y en este post veremos los más importantes.

### Versión 3: Intrinsic locking

El mecanismo más antiguo implementado en Java para la creación de secciones críticas es conocido como [Intrinsic Locking](https://docs.oracle.com/javase/tutorial/essential/concurrency/locksync.html), o Monitor Locking. A grandes rasgos, cada objeto creado en Java tiene asociado un lock (intrinsic lock o monitor lock) que puede ser utilizado con fines de exclusión en nuestros threads mediante el uso de la keyword `synchronized`:

{% highlight java %}
//...
Object myObject = new Object();
//...
synchronized(myObject) {
    //critical section
}
{% endhighlight %}

En este ejemplo utilizamos una instancia de Object como lock, de forma que cada thread que desee acceder a la sección crítica debe obtener el lock, cosa que intenta hacer en la sentencia `synchronized`. Si el lock está disponible, el thread se lo queda y no estará disponible para ningún otro thread, que en caso de intentar obtenerlo fracasará y será puesto en estado "Blocked" por el Thread Scheduler.

Internet está plagado de ejemplos sobre el uso de `synchronized`, por lo que no entraré aquí sobre las mejores o peores prácticas. Solo añadir algunos puntos a considerar:

* Es habitual sincronizar en `this` (`synchronized(this)`), con lo que la propia instancia se utiliza a sí misma como lock para proteger a sus clientes de problemas de concurrencia. No obstante, hay que ser muy cuidadosos si hacemos esto porque los clientes podrían sincronizar en la misma instancia resultando en un [DeadLock](https://docs.oracle.com/javase/tutorial/essential/concurrency/deadlock.html)
* Personalmente considero mejor práctica utilizar un lock privado (como el utilizado en el fragmento de código tres párrafos arriba), de forma que no exponemos el mecanismo de locking utilizado al exterior encapsulándolo en la propia clase
* `synchronized` tiene otro fin además de la exclusión, y es la *visibilidad*. De la misma forma que la keyword `volatile` nos garantiza la visibilidad inmediata de la variable modificada, `synchronized` garantiza la visibilidad del estado del objeto utilizado como lock (abarcando más ámbito, pues). Esta visibilidad está garantizada por el [Java Memory Model](https://en.wikipedia.org/wiki/Java_memory_model), del que hablaremos algún día.

#### Mecanismos de espera

Tan solo con mecanismos de locking no podemos implementar correctamente la eliminación de la espera activa en nuestra aplicación. Necesitamos algo más, y son los mecanismos de espera.

Cada objeto expone un método, `wait()`, que al invocarse por un thread hace que el Thread Scheduler lo suspenda, quedando en estado "Waiting". Es decir:

{% highlight java %}
//thread state is running
i++
lock.wait(); // => thread state changes to Waiting
{% endhighlight %}

Este ejemplo está algo cogido con pinzas, porque nunca debe invocarse `wait` de esta forma. El "idiom" adecuado a la hora de implementar mecanismos de espera es:

{% highlight java %}
synchronized (lock) {
    try {
        while (!condition)
            lock.wait();

        //Excecute code after waiting for condition

    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
}
{% endhighlight %}

En el código vemos como:

1. Es necesario adquirir el lock sobre el objecto en el que queremos invocar `wait`
2. Ese `wait` implica que esperamos "algo". Ese algo es una condición (condition predicate) que puede que se cumpla antes de tener que esperar. Por tanto preguntamos por esa condición antes de invocar a `wait`
3. La espera se realiza en un bucle y no en una sentencia if por varios motivos, pero el más importante de ellos es el conocido como "spurious wakeups". Por su nombre es fácil de deducir en qué consiste, en ocasiones un thread se despierta del estado "Waiting" sin que nadie se lo haya indicado, por lo que puede que la condición no se esté cumpliendo y deba volver a esperar.
4. Por último, `wait` lanza la excepción `InterruptedException`, que manejamos de la forma comentada [en la primera parte de esta serie](/2015/05/multithreading-1/)

Visto esto, tenemos que un thread pasa a estado "Waiting" a la espera de una condición, pero alguien deberá indicar que uno o varios threads en espera deben despertarse, ¿no? Bien, esto se lleva a cabo mediante los métodos `notify` y  `notifyAll`, que como es fácil de deducir, indican a uno o a todos los threads esperando sobre un lock que se despierten y comprueben la condición. El idiom es:

{% highlight java %}
synchronized(lock) {
    //....
    condition = true;
    lock.notifyAll(); //or lock.notify();
}
{% endhighlight %}

De nuevo debemos tener el lock en nuestra posesión para poder invocar los métodos sobre el objeto. Sobre el uso de `notify` vs `notifyAll` [se ha escrito mucho al respecto](http://stackoverflow.com/questions/37026/java-notify-vs-notifyall-all-over-again), y depende de cada aplicación en concreto. Precisamente el uso de `notifyAll` es otro de los motivos por los que la espera de la condición se hace en un bucle y no en una condición, en ocasiones solo un thread de todos los que estén en espera puede progresar tras cumplirse el predicado.

Por fin ha llegado el momento de ver cómo quedaría nuestro juego de Ping Pong tras aplicar los conceptos que acabamos de ver:

{% highlight java %}
public class Player implements Runnable {

    private final String text;

    private final Object lock;

    private Player nextPlayer;

    private volatile boolean play = false;

    public Player(String text,
                  Object lock) {
        this.text = text;
        this.lock = lock;
    }

    @Override
    public void run() {
        while(!Thread.interrupted()) {
            synchronized (lock) {
                try {
                    while (!play)
                        lock.wait();

                    System.out.println(text);

                    this.play = false;
                    nextPlayer.play = true;

                    lock.notifyAll();

                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        }
    }

    public void setNextPlayer(Player nextPlayer) {
        this.nextPlayer = nextPlayer;
    }

    public void setPlay(boolean play) {
        this.play = play;
    }
}
{% endhighlight %}

El lock en esta aplicación vendría a ser la pelota en juego, que en cada jugada solo puede estar en posesión de un jugador. También vemos que tras imprimir el texto por salida estándar notifica al otro jugador que puede continuar. He utilizado `notifyAll`, aunque podría ser `notify` sin problemas.

La clase que conduce el juego no varía mucho sobre la última versión de la primera parte de esta serie:

{% highlight java %}
public class Game {

    public static void main(String[] args) {

        Object lock = new Object();

        Player player1 = new Player("ping", lock);
        Player player2 = new Player("pong", lock);

        player1.setNextPlayer(player2);
        player2.setNextPlayer(player1);

        System.out.println("Game starting...!");

        player1.setPlay(true);

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

        System.out.println("Game finished!");
    }

}
{% endhighlight %}

### Versión 4. Locks explícitos y condiciones

Java expone en su API concurrency una interfaz, `Lock`, que permite implementar los mismos mecanismos de exclusión vistos mediante el uso de intrinsic locks, pero con un acercamiento diferente.

La implementación principal de `Lock` es `ReentrantLock`. El nombre se debe a que los locks en Java son reentrantes, por lo que una vez adquirido por un thread, si el mismo thread realiza un nuevo intento de adquirirlo este no fracasa. Lo que haremos será implementar los mismos ejemplos vistos más arriba con esta API.

#### Secciones críticas

{% highlight java %}
Lock lock = new ReentrantLock();
//...
lock.lock();
try {
    //critical section...
} finally {
    lock.unlock();
}
{% endhighlight %}

Fácil, tan sólo tener en cuenta que debemos invocar el método `unlock` en la claúsula `finally` para garantizar que el lock es liberado incluso en caso de error.

Personalmente no diría que este mecanismo es mejor que el ofrecido por `synchronized`, siendo este último más compacto. Las grandes ventajas del uso de `Lock` vienen de una serie de métodos que nos dan la posibilidad de desarrollar mecanismos de locking más complejos como:

* `tryLock()`: intentamos adquirir el lock, pero el thread no se bloquea ni no lo consigue
* fairness: podemos crear un lock como "fair". Por defecto los locks en Java no lo son, por lo que un thread en espera puede ser el elegido para adquirir el lock aunque sea el último que ha llegado. Con un fair lock se implementará un locking FIFO

Os recomiendo echar un vistazo completo [a la API](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/locks/ReentrantLock.html) para más detalles.

#### Mecanismos de espera

La implementación de estos mecanismos se realiza mediante el uso de la clase [Condition](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/locks/Condition.html). La creación de una instancia de `Condition` debe hacerse siempre a partir de un `Lock`:

{% highlight java %}
Condition condition = lock.newCondition();
{% endhighlight %}

La clase `Condition` expone dos métodos, `await()` y `signal()` que vienen a ser el equivalente a `wait()` y `notify()` en los intrinsic locks. Además podemos utilizar otros métodos como:

* `await(long time, TimeUnit unit)`: espera a una condición no más del tiempo proporcionado por parámetro
* `awaitUninterruptibly()`: versión de `await()` no interrumpible. Es decir, si el thread que esté suspendido a la espera de una condición es interrumpido, este método no lanzará la conocida `InterruptedException`, por lo que solo pasará a estar activa si se invoca `signal()`/`signalAll()` sobre la condición (spurious wakeups aparte).

En general, para mecanismos de espera diría que el uso de `Condition` ofrece una seria de funcionalidades muy interesantes, además de permitir la creación de varias condiciones asociadas al mismo lock, cosa que no es posible (o si lo es su implementación es muy complicada) con intrinsic locks.

Veamos cómo queda nuestra aplicación mediante el uso de `Lock` y `Condition`:

{% highlight java %}
public class Player implements Runnable {

    private final String text;

    private final Lock lock;
    private final Condition myTurn;
    private Condition nextTurn;

    private Player nextPlayer;

    private volatile boolean play = false;

    public Player(String text,
                  Lock lock) {
        this.text = text;
        this.lock = lock;
        this.myTurn = lock.newCondition();
    }

    @Override
    public void run() {
        while(!Thread.interrupted()) {
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

Vemos como el uso de `Condition` hace más clara la lectura del código. Además, hemos utilizado el método `awaitUninterruptibly`, de forma que se garantiza fácilmente la consecución de la última jugada pendiente por parte de cada jugador cuando el hilo principal interrumpe los threads:

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

        System.out.println("Game finished!");
    }
}
{% endhighlight %}

### Bonus, escalando a N jugadores

Vamos a ver de qué forma tan sencilla podemos escalar el juego a varios jugadores, de forma que se vayan pasando la "pelota" entre ellos por orden. Es decir, la salida del programa sería algo como:

{% highlight text %}
Game starting...!
player0
player1
player2
player3
player4
player5
...
Game finished!
{% endhighlight %}

Resulta que, ¡no necesitamos modificar para nada la clase `Player`! En efecto, como cada jugador solo ha de ser consciente del siguiente en el juego, los únicos cambios necesarios tendrán que hacerse en la clase `Game`:

{% highlight java %}
public class GameScale {

    public static final int NUM_PLAYERS = 6;

    public static void main(String[] args) {
        Lock lock = new ReentrantLock();

        int length = NUM_PLAYERS;

        Player[] players = new Player[length];

        for (int i=0; i < length; i++) {
            Player player = new Player("player"+i, lock);
            players[i] = player;
        }

        for (int i=0; i < length - 1; i++) {
            players[i].setNextPlayer(players[i+1]);
        }
        players[length - 1].setNextPlayer(players[0]);

        System.out.println("Game starting...!");

        players[0].setPlay(true);

        //Threads creation
        Thread[] threads = new Thread[length];
        for (int i=0; i < length; i++) {
            Thread thread = new Thread(players[i]);
            threads[i] = thread;
            thread.start();
        }

        //Let the players play!
        try {
            Thread.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        //Tell the players to stop
        for (Thread thread : threads) {
            thread.interrupt();
        }

        //Don't progress main thread until all players have finished
        try {
            for (Thread thread : threads) {
                thread.join();
            }
        }  catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Game finished!");
    }

}
{% endhighlight %}

El código es algo más complejo, pero creo que se entiende bien. Tan sólo cambiando la constante podremos escalar el juego todo lo que queramos, y la concurrencia nos garantizará los turnos perfectamente :)

[En la siguiente entrega de la serie](/2015/06/multithreading-3) nos centraremos en la creación y gestión de threads, de forma que la clase `Game` sea mucho menos críptica de lo que es en esta última versión.
