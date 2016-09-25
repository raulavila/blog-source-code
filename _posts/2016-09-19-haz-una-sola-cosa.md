---
layout: post
title: Haz una sola cosa
permalink: 2016/09/haz-una-sola-cosa/
tags:
- desarrollo
- 5-mantras
comments: true
---

Ya sabéis que tengo [5 mantras para escribir un mejor software](/2016/07/mis-5-mantras/) (aunque es posible que esta cifra aumente con el tiempo :)). En este post profundizaré un poco en el segundo "Do one thing, do it well, and do it only".

Diría que este principio no aplica solamente al software. La historia está llena de fracasos en productos desarrollados para hacer más de una cosa, pero ninguna de ella demasiado bien. En la mayoría de casos, además, consideramos tales inventos como auténticos engendros, como el [coche anfibio](http://www.hola.com/actualidad/2013073166071/panther-coche-anfibio/), la [guitarra / bajo](https://www.amazon.com/String-Bass-Double-Neck-Guitar/dp/B001F1UFL2), o la [super-navaja de Amazon](https://www.amazon.es/Wenger-19201-Navaja-suiza/dp/B000R0JDSI).

<!--break-->

Si todos nos reímos cuando vemos estas cosas, ¿por qué no nos sorprendemos cuando desarrollamos una clase que realiza mil y una tareas sin ningún tipo de orden ni concierto? Por ejemplo:

{% highlight java %}
public class BankUtil() {
    public String formatAccount(String accountNumber) {...}

    public boolean isNumber(String text) {...}

    public PersonDetails getPersonDetails(Stirng accountNumber) {...}

    //...
}
{% endhighlight %}

No sigo, ya sabéis a lo que me refiero. Es para evitar cosas como ésta que siempre tengo presente el mantra mencionado más arriba, que no es otra cosa que el Single Responsibility Principle (SRP) o [Principio de Responsabilidad Única](https://es.wikipedia.org/wiki/Principio_de_responsabilidad_%C3%BAnica). Este principio viene a decir (no es la primera vez que lo cito en este blog) que "las únidades de código deben tener sólo una razón para el cambio", o también "las mejores funciones o módulos son aquellos que tienen una única responsabilidad". Tras leer esto es posible que nos venga a la cabeza una pregunta...

##¿Qué es una responsabilidad?

Lo de hacer una sola cosa está muy bien, pero en la práctica esto no es tan sencillo. Vamos a empezar con el mínimo nivel en qué podemos definir responsabilidades, las funciones.

Una función tiene una única responsabilidad si:

* Su nombre es perfectamente descriptivo de lo que hace
* Dicho nombre no tiene conjunciones (And / Or / But), por lo que métodos como `washAndClean()` no cumplirían el SRP
* Preferiblemente no tiene efectos secundarios, y con esto quiero decir que no hará nada que no esperemos dado el nombre de la función y leyendo sus argumentos y tipo de retorno

Veamos a continuación ejemplos de funciones que no cumplen el SRP:

{% highlight java %}
public String getName(String id) {
    String name = namesById.get(id);
    auditService.registerAccess(id);
    return name;
}
{% endhighlight %}

En efecto, esta función `getName` está auditando sus accesos, lo cual puede resultar un poco confuso para los usuarios de nuestro código.

{% highlight java %}
public void parseAndSave(String messageStr) {
    Message message = parser.parseMessage(messageStr);
    messageDAO.save(message);
}
{% endhighlight %}

En este ejemplo es evidente que estamos llevando a cabo dos tareas bien diferentes, pero, ¿por qué deberíamos rediseñar esta función? ¿Qué define una responsabilidad? Para entender bien esto, pasemos al siguiente nivel de abstracción, las clases.

##Clases y responsabilidades

Una clase puede ser definida de muchas maneras, pero nadie se llevará las manos a la cabeza si la defino aquí como "familia de funciones con un objetivo bien definido". Este objetivo no es otra cosa que la responsabilidad de la clase, y dicha responsabilidad no es más que la implementación de una serie de soluciones a las necesidades de un **actor**.

Un actor es la personificación de un rol determinado que interactúa con nuestro sistema, y que es la audiencia de una responsabilidad determinada. Esta definición es muy flexible, y puede abarcar desde:

* El usuario final de una aplicación de móvil, y que puede dictaminar, tras un A/B testing, que es más adecuado ubicar un botón en la parte superior o inferior de la pantalla
* Sistemas externos con los que interactúamos a través de una API REST, y que pueden cambiar su contrato, haciéndonos modificar nuestros clientes de dicha API
* Los contables de nuestra empresa, que pueden añadir nuevos conceptos a contabilizar en un documento determinado que estemos generando
* Los arquitectos software de nuestra empresa, que por capricho pueden decidir cambiar un framework determinado y obligarnos a rehacer la implementación de una funcionalidad concreta para adaptarse a dicho framework

Es decir, un rol abarca aspectos muy heterogéneos.

La idea es que, cada clase deberá satisfacer las necesidades de un rol o actor determinado, **y sólo ese**. Esto se traduce en que si las necesidades de dos actores diferentes cambian al mismo tiempo, es materialmente imposible que dichos cambios apliquen en la misma clase.

##Por qué es importante

En primer lugar, si seguimos este principio jamás se dará la situación de que dos programadores realicen cambios en el mismo fichero si trabajan en tareas diferentes, evitando todo tipo de desagradables conflictos en el control de versiones. Pero esto es una ventaja pequeña comparada con la principal de todas, y que define el conocido como "valor primario del software":

>"Software is soft" (Uncle Bob)

Así es, el software es "blando", se puede moldear y modificar según nuestras necesidades. Si en un momento determinado esto deja de ocurrir el software perderá su principal valor y dejará de servirnos, porque en el mundo actual el cambio es constante y debemos adaptarnos a él tan rápido como sea posible, y de forma sostenible.

Este principio es incluso más importante que el "valor secundario del software", y que viene a decir "el software debe satisfacer las necesidades del usuario". Por increíble que parezca, el primer valor es más importante, porque si se da la situación de que un sistema que acabamos de desarrollar no satisface completamente las necesidades del usuario, siempre será posible transformarlo ("moldearlo") para que así sea, y cuánto mejor diseñado esté más sencillo será.

Volviendo al SRP, si una clase no lo cumple ocurrirán las siguientes cosas:

* El número de líneas de código de la clase comenzará a crecer sin ningún tipo de estrategia que limite el crecimiento
* Nuestra clase necesitará utilizar diferentes colaboradores muy heterogéneos (ejemplo: un cliente http, y una API de acceso a base de datos, entre otros). Esto hace que a la larga, decisiones que sólo deberían afectar a una pequeña porción de código tienen un efecto en cadena devastador
* Comenzaremos a compartir código entre diferentes responsabilidades, código que acoplará cosas que no deberían estarlo. Por ejemplo, una función de utilidad para parsear un mensaje determinado, utilizada a la vez tras extraer el cuerpo en una respuesta HTTP y un mensaje de la base de datos. Si por algún motivo, el contrato de la API REST cambia pero los mensajes en base de datos permanecen inalterados el impacto será relativamente grande
* Nuestras clases serán extraordinariamente difíciles de testear, principalmente mediante tests unitarios

##Cómo conseguirlo

La mejor forma de diseñar clases que cumplan con el SRP es siguiendo dos sencillas reglas:

* Desarrollo mediante TDD: los tests unitarios son la mejor forma de poner presión sobre nuestro diseño para que nuestras clases tengan responsabilidades bien definidas
* Refactorizando: la refactorización de nuestro código, para aumentar su legibilidad es clave. En ocasiones, mediante el refactoring emergen responsabilidades que se nos habían pasado por alto

Precisamente mi tercer mantra está muy relacionado con este último punto, y hablaré de él en el siguiente post.

##Otros niveles de abstracción: paquetes y aplicaciones

Tras las clases tenemos los paquetes, y aquí de nuevo deberíamos esforzarnos por crear paquetes bien cohesionados. Sobre este tema, os recomiendo encarecidamente que leáis [este esencial post](http://olivergierke.de/2013/01/whoops-where-did-my-architecture-go/), que explica las diferencias entre layers y slices, y cuál es la mejor forma de diseñar nuestros paquetes para ajustarse a estos criterios. En pocas palabras, viene a decir que no organicéis vuestros paquetes así:

* model
    * Account
    * Client
* service
    * AccountService
    * ClientService
* repository
    * AccountDAO
    * ClientDAO

Sino así:

* account
    * Account
    * AccountService
    * AccountDAO
* client
    * Client
    * ClientService
    * ClientDAO

Creo que queda clara la idea, y si queréis profundizar [ya sabéis](http://olivergierke.de/2013/01/whoops-where-did-my-architecture-go/).

Para terminar, pasemos al nivel de aplicación, algo que quizás puede sonar extraño, pero no lo es tanto en este mundo donde los [microservicios](http://martinfowler.com/articles/microservices.html) son el último grito. Este nivel puede que sea el único donde os diga de inicio que reconsideréis bien vuestra decisión antes de decidir desplegar una responsabilidad única como una aplicación independiente. Esa no fue la idea con la que surgieron los microservicios, y os remito a [este otro excelente post](http://highscalability.com/blog/2014/4/8/microservices-not-a-free-lunch.html) para que tengáis en cuenta todas las consideraciones necesarias en el momento que os decidáis por esta arquitectura.
