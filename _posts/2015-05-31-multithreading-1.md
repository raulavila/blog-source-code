---
layout: post
title: Multithreading para dummies (1)
permalink: 2015/05/multithreading-1
tags:
- java
- desarrollo
- multithreading
comments: true
---

Voy a iniciar con este post una serie en la que introduciré conceptos básicos de programación concurrente / multithreading. Lo sé, Internet está plagado de referencias, tutoriales, artículos, etc, pero mi intención será dar un enfoque diferente en este blog. Lo normal, o al menos eso me ha parecido a mí siempre que he intentado empaparme de este tema es profundizar mucho en la teoría por un lado, o bien en ejemplos excesivamente complejos por otro.

Por mi parte lo que intentaré es no soltar demasiadas parrafadas en mis posts, y llevar a cabo un acercamiento práctico de los diferentes conceptos y API's que es necesario conocer para desarrollar aplicaciones concurrentes en Java. Además, trataré de hacerlo mediante el desarrollo de una aplicación extremadamente sencilla que irá creciendo en complejidad añadiendo un concepto nuevo cada vez. Espero conseguir mi objetivo.

<!--break-->

##La aplicación: Ping pong

Tan sencillo como eso. Nuestra aplicación Java deberá mostrar por salida estándar y de forma alterna los textos "ping" / "pong", además de un texto cabecera y de finalización:

{% highlight text %}
Game starting...!
ping
pong
ping
pong
//....
Game finished!
{% endhighlight %}

Conceptualmente deberán existir dos jugadores / actores, que deberán imprimir el texto "ping" y "pong". El actor "ping" deberá jugar primero.

###Versión cero: un solo hilo

La primera versión correrá en un solo hilo de ejecución, por lo que no habrá programación concurrente que valga :). En estas primeras versiones, además, el juego finalizará después de que ambos jugadores hayan participado un número de veces (definido por constante), pongamos 10.

Veamos como quedaría el código de la clase `Player`:

{% highlight java %}
public class Player {

    private final String text;

    private int turns = Game.MAX_TURNS;

    private Player nextPlayer;

    public Player(String text) {
        this.text = text;
    }

    public void play() {
        if (!gameFinished()) {
            System.out.println(text);
            turns--;
            nextPlayer.play();
        }
    }

    private boolean gameFinished() {
        return turns == 0;
    }

    public void setNextPlayer(Player nextPlayer) {
        this.nextPlayer = nextPlayer;
    }

}
{% endhighlight %}

Cada jugador imprime su texto y le dice al otro que juegue, por lo que se van alternando. La clase que inicia el juego sería bastante sencilla también:

{% highlight java %}
public class Game {

    public static final int MAX_TURNS = 10;

    public static void main(String[] args) {

        Player player1 = new Player("ping");
        Player player2 = new Player("pong");

        player1.setNextPlayer(player2);
        player2.setNextPlayer(player1);

        System.out.println("Game starting...!");

        player1.play();

        System.out.println("Game finished!");
    }
}
{% endhighlight %}

Poca historia. Vemos que en el constructor de `Player` no es posible pasar quien es el otro jugador porque no siempre va a estar instanciado, por lo que hay que hacerlo mediante setter.

