---
layout: post
title: Mi primera experiencia con TDD
description: Mi primera vez con Test Driven Development
permalink: 2015/08/primera-experiencia-tdd/
tags:
- TDD
- desarrollo
comments: true
---

[Test Driven Development](https://en.wikipedia.org/wiki/Test-driven_development), más conocido como TDD, es una metodología de desarrollo de software que en la última década ha ido cobrando más vigencia cada año. Inicialmente "inventado" por [Kent Beck](https://twitter.com/KentBeck), una de las figuras más importantes de este mundillo, adoptado por una parte importante del mundo [Agile](http://agilemethodology.org/), y especialmente por el movimiento [Software Craftsmanship](http://manifesto.softwarecraftsmanship.org/), a nivel personal mi acercamiento hasta hace nada había sido puramente teórico.

En efecto, puedo contar por decenas el número de ocasiones en que he visto vídeos, leído posts, capítulos de libros, asistido a conferencias, etc, introduciendo el tema. Pero nunca hasta esta semana había experimentado de primera mano lo que es desarrollar utilizándolo y los desafíos que plantea.

<!--break-->

El último año mi interés por el TDD ha ido en aumento. El factor que más me ha despertado la curiosidad es comprobar el entusiasmo con que los defensores del TDD hablan de ello, a pesar de que siempre reconocen la dificultad que entraña dominarlo ("no pain no gain" que dirían los ingleses). Por otro lado está claro que, sin ser un experto en absoluto, veo pocas pegas al uso de esta metodología. En ocasiones, cuando se evalúan los pros y contras de utilizar cualquier cosa (herramienta, framework, proceso de desarrollo, etc), siempre cuesta decidirse, porque hay contras claros que deben venir compensados por los pros. Pero en el caso de TDD, el único punto flaco que le veo es su curva de aprendizaje. ¿Debería ser esto una excusa para dejarlo de lado? ¡Por supuesto que no!

### Breve descripción del proceso y referencias

No es mi intención explicar en profundidad en qué consiste TDD, como ya he mencionado estoy en fase de aprendizaje, e internet está plagado de recursos. Pero sería imposible hablar de TDD sin mencionar los pasos básicos del famoso proceso red-green-blue:

1. Escribir un test fallido, un código no compilable se considera un test fallido (red)
2. Implementar el código de producción que haga ejecutar el test de forma exitosa (green)
3. Refactorizar (blue)
4. Volver al punto 1

Lo más chocante de todo la primera vez que se ve una demostración de este proceso es el extremo al que se lleva el punto 2, y que puedo resultar incluso poco intuitivo. Pongamos por ejemplo lo que tendríamos que hacer, siguiendo la metodología TDD, para hacer pasar este test:

{% highlight java %}
@Test
public void shouldReturn2_when1and1Added() {
    Calculator calculator = new Calculator();
    int result = calculator.add(1,1);
    assertThat(result).isEqualTo(2);
}
{% endhighlight %}

En primer lugar me gustaría resaltar que para este test estaríamos en una segunda iteración, porque la primera sería la que nos llevaría a crear la clase `Calculator` (sin métodos de momento), ya que la primera línea de nuestro test no compilaría (por tanto se considera un test fallido en el punto 1).

Pues bien, para hacer pasar este test, el código a implementar sería:

{% highlight java %}
class Calculator {
    public int add(int a, int b){
        return 2;
    }
}
{% endhighlight %}

¿A que choca? Bien, a mí también me pasó, pero una vez que se entienden los beneficios del TDD todo cobra sentido. Será en posteriores ciclos del proceso cuando vayamos refinando el método para que funcione correctamente con cualquier combinación de operadores.

Dejo la descripción del proceso aquí, pero si queréis seguir profundizando unos links básicos serían:

* [El libro de Kent Beck](http://www.amazon.es/Driven-Development-Example-Addison-Wesley-Signature/dp/0321146530/ref=sr_1_1?ie=UTF8&qid=1438434705&sr=8-1&keywords=Kent+Beck)
* El [Bowling Game Kata](https://www.youtube.com/watch?v=rklz35GWtrQ) de [Uncle Bob](https://twitter.com/unclebobmartin). En su libro [Clean Coder](http://www.amazon.es/Clean-Coder-Conduct-Professional-Programmers/dp/0137081073/ref=sr_1_1?ie=UTF8&qid=1438436471&sr=8-1&keywords=Clean+Coder), Uncle Bob cuenta como Kent Beck le presentó personalmente el proceso, su estupor inicial, y su conversión ferviente :). La implementación de este Kata es quizás uno de los mejores ejemplos que existen para entender qué es TDD y por qué es tan beneficioso.
* El libro [Diseño Ágil con TDD](http://www.carlosble.com/libro-tdd/) de [Carlos Blé](https://twitter.com/carlosble), uno de los máximos exponentes de TDD en España. Los ejemplos están en C#, pero se entiende muy bien sin importar el background de cada uno. Aunque existen muchos libros en inglés, creo que este está a la altura de las mejoras obras publicadas sobre el tema.

### Mi primera experiencia práctica. Conclusiones

El pasado miércoles asistí a un Meetup de la [London Software Craftsmanship Community](http://www.meetup.com/es/london-software-craftsmanship/). Fundada por [Sandro Mancuso](https://twitter.com/sandromancuso) (un tío de lo más pasional, autor de [The Software Craftsman](http://www.amazon.co.uk/books/dp/0134052501/ref=sr_1_1?ie=UTF8&qid=1438436878&sr=8-1&keywords=the+software+craftsman)), esta comunidad organiza eventos con mucha frecuencia en los que se fomenta el uso de buenas prácticas, con TDD a la cabeza.

En dicha sesión había que implementar la solución a un problema determinado utilizando TDD y programando por parejas. Yo tuve la suerte de que mi compañero tuviera más experiencia que yo en el tema, y fue una jornada bastante enriquecedora. Tras dos horas programando, se abrió un debate sobre los beneficios del uso de TDD.

Las conclusiones a las que llegué son:

* TDD es más que una metología de desarrollo de software, es una metodología de **diseño** de software. Puede que el nombre lleve a confusión en este sentido, y por eso me parece muy acertado el título del libro de Carlos Blé
* El mayor beneficio que obtenemos de su uso es el **descubrimiento progresivo de los requisitos de nuestra aplicación**. En el desarrollo tradicional tendemos al [Big Design Up Front](https://en.wikipedia.org/wiki/Big_Design_Up_Front), de manera que diseñamos mucho antes de tirar una línea de código, y esto nos lleva a estructuras de clases complejas que en ocasiones reflejan la realidad del problema, pero dificultan la implementación. En TDD la implementación va por delante del problema real, y es posible que en ocasiones no necesitemos representar actores que sí existen en la realidad porque el software soluciona el problema igualmente sin hacerse eco de ellos
* La curva de aprendizaje es elevada. Hablé con un español que lleva 7 años trabajando con TDD y me confirmó mis impresiones, ya que aunque él nunca abandonaría el TDD actualmente me reconoció que le llevó años dominarlo

En los próximos meses tengo intención de seguir aprendiendo TDD por mi cuenta, y quién sabe si en el futuro se me planteará la posibilidad de utilizarlo en un entorno profesional. Espero dejar cuenta de mis progresos y futuras impresiones en el blog.

### Una última reflexión

Es a menudo curiosa la forma en que muchas empresas (especialmente en UK) presumen del uso de TDD en ofertas de trabajo. Parece que da cierto caché atreverse a afirmar tal cosa, aunque luego no se utilice en realidad. Esto, además de absurdo, es contraproducente, porque puede llevar a situaciones de confusión bastante desagradables si un desarrollador especialmente cualificado en TDD (y por tanto, entusiasta del tema) entra a trabajar en una empresa para descubrir de inmediato que las prácticas en el día a día siguen siendo las de siempre. Habría que tener mucho cuidado con esto, yo aún no entiendo qué lleva a las compañías a requerir "skills" que luego no utilizan en sus proyectos.
