---
layout: post
title: Acoplamiento temporal
permalink: 2015/07/acoplamiento-temporal/
tags:
- desarollo
- buenas-prácticas
comments: true
---

Creo que todos habremos leído más de un artículo o libro en el que, mencionando las buenas prácticas que debemos seguir a la hora de diseñar e implementar nuestras aplicaciones, se hace especial hincapié en dos conceptos:

* Alta cohesión ([high cohesion](http://stackoverflow.com/questions/10830135/what-is-high-cohesion-and-how-to-use-it-make-it)): sin ánimo de profundizar, la alta cohesión se consigue cuando una clase hace una labor bien definida, y  no múltiples tareas poco relacionadas. Una medida de alta cohesión podría ser el uso que los diferentes métodos de una clase hacen de las variables de instancia, si todos los métodos utilizan todas las variables de instancia la cohesión es alta. A nivel de módulo voy a poner un contraejemplo bastante significativo, la clase `java.util`, que contiene tanto las [Java Collections](http://docs.oracle.com/javase/7/docs/api/java/util/Collections.html) como clases para manejar fechas y demás. Curioso que algo tan definido como las Collections no tenga su propio paquete, pero evidentemente por motivos históricos y compatibilidad hacia atrás se ha tenido que quedar así

<!--break-->

* Bajo acoplamiento ([loose coupling](http://stackoverflow.com/questions/226977/what-is-loose-coupling-please-provide-examples)): dos clases están muy acopladas cuando su interacción se basa en detalles de implementacion y no en abstracciones, por lo que un cambio en la implementación de una de ellas forzará la modificación de la otra. Para conseguir bajo acoplamiento debemos seguir principios tan importantes como ["programa sobre interfaces y no sobre implementaciones"](http://www.fatagnus.com/program-to-an-interface-not-an-implementation/) o los [principios SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) (entre los que destaca el principio de inyección de dependencias, [del que ya hablamos](/2015/03/principios-dependencias))

Siempre me ha llamado la atención que no se preste atención al "acoplamiento temporal" (temporal coupling), quizás porque no es tan importante. De hecho no he encontrado demasiadas referencias en castellano, así que creo que merece la pena repasar en qué consiste y cómo evitarlo cuando sea posible.

###Diseñando una API

Cuando creamos una API, es de recibo hacerlo cuidadosamente de forma que sea clara para los clientes que la vayan a utilizar, que no exponga detalles internos de implementación (para conseguir el bajo acoplamiento), que no abarque más de lo necesario (alta cohesión), y a ser posible que sea intuitiva en su uso. La elección de nombres aquí es muy importante, y en ningún caso nuestros métodos deberían hacer más de lo que el usuario espera de ellos, lo que se conoce como "Principio de la mínima sorpresa" ([Principle of Least Surprise](http://www.catb.org/esr/writings/taoup/html/ch11s01.html)).

Vamos a crear una API que prepare un plato para un restaurante. Una primera versión podría ser:

{% highlight java %}
public interface Dish {
    void addIngredient(Ingredient ingredient);
    public void mix();
    public void cook();
    public void serve();
}
{% endhighlight %}

Se trata de una interfaz, así que creemos una implementación sencilla:

{% highlight java %}
public class DishImpl implements Dish {

    private final String name;
    private boolean mixed = false;
    private boolean cooked = false;
    private final List<Ingredient> ingredientList = Lists.newArrayList();

    public DishImpl(String name) {
        this.name = name;
    }

    public void addIngredient(Ingredient ingredient) {
        Validate.notNull(ingredient);

        System.out.printf("%s - Adding ingredient %s%n",
                                name, ingredient.getName());
        ingredientList.add(ingredient);
    }

    public void mix() {
        if(ingredientList.isEmpty())
            throw new IllegalStateException("There are no ingredients to mix");

        System.out.printf("%s - Mixing ingredients: %s%n",
                                name, ingredientList.toString());

        mixed = true;
    }

    public void cook() {
        if(!mixed)
            throw new IllegalStateException("Ingredients are not mixed");

        System.out.printf("%s - Cooking...%n", name);

        cooked = true;
    }

    public void serve() {
        if(!cooked)
            throw new IllegalStateException("Dish is not cooked");

        System.out.printf("%s - Serving...%n", name);

    }
}
{% endhighlight %}

La clase `Ingredient` es trivial:

{% highlight java %}
public class Ingredient {

    private final String name;

    public Ingredient(String name) {
        this.name = name;
    }
    //getter, toString...
}
{% endhighlight %}

¿Podríamos decir que este diseño cumple los principios reseñados más arriba?

* Alta cohesión: los diferentes métodos expuestos por `Dish` están claramente relacionados entre ellos, ya que forman parte del proceso de preparación de un plato
* Bajo acoplamiento: los clientes de esta clase la utilizarán únicamente mediante su interfaz, y necesitarán una dependencia adicional (la clase `Ingredient`). Siempre que respetemos la API no deberíamos preocuparnos por los detalles de implementación
* Principio de la mínima sorpresa: bueno, no parece que la invocación a `serve` haga nada más que servir el plato (no limpia el horno que se puede haber utilizado para prepararlo, por ejemplo :) ). Lo mismo con los demás métodos.

Parece que no está mal. Veamos cómo utilizaríamos la API:

{% highlight java %}
public class Chef {
    public void cookPaella() {
        Dish dish = new DishImpl("paella");

        dish.addIngredient(new Ingredient("rice"));
        dish.addIngredient(new Ingredient("chicken"));
        dish.addIngredient(new Ingredient("peas"));

        dish.mix();
        dish.cook();
        dish.serve();
    }
}
{% endhighlight %}

Si invocamos el método `Chef.cookPaella`, la salida sería:

{% highlight text %}
paella - Adding ingredient rice
paella - Adding ingredient chicken
paella - Adding ingredient peas
paella - Mixing ingredients: [Ingredient{name='rice'}, Ingredient{name='chicken'}, Ingredient{name='peas'}]
paella - Cooking...
paella - Serving...
{% endhighlight %}

###Qué es el acoplamiento temporal

Vamos a realizar una modificación muy pequeña en la clase `Chef`:

{% highlight java %}
public class Chef {
    public void cookPaella() {
        Dish dish = new DishImpl("paella");

        dish.addIngredient(new Ingredient("rice"));
        dish.addIngredient(new Ingredient("chicken"));
        dish.addIngredient(new Ingredient("peas"));

        // dish.mix();   => Don't mix the ingredients
        dish.cook();
        dish.serve();
    }
}
{% endhighlight %}

Hemos comentado el método `mix`, ya que consideramos que no es necesario mezclar los ingredientes antes de cocinarlos. La salida de la aplicación, en este caso, es:

{% highlight text %}
paella - Adding ingredient rice
paella - Adding ingredient chicken
paella - Adding ingredient peas
Exception in thread "main" java.lang.IllegalStateException: Ingredients are not mixed
	at com.raulavila.temporalcoupling.kitchen.DishImpl.cook(DishImpl.java:35)
	at com.raulavila.temporalcoupling.kitchen.Chef.cookPaella(Chef.java:14)
	at com.raulavila.temporalcoupling.kitchen.MainApp.main(MainApp.java:7)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:483)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:134)
{% endhighlight %}

¡La excepción nos indica que los ingredientes no están mezclados! Es decir, los métodos 'mix' y 'cook' están acoplados **temporalmente**, de forma que es necesario invocarlos en un orden determinado. En esto exactamente consiste el acoplamiento temporal. De hecho, no es el único que existe en la clase `Dish`, los métodos `cook` y `serve` también están acoplados temporalmente, y lo mismo ocurre con `addIngredient` y `mix`. En general, la API completa requiere que los métodos sean invocados siguiendo un orden determinado, o no ejecutará de forma correcta su tarea.

Como desarrolladores, deberíamos evitar en la medida de lo posible llegar a este tipo de condiciones de uso, ya que dificulta el desarrollo a terceras partes (los clientes de nuestra API). En el ejemplo está muy claro lo que ocurre, pero en otros casos se puede dar lugar a bugs escondidos y difíciles de seguir.

###Medidas para evitarlo

Primero de todo, no siempre es posible evitar completamente el acoplamiento temporal, así que, en caso de que no quede otro remedio la solución es hacerlo patente mediante excepciones explícitas en caso romperlo. Por tanto, la primera medida sería mejorar los mensajes de nuestras excepciones. En nuestro ejemplo, el método `cook` podría quedar así:

{% highlight java %}
public void cook() {
    if(!mixed)
        throw new IllegalStateException(
                "Ingredients are not mixed, please call mix first");

    System.out.printf("%s - Cooking...%n", name);

    cooked = true;
}
{% endhighlight %}

Un cambio tan sencillo dejará claro al usuario cómo ha de proceder en caso de error.

Pero antes de llegar a este punto debemos intentar eliminar los acoplamientos existentes, obligando al usuario a utilizar nuestra API de una forma determinada (= más rígida). En ocasiones mayor rigidez no signfica peor diseño. En el caso concreto de la API `Dish`, vamos a obligar al usuario a pasar en el constructor la lista de ingredientes que componen el plato, veamos cómo quedaría:

{% highlight java %}
public interface Dish {
    public void cook();
    public void serve();
}

public class DishImpl implements Dish {

    private final String name;
    private boolean mixed = false;
    private boolean cooked = false;
    private final List<Ingredient> ingredientList;

    public DishImpl(String name, List<Ingredient> ingredientList) {
        Validate.notEmpty(ingredientList);

        this.name = name;
        this.ingredientList = Lists.newArrayList(ingredientList);

        mix();
    }

    private void mix() {
        System.out.printf("%s - Mixing ingredients: %s%n",
                            name,
                            ingredientList.toString());

        mixed = true;
    }

    public void cook() {
        if(!mixed)
            throw new IllegalStateException(
                "Ingredients are not mixed, please call mix first");

        System.out.printf("%s - Cooking...%n", name);

        cooked = true;
    }

    public void serve() {
        if(!cooked)
            throw new IllegalStateException(
                "Dish is not cooked, please call cook first");

        System.out.printf("%s - Serving...%n", name);

    }
}
{% endhighlight %}

Hemos simplificado la interfaz, de forma que solo contiene los métodos `cook` y `serve`. Esto maximiza la flexibilidad de las implementaciones, ¡tanto que ni siquiera estamos obligados a utilizar instancias de `Ingredient` al uso! El constructor de la implementación obliga a los clientes a pasar una lista de ingredientes, y una vez almacenados localmente los mezcla sin necesidad de que lo hagan los usuarios. ¿No es más claro el uso ahora?

{% highlight java %}
public class Chef {
    public void cookPaella() {
        Dish dish = new DishImpl(
                "paella",
                Lists.newArrayList(
                        new Ingredient("rice"),
                        new Ingredient("chicken"),
                        new Ingredient("peas")
                ));

        dish.cook();
        dish.serve();
    }
}
{% endhighlight %}

La salida sería:

{% highlight text %}
paella - Mixing ingredients: [Ingredient{name='rice'}, Ingredient{name='chicken'}, Ingredient{name='peas'}]
paella - Cooking...
paella - Serving...
{% endhighlight %}

En este punto la única restricción es que los métodos `cook` y `serve` deben ejecutarse ambos y por este orden (siempre que nos queramos comer el plato, claro, siempre podemos cocinarlo sin más...). Podríamos rizar el rizo y fusionarlos en un nuevo método `cookAndServe` como método único en la interfaz, pero creo que ya han quedado bastante claros los conceptos (cómo dirían algunos, lo dejo como ejercicio :)). Por otro lado, un uso incorrecto de la API quedará expuesto inmediatamente por las excepciones lanzadas.

###Bonus: DSL's

En los últimos tiempos se está extendiendo mucho el uso de DSL's ([Domain Specific Language](http://www.infoq.com/articles/internal-dsls-java)), que a grandes rasgos son fluent API's (basadas en el patrón [Builder](https://en.wikipedia.org/wiki/Builder_pattern)), y permiten diseñar soluciones a problemas específicos de forma muy legible. Por ejemplo, el framework de integración [Apache Camel](http://camel.apache.org/) nos permite crear una ruta entre dos puntos con una transformación en medio tal que así:

{% highlight java %}
from("direct:in")
    .transform(body().append(" text appended"))
    .to("mock:result")
{% endhighlight %}

No creo que haga falta entrar en muchos detalles para entender lo que haría este pequeño fragmento de código. Este tipo de fluent API contiene decenas de métodos, existiendo en ocasiones acoplamiento temporal.

Lo importante si diseñamos DLS's es, como hemos comentado más arriba, exponer el uso incorrecto con mensajes lo suficientemente claros (refiriendo incluso a links con documentación, etc).

([El código, en GitHub](https://github.com/raulavila/blog-examples/tree/master/src/main/java/com/raulavila/temporalcoupling/kitchen))