También vemos como en `Player` el atributo `text` es declarado `final`. Es buena práctica en aplicaciones concurrentes (y en todas, la verdad) declarar un atributo como `final` si sabemos que no va a ser modificado. No sólo hace más fiable a nuestro código, también garantiza la visibilidad de las variables entre threads, un concepto conocido como "Safe publication", y del que podéis leer una discusión [aquí](http://stackoverflow.com/questions/801993/java-multi-threading-safe-publication). Yendo un poco más allá, siempre que podamos deberíamos diseñar nuestras clases como [inmutables](https://docs.oracle.com/javase/tutorial/essential/concurrency/imstrat.html), aunque en nuestro ejemplo no es posible.

###Versión 1: jugadores como threads

Vamos a llevar nuestra aplicación un poco más allá para que funcione en modo concurrente. Cómo la intención es hacerlo en incrementos pequeños nos iremos encontrando que nuestros primeros acercamientos no funcionan como es debido.

En primer lugar crearemos nuestra clase `Player` como `Runnable` (más información [aquí](https://docs.oracle.com/javase/tutorial/essential/concurrency/runthread.html)):

{% highlight java %}
public class Player implements Runnable {

    private final String text;

    private int turns = Game.MAX_TURNS;

    private Player nextPlayer;

    private boolean mustPlay = false;

    public Player(String text) {
        this.text = text;
    }

    @Override
    public void run() {
        while(!gameFinished()) {
            while (!mustPlay);

            System.out.println(text);
            turns--;

            this.mustPlay = false;
            nextPlayer.mustPlay = true;

        }
    }

    private boolean gameFinished() {
        return turns == 0;
    }

    public void setNextPlayer(Player nextPlayer) {
        this.nextPlayer = nextPlayer;
    }

    public void setMustPlay(boolean mustPlay) {
        this.mustPlay = mustPlay;
    }
}
{% endhighlight %}

El método principal (`run`) estará formado por un bucle que itera hasta que termina el número de turnos para el jugador. Además, en cada iteración del bucle se produce una [espera activa](http://es.wikipedia.org/wiki/Espera_activa) hasta que le llegue el turno. Una vez le toca imprime el texto, se indica a sí mismo que no le toca jugar hasta que alguien indique lo contrario, y le dice al siguiente jugador que es su turno.

Una gran diferencia entre esta clase `Player` y la anterior es que en la primera versión un jugador le indicaba al otro que jugara mediante paso de mensajes (invocando al método `play`), mientras que aquí se realiza modificando el valor de un flag (`mustPlay`), que cada jugador de forma individual es responsable de verificar.

Veamos cómo quedaría la clase `Game`:

{% highlight java %}
public class Game {

    public static final int MAX_TURNS = 10;

    public static void main(String[] args) {

        Player player1 = new Player("ping");
        Player player2 = new Player("pong");

        player1.setNextPlayer(player2);
        player2.setNextPlayer(player1);

        System.out.println("Game starting...!");

        player1.setMustPlay(true);

        Thread thread2 = new Thread(player2);
        thread2.start();
        Thread thread1 = new Thread(player1);
        thread1.start();

        System.out.println("Game finished!");
    }

}
{% endhighlight %}

La gran diferencia es que los threads se inician de forma separada, y nosotros solo somos responsables de configurar el flag `mustPlay` de forma adecuada. De hecho, he arrancado primero el thread de `player2` a propósito para confirmar que incluso así se imprime primero el mensaje "ping".

Veamos qué pasa al iniciar la aplicación:

{% highlight text %}
Game starting...!
ping
Game finished!
{% endhighlight %}

¿Qué ha ocurrido? Nuestra aplicación tiene ahora tres hilos:

* Hilo principal (`Game.main`)
* Hilo player1
* Hilo player2

El problema es que el hilo principal finaliza tan pronto como inicia los threads, por lo que aunque los otros dos hilos continúan su ejecución y finalizan correctamente nuestro IDE no recoge la salida de esos threads adicionales, creando además doble confusión al imprimir el mensaje "Game finished!". Para evitar esto una forma bastante directa es utilizar el método `join()`:

{% highlight java %}
public class Game {

    public static final int MAX_TURNS = 10;

    public static void main(String[] args) {

        Player player1 = new Player("ping");
        Player player2 = new Player("pong");

        player1.setNextPlayer(player2);
        player2.setNextPlayer(player1);

        System.out.println("Game starting...!");

        player1.setMustPlay(true);

        Thread thread2 = new Thread(player2);
        thread2.start();
        Thread thread1 = new Thread(player1);
        thread1.start();

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

Este método condiciona el progreso del hilo en ejecución a la finalización del hilo sobre el que se invoca el método `join`. Al ser join un método bloqueante es susceptible de ser interrumpido, por lo que lanza la checked exception `InterruptedException`. Más adelante hablaremos de las complicaciones que existen para interrumpir un thread.

Por tanto, ahora nuestro thread principal esperará hasta que los jugadores agoten sus turnos...o no? Bien, si ejecutamos la aplicación un par de veces, dependiendo de la suerte que tengamos es posible que todo sea correcto, pero si lanzamos ejecuciones reiteradas es más que posible que obtengamos una salida como la que sigue:

{% highlight text %}
Game starting...!
ping
{% endhighlight %}

¡Nuestra aplicación se queda bloqueada y no progresa en absoluto!

¿Qué ha pasado? Uno de los grandes problemas en la programación concurrente es la "visibilidad". Java solo garantiza la visiblidad de atributos compartidos entre threads si seguimos una serie de directrices, que vienen reguladas por el [Java Memory Model](http://en.wikipedia.org/wiki/Java_memory_model), y en concreto por la relación "happens-before". Según la Wikipedia, en Java esta relación viene a decir que:

>In Java specifically, a happens-before relationship is a guarantee that memory written to by statement A is visible to statement B, that is, that statement A completes its write before statement B starts its read

En nuestro código no estamos siguiendo ninguna convención que nos asegure la visibilidad de la modificación del atributo `mustPlay` entre threads. Evidentemente, la modificación de su propio `mustPlay` es visible para el propio thread, que no continúa jugando, pero de la forma en que lo estamos haciendo la modificación del atributo `mustPlay` del otro thread no se hace visible para el thread interesado, y nuestro programa queda en situación de bloqueo (o [deadlock](http://en.wikipedia.org/wiki/Deadlock)).

Para corregir este problema vamos a introducir el modificador `volatile`. [Este modificador](http://www.javamex.com/tutorials/synchronization_volatile.shtml) indica a la JVM que el atributo es susceptible de ser compartido entre threads, y que por tanto sus lecturas no deben ser cacheadas en modo alguno, accediendo siempre a memoria principal, y además sus escrituras deben hacerse de forma atómica y hacerse visibles de manera inmediata.

Nuestro código quedaría así:

{% highlight java %}
public class Player implements Runnable {
    //...
    private volatile boolean mustPlay = false;
    //....
}
{% endhighlight %}

Y ahora sí, nuestra aplicación funciona de forma determinista en cada ejecución. Uno de los mayores problemas de la visibilidad en aplicaciones concurrentes es que falla aleatoriamente, por lo que si no somos conscientes de las directrices a seguir, depurar estos problemas puede ser extremadamente complicado.

###Versión 2: juego infinito

En lugar de jugar un número determinado de turnos vamos a poner a los dos actores a jugar para siempre. O mejor dicho, hasta que el hilo principal quiera. Para ello, debemos utilizar las funcionalidades que ofrece Java para interrumpir un thread. Veamos cómo quedaría la clase `Game`:

{% highlight java %}
public class Game {

    public static void main(String[] args) {

        Player player1 = new Player("ping");
        Player player2 = new Player("pong");

        player1.setNextPlayer(player2);
        player2.setNextPlayer(player1);

        System.out.println("Game starting...!");

        player1.setMustPlay(true);

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

Vemos que, una vez iniciados los jugadores, la clase principal se duerme durante un tiempo (2ms), y a nada que regresa a su estado "running" **les pide** a los dos threads que finalicen.

Repito, **les pide**. Lo único que ocurre al invocar el método `interrupt` sobre un thread es que se pone a true un flag "interrupted" en ese thread. Es responsabilidad del propio thread actuar si lo desea, realizar labores de limpieza y finalizar. Pero bien puede decidir no hacer nada de nada y continuar con su ejecución (aunque eso no sería muy correcto, claro). La forma de consultar ese flag es mediante el método `Thread.interrupted()`, por lo que nuestra clase `Player` quedarían así:

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

En lugar de chequear en cada vuelta del bucle si hemos agotado turnos miramos el estado del flag "interrupted", y concluimos en caso de que sea true. Tan sencillo como eso.

####Versión 2b: Más sobre interrupt

Antes de finalizar este primer post de la serie, vamos a mirar un poco más en profundidad las implicaciones de interrumpir un thread.

En varias ocasiones hemos visto como algunos de los métodos de la clase `Thread` (`join`, `sleep`...) lanzan la excepción `InterruptedException`. Esto ocurre cuando un thread es interrumpido encontrándose en situación de bloqueo debido a la invocación de alguno de estos métodos. En tal caso, lo que ocurre es que el método limpia el flag "interrupted" en el thread en cuestión, y lanza la excepción `InterruptedException`. Sin ser yo muy fan de las checked exceptions, este es uno de los casos en los que encuentro más justificado su uso.

Modifiquemos ligeramente la clase `Player` para que, una vez le llegue el turno a un jugador se duerma durante 1ms antes de imprimir el texto:

{% highlight java %}
public class Player implements Runnable {
    //...
    @Override
    public void run() {
        while(!Thread.interrupted()) {
            while (!mustPlay);

            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println(text);

            this.mustPlay = false;
            nextPlayer.mustPlay = true;

        }
    }
    //...
}
{% endhighlight %}

Utilizando la misma versión de la clase `Game`, es altamente probable que el juego se ejecute indefenidamente. ¿Por qué? Porque si la interrupción le llega al thread mientras está durmiendo, el método sleep se traga el estado "interrupted" antes de lanzar la excepción, y como nosotros sólo nos hemos limitado a imprimir el error, el bucle no detecta este estado interrupted y continúa para siempre.

La solución a esto es restablecer el Thread a interrupted:

{% highlight java %}
@Override
public void run() {
    while(!Thread.interrupted()) {
        while (!mustPlay);

        try {
            Thread.sleep(1);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        System.out.println(text);

        this.mustPlay = false;
        nextPlayer.mustPlay = true;

    }
}
{% endhighlight %}

En general, hemos de ser muy cuidadosos a la hora de manejar `InterruptedException`. Otra estrategia recomendada, que implicaría modificar la lógica de nuestro método `run`, es volver a lanzar la excepción para que sea manejada en algún otro lugar. En ningún caso **nunca** debemos tragarnos la excepción sin más.

Quedan muchas mejoras por llevar a cabo, la aplicación está lejos de ser óptima (empezando por esa horrenda espera activa). [En el siguiente post](/2015/06/multithreading-2) añadiremos mejoras para optimizar el uso de la CPU mediante el uso de locks y condiciones.

(El código, como siempre, [en Github](https://github.com/raulavila/blog-examples/tree/master/src/main/java/com/raulavila/pingpong)).
