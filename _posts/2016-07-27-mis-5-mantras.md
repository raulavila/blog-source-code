---
layout: post
title: Mis cinco mantras
permalink: 2016/07/mis-5-mantras/
tags:
- desarrollo
comments: true
---

En mi día a día, cuando estoy desarrollando software, cosa que ocurre la mayor parte del tiempo, intento seguir una serie de principios y buenas prácticas de cara a producir código con la máxima calidad. La mayoría conocemos, al menos de oídas, cuales son esos principios, aunque no siempre los apliquemos: [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself), [SOLID](https://es.wikipedia.org/wiki/SOLID), etc.

Siglas aparte, y a un nivel algo diferente, con el paso de los años he ido interiorizando una serie de mantras que están presentes en mi cabeza de forma permanente, y creo que me han sido de gran ayuda para mejorar como programador. Últimamente, y gracias al entorno de pair programming en que me muevo, los he empezado a dar aún más importancia, ya que en varias ocasiones me he visto en la tesitura de tener que explicar sus beneficios al que fuera mi compañero en ese momento.

En este post pasaré lista a estos "mantras", que en general no son más que lecciones aprendidas en determinados libros o video-tutoriales, y que han ido calando en mi subconsciente. Me permitiré la licencia de ponerlos en inglés, ya que así los aprendí y así los utilizo, y los listaré por orden de importancia.

<!--break-->

##Don't leave broken windows

"No dejes ventanas rotas". Sin lugar a dudas la sección más importante del libro [The Pragmatic Programmer](https://www.amazon.co.uk/Pragmatic-Programmer-Andrew-Hunt/dp/020161622X), donde lo leí por primera vez.

Resulta que está (¿científicamente?) demostrado que, cuando un edificio está abandonado, puede aguantar un tiempo razonable sin que la gente que lo ve a diario se percate de la situación. Pero siempre llega un momento donde se produce un hecho crucial que lo cambia todo: una ventana rota. Dicha ventana, al tratarse de un edificio abandonado, no es reparada, por lo que rápidamente se hace patente la situación de abandono, los vándalos comienzan a hacer de las suyas, irrumpen en el edificio...y el deterioro es imparable.

Exactamente la misma idea debe aplicarse al desarrollo software. Cuando tomamos determinados atajos para entregar algo en el menor tiempo posible, y nuestro código refleja claramente nuestro descuido, los que vengan después (que podemos ser nosotros mismos, programadores que compartan equipo en este momento u otros por llegar) sacarán como lectura que no es tan importante ser cuidadosos con el código que producimos, y poco a poco, y cada vez más rápido, nuestra aplicación o sistema se irá deteriorando.

Ejemplos de ventanas rotas pueden ser cosas tan flagrantes como no escribir tests automatizados de una nueva funcionalidad, pero también otras menos importantes a priori como no ser consistentes en nuestro estilo de codificación (utilizar diferentes indentaciones, formatear los bloques de forma diferente, etc) o no eliminar dependencias que nuestro código ha dejado de utilizar.

Por favor, no dejéis ventanas rotas. ¡Ojo!, que no estoy hablando de [deuda técnica](https://es.wikipedia.org/wiki/Deuda_t%C3%A9cnica), que puede estar justificada en determinadas circunstancias (siempre que seamos conscientes de estar incurriendo en dicha deuda y volvamos a ella en el futuro).

##Do one thing, do it well and do it only

"Haz una cosa, hazla bien, y sólo esa". Este mantra se lo debo a [Uncle Bob](https://twitter.com/unclebobmartin), y no es más que el [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle) formulado de forma pegadiza :)

No creo que sea necesario profundizar mucho aquí. Nunca, **nunca**, debemos añadir más de una responsabilidad a nuestros métodos o clases. En general, mi norma es que si necesitas una conjunción ("y...", "o...") para describir lo que hace un determinado método, tu método está haciendo demasiadas cosas.

Otra forma de ver esto es pensar desde la perspectiva de los diferentes actores involucrados en nuestro sistema. Cada actor podrá motivar cambios en nuestro código, y nosotros deberemos diseñarlo de forma que los cambios de opinión de cada actor pueda motivar cambios en un lugar perfectamente aislado. Ejemplos de actores pueden ser: nuestro sistema de transporte (http, colas de mensajes...), el equipo de seguridad de nuestra empresa, los diseñadores gráficos, nuestro sistema de bases de datos...muy heterogéneo vaya.

##Extract till you drop

Algo así como "extrae hasta que te caigas a pedazos". También se lo debo a Uncle Bob, y en su serie de videos [Clean Coders](https://cleancoders.com/) lo complementa con esta sentencia:

>1. Methods should be small
>2. They should be smaller than that

Dice nuestro buen amigo que un método no debería tener más de cuatro o cinco líneas de código. Yo creo que quizás se pase un poquito, pero la verdad es que, personalmente, cuando veo que un método tiene más de 10 líneas ya empiezo a sospechar.

Extraer nuestro código en métodos pequeños es extremadamente sencillo hoy día con los IDE's, y el resultado, si lo hacemos bien, es infinitamente mejor. Por otra parte esto no sólo aplica a métodos, también a clases e incluso a paquetes o módulos.

Me voy a ahorrar un ejemplo, ya que no me veo capaz de hacerlo mejor que [el autor de este mantra](https://sites.google.com/site/unclebobconsultingllc/one-thing-extract-till-you-drop). Y por favor, si no lo habéis hecho ya, leed [Clean Code](https://www.amazon.es/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882).

##Abstractions in code, details in data

"Abstracciones en código, detalles en datos". Esto lo aprendí en el curso ["World's best intro to TDD"](http://www.jbrains.ca/permalink/the-worlds-best-intro-to-tdd-demo-video), aunque indirectamente lo venía aplicando en determinadas circunstancias.

La clave de este mantra es que, cuando nos vemos codificando un método o clase con diferentes caminos de ejecución que dependen de datos de entrada o del contexto en que nuestro código se está ejecutando, suele ser posible abstraer los datos concretos fuera de nuestro código. De esta forma disminuimos la [complejidad ciclomática](https://es.wikipedia.org/wiki/Complejidad_ciclom%C3%A1tica) y facilitamos el testing, dos grandes beneficios que mejorarán infinitamente la calidad de nuestro software.

Veamos un ejemplo muy sencillo, el siguiente código devuelve un saludo a mostrar a un usuario según su perfil:

{% highlight java %}
public String fetchGreetingMessage(String role) {
    String greeting;

    if ("MANAGER".equals(role)) {
        greeting = "Hello Manager!";
    } else if ("GUEST".equals(role)) {
        greeting = "Hello Guest!";
    } else if ("EMPLOYEE".equals(role)) {
        greeting = "Hello Employee!";
    } else {
        throw new RuntimeException("Unrecognized role!");
    }

    return greeting;
}
{% endhighlight %}

Estoy seguro de que os habéis encontrado con métodos de similar estructura más de una vez. Una alternativa que se adhiera a este principio podría ser:

{% highlight java %}
class GreetingService {
    private Map<String, String> greetingByRole =
                ImmutableMap.<String, String>builder()
                                .put("MANAGER", "Hello Manager!")
                                .put("GUEST", "Hello Guest!")
                                .put("EMPLOYEE", "Hello Employee!")
                            .build();


    public String fetchGreetingMessage(String role) {
        if (greetingByRole.containsKey(role)) {
            return greetingByRole.get(role);
        } else {
            throw new RuntimeException("Unrecognized role!");
        }
    }
}
{% endhighlight %}

Claramente, los beneficios de este diseño es que nuestro método se ha convertido en inmutable (adhiriéndose de paso, al [Open Closed Principle](https://en.wikipedia.org/wiki/Open/closed_principle)), sólo contiene dos caminos de ejecución en lugar de cuatro, y si necesitamos añadir nuevos roles (con sus saludos) solo tendremos que añadir una nueva entrada al Map `greetingByRole`, mientras que en la primera versión deberíamos modificar el método para cada nuevo rol.

Podría discutirse que la clase `GreetingService` sigue sin ser inmutable, pero esto podríamos solucionarlo extrayendo la información del mapa en ficheros de configuración o de propiedades, por ejemplo, e inyectando un mapa equivalente creado por el contenedor de inyección de dependencias.


##Make it work, make it better, make it pretty

"Haz que funcione, hazlo mejor, hazlo bonito". Esto lo he aprendido recientemente, y es el enfoque que utilizamos en mi empresa durante el desarrollo mediante TDD. Cuando tenemos que desarrollar una determinada funcionalidad solemos seguir estos pasos:

* Comenzamos escribiendo los tests que la describan y guíen nuestra codificación y diseño
* Nuestro objetivo es conseguir que nuestros tests se ejecuten de forma exitosa (make it work)
* Una vez conseguido nos paramos a pensar en si el diseño puede mejorar (make it better)
* Y una vez llevados a cabo los refactorings que consideremos oportunos, eliminamos de nuestro código cualquier "ventana rota" para dejarlo lo más limpio posible (make it pretty)

Encuentro este enfoque increíblemente efectivo, pero la verdad es que no tiene sentido si no trabajáis con TDD, así que en tal caso, [empezad por ahí](/2016/01/aprendiendo-TDD/).

Y con esto termino mis mantras. Si tú tienes otros, estaría encantado de que los añadieras en la sección de comentarios.
