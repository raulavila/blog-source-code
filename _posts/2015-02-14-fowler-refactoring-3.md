---
layout: post
title: Desgranando Fowler's Refactoring (y 3)
permalink: 2015/02/fowler-refactoring-3
tags:
- refactoring
- desarrollo
comments: true
---

Termino con este post mi repaso a los "puntos mejorables" en el libro de Fowler. Insisto en mi consideración a esta obra como una referencia indiscutible, de hecho en casi 400 páginas sólo he encontrado unos pocos puntos dignos de discusión. Además si algún lector considera que he desbarrado en algún aspecto, admitiré encantado cualquier comentario.

<!--break-->

###Inline Temp (pág. 119)

Este refactoring es bastante sencillo, en el libro de Fowler se propone reemplazar

{% highlight java %}
double basePrice = anOrder.basePrice();
return (basePrice > 1000);
{% endhighlight %}

por

{% highlight java %}
return (anOrder.basePrice() >1000);
{% endhighlight %}

No diría que este refactoring es un gran error "per se", pero la experiencia me ha enseñado que siempre es mejor almacenar el retorno de un método en una variable temporal **para facilitar los procesos de debugging**. En efecto, si en algún momento necesitamos depurar y leer el valor de retorno del método basePrice, con la primera versión del código bastaría con ubicar un breakpoint en la sentencia return, mientras que en la segunda versión necesitamos navegar dentro del método basePrice (y si por cualquier motivo este método no utilizara variables temporales estaríamos en las mismas).

En resumen, no creo que la se obtenga un gran beneficio. Quizás podría discutirse que estamos creando una variable temporal "poco útil", pero la deuda técnica de no utilizarla es un potencial incremento en el tiempo de depuración. Otro peligro es la reutilización de variables temporales, pero esto es fácil de evitar utilizando el modificador final:

{% highlight java %}
final double basePrice = anOrder.basePrice();
return (basePrice > 1000);
{% endhighlight %}

###Replace Subclass with Fields (pág. 232)

Este refactoring, [descrito en la web de Fowler](http://refactoring.com/catalog/replaceSubclassWithFields.html), pretende eliminar jerarquías innecesarias, pero no diría que el resultado es especiamente brillante:

{% highlight java %}
public class Person {
    private final boolean male;
    private final char code;

    private Person(boolean male, char code) {
        this.male = male;
        this.code = code;
    }

    public static Person createMale() {
        return new Person(true, 'M');
    }

    public static Person createFemale() {
        return new Person(false, 'F');
    }

    public boolean isMale() {
        return male;
    }

    public char getCode() {
        return code;
    }
}
{% endhighlight %}

Disculpemos el uso de un char para indicar el sexo de la persona, debido a la no existencia de enums en aquel entonces. Aún así, sigue habiendo dos claros errores de diseño:

* El atributo male es totalmente redundante, pudiéndose extraer como code == 'M'. Recordemos, [Don't repeat yourself](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself).
* De nuevo vuelve a utilizar métodos factory estáticos en la propia clase. En este caso diría que no es tan grave [como en el ejemplo del artículo anterior](/2015/02/fowler-refactoring-2), pero sigue sin gustarme.

Mi propuesta de mejora contiene una enumeración y una clase Factory:

{% highlight java %}
enum Gender {
    MALE('M'),
    FEMALE('F');

    private final char code;

    Gender(char code) {
        this.code = code;
    }

    public char getCode() {
        return code;
    }
}

public class Person {
    private Gender gender;

    public Person(Gender gender) {
        this.gender = gender;
    }

    public boolean isMale() {
        return gender == Gender.MALE;
    }

    public char getCode() {
        return gender.getCode();
    }
}

public class PersonFactory {

    public static Person createMale() {
        return new Person(Gender.MALE);
    }

    public static Person createFemale() {
        return new Person(Gender.FEMALE);
    }
}
{% endhighlight %}

Mantengo el código de tipo char, ya que los clientes pueden seguir necesitando dicho identificador, pero en esta ocasión lo encapsulo dentro de un enumerado. El enumerado es de ámbito package, por lo que fuera de la factoria y la clase Person no puede ser utilizado.

( [El código de este apartado está subido en mi repositorio de GitHub](https://github.com/raulavila/fowlers-refactoring-errors) )

###Class.forName

En el refactoring "Replace Constructor with Factory Method" (pág 304), se crea un método factory genérico de la siguiente forma:

{% highlight java %}
public static Employee create(String name){
    try {
        return (Employee) Class.forName(name).newInstance();
    } catch (Exception e) {
        throw new IllegalArgumentException("Unable to instantiate" + name);
    }
}

//....
   return create("Engineer");
//....
{% endhighlight %}

Esta forma de crear instancias es totalmente inadecuada, porque se pierde seguridad de tipos ([type safety](http://en.wikipedia.org/wiki/Type_safety)). Lo mejor del caso es que el propio Fowler lo reconoce nada más plantear la alternativa. Bajo mi punto de vista, esta forma de instanciar objetos solo estaría justificada si el parámetro del método create está configurado fuera del ámbito de la aplicación (por ejemplo, en una base de datos). En tal caso, sería imprescindible cubrir con tests dicha configuración para evitar errores en tiempo de ejecución.



### Conclusiones

Y aquí termino mi repaso a este gran libro. Me dejo algunas cosas en el tintero, pero que creo que ya han sido discutidas sobradamente en la comunidad, como por ejemplo:

* El uso de [Checked Exceptions vs Unchecked Exceptions](http://stackoverflow.com/questions/6115896/java-checked-vs-unchecked-exception-explanation) (yo apenas utilizo Checked Exceptions de un tiempo a esta parte)
* Extender una clase mediante herencia o con un wrapper ([Inheritance vs. Composition](http://stackoverflow.com/questions/2150273/java-extend-or-wrap-a-class-to-add-extra-functionality))

 Concluyo citando el propio libro. En referencia a la importancia de refactorizar en un proyecto corporativo, donde las fechas de entrega añaden una presión difícil de soportar en ocasiones, y cuando los managers no entienden la importancia del Refactoring, el consejo de Fowler en estas ocasiones es claro e implacable: "No digas que refactorizas".

 >"Of course, many people say they are driven by quality but are more driven by schedule. In these cases I give my more controversial advice: Don't tell! Subversive? I don't think so. Software developers are professionals. Our job is to build effective software as rapidly as we can. My experience is that refactoring is a big aid to building software quickly. If I need to add a new function and the design does not suit the change, I find it's quicker to refactor first and then add the function. If I need to fix a bug, I need to understand how the software works—and I find refactoring is the fastest way to do this. A schedule-driven manager wants me to do things the fastest way I can; how I do it is my business. The fastest way is to refactor; therefore I refactor"
