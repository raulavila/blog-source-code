---
layout: post
title: Fowler's Refactoring. Un clásico con errores (parte 1)
permalink: 2015/02/fowler-refactoring-1
tags:
- refactoring
- desarrollo
comments: true
---

![Refactoring, el libro](/public/pictures/refactoring_fowler.jpg)

El libro ["Refactoring: Improving the design of existing code"](http://www.amazon.es/Refactoring-Improving-Design-Existing-Technology/dp/0201485672/ref=sr_1_1?ie=UTF8&qid=1423418116&sr=8-1&keywords=Refactoring), es un clásico absoluto de la Ingeniería del Software. Cualquier desarrollador experimentado debe conocer forzosamente muchas de las refactorizaciones propuestas en el libro. El propio autor las tiene recogidas online en [este catálogo](http://refactoring.com/catalog/), aunque sin entrar en muchos detalles (hay que seguir vendiendo el libro, cosa totalmente lícita). En pocas palabras, refactorizar consiste en modificar el código sin que la funcionalidad se vea afectada, y es algo totalmente necesario para la buena salud de nuestro código.

Recientemente he terminado de leer este compendio de técnicas para mejorar el código que desarrollamos, y la valoración general es muy positiva. Me ha servido para plantearme ciertas cosas que hago o dejo de hacer a diario, y diría que soy un poco mejor programador que antes de leerlo. Por supuesto lo recomendaría a cualquiera.

Sin embargo, diría que el tiempo no le ha sentado demasiado bien a este libro, y necesitaría de una segunda edición con relativa urgencia. Para empezar, estamos hablando de una obra publicada en 1999, y con todos los ejemplos basados en Java 1.2. Mucho ha llovido desde entonces, para entendernos en aquella versión de Java no existían ni las [enums](http://docs.oracle.com/javase/tutorial/java/javaOO/enum.html), ni los [generics](http://docs.oracle.com/javase/tutorial/java/generics/why.html), herramientas del lenguaje totalmente fundamentales hoy día, por poner un par de ejemplos. Varios de los refactorings carecen totalmente de sentido desde una perspectiva actual.

Pero en esta serie de artículos pretendo comentar errores que me han llamado poderosamente la atención, dejando de lado aspectos relativos a versiones más o menos antiguas del lenguaje. Estoy seguro de que el propio autor los corregiría si fuera posible. No es mi intención despreciar el trabajo de Fowler, por supuesto, yo a su lado soy un insecto, solo considero importante resaltar ciertas cosas que no estaría nada bien quedaran fijadas como buenas prácticas de programación por aparecer en los ejemplos del libro.

Si algún lector considera que me falta razón en alguno de los puntos, estaré encantado en comentarlo. Empezaré por el único fallo indiscutible se mire por donde se mire.

### ¿Clase inmutable?

En el refactoring "Introduce parameter object" (pág 295) aparece un ejemplo muy desafortunado de [clase inmutable](http://howtodoinjava.com/2012/10/28/how-to-make-a-java-class-immutable/):

{% highlight java %}

public class DateRange {
    
    private final Date start;
    private final Date end;

    public DateRange(Date start, Date end) {
        this.start = start;
        this.end = end;
    }

    public Date getStart() {
        return start;
    }

    public Date getEnd() {
        return end;
    }
}

{% endhighlight %}

¿Por qué es desafortunado este ejemplo? ¡Porque esta clase no es inmutable en absoluto! Veamos:

{% highlight java %}

@Test
public void testInmutableClass() throws Exception {
    Date start = parse("2001-01-01");
    Date end = parse("2001-01-02");

    DateRange range = new DateRange(start, end);

    Date dateAux = range.getEnd();
    
    //Modifies end date!
    dateAux.setTime(123123);

    assertThat(range.getStart()).isEqualTo(parse("2001-01-01")) ;
    
    //End date of the range modified!
    assertThat(range.getEnd()).isNotEqualTo(parse("2001-01-02"));
}

{% endhighlight %}

Para que una clase sea inmutable no basta con que sus atributos sean final y no proveer setters, si dichos atributos contienen referencias a objetos, dichos objetos han de ser también inmutables, cosa que no ocurre con la clase Date (¡cuántos quebraderos de cabeza nos ha dado esta clase a los desarrolladores Java!).

Si queremos usar de todas todas la clase Date para limitar los rangos de DateRange, la forma adecuada de que DateRange sea inmutable es crear "defensive copies" en los getters. Esta técnica está explicada estupendamente en el libro ["Effective Java"](http://www.amazon.es/Effective-Java-Programming-Language-Guide/dp/0321356683/ref=sr_1_1?ie=UTF8&qid=1423422393&sr=8-1&keywords=Effective+Java) de Joshua Bloch. Consiste en crear una copia del objeto a devolver en lugar del propio objeto. de esta forma el cliente no tiene una referencia de la instancia contenida en la clase inmutable, sino una copia, y puede hacer lo que quiera con ella, ya que la instancia original seguirá encapsulada en dicha clase inmutable:

{% highlight java %}

public class DateRangeInmutable {

    private final Date start;
    private final Date end;

    public DateRangeInmutable(Date start, Date end) {
        this.start = start;
        this.end = end;
    }

    public Date getStart() {
        return new Date(start.getTime());
    }

    public Date getEnd() {
        return new Date(end.getTime());
    }
}

{% endhighlight %}

En el siguiente test se puede comprobar como la clase es realmente inmutable:

{% highlight java %}

@Test
public void testInmutableClass() throws Exception {
    Date start = parse("2001-01-01");
    Date end = parse("2001-01-02");

    DateRange range = new DateRange(start, end);

    Date dateAux = range.getEnd();

    dateAux.setTime(123123);

    assertThat(range.getStart()).isEqualTo(parse("2001-01-01")) ;

    //End date of the range is not modified!
    assertThat(range.getEnd()).isEqualTo(parse("2001-01-02"));
}

{% endhighlight %}

Prefiero dejarlo aquí por hoy. Me he dado cuenta de que explicar cada ejemplo con detalle lleva su tiempo, así que iré desgranando los diferentes puntos en posteriores artículos.

Subiré todos los ejemplos en [este repositorio de GitHub](https://github.com/raulavila/fowlers-refactoring-errors).


