---
layout: post
title: Mis etapas con el refactoring
description:
permalink: 2017/07/etapas-refactoring/
tags:
- refactoring
- desarrollo
comments: true
---

Mi relación con el refactoring viene de lejos, de hecho entre los primeros posts que publiqué se encontraba [una serie de tres](/2015/02/fowler-refactoring-1/) donde analizaba de forma algo crítica el libro "Refactoring" de Fowler, y que seguramente hoy día escribiría de forma diferente.

Al cabo de los años he llegado a definirme a mí mismo, medio en coña medio en serio, como un "yonki del refactoring". En efecto, refactorizar código y dejarlo apañado es una de las tareas que más disfruto en mi día a día como desarrollador, tanto que hubo un momento determinado en que quizás hasta llegué a pasarme.

En este post voy a describir cada una de las etapas que he atravesado en mi relación con el refactoring. Supongo que muchos de vosotros os veréis reflejados en algunas de ellas.

<!--break-->

## Desconocimiento total

En la carrera nunca me hablaron de refactoring, y tampoco en mis primeros trabajos. [Como tampoco ponía demasiado interés en investigar demasiado por mi cuenta](/2017/01/yo-fui-un-mal-programador/), para mí este concepto no existió durante varios años. De forma que al principio de mi carrera profesional yo me dedicaba a tirar y tirar código como mejor sabía, y "con mucho cuidadito" para no romper nada de lo que habían dejado los anteriores programadores. Porque de tests automatizados de momento ni hablemos, claro.

## Reingeniería

La primera vez que escuché este palabro en uno de mis primeros proyectos no sabía muy bien lo que significaba. Resulta que en un proyecto en el que estaba trabajando llegamos a tener un módulo contenido en un único fichero fuente con más de 30.000 líneas (cágate lorito). Sumemos a esto que no existía un control de versiones en condiciones y todos los programadores teníamos que acceder a los mismos ficheros ubicados en el servidor de desarrollo. Pensad detenidamente lo que esto significa...en efecto, era necesario bloquear el fichero (lo abríamos con Vim) para que si otro venía después viera que alguien estaba editando y se abstuviera. Si el cambio era urgente tenías que preguntar en persona si el que lo tenía pillado podía liberarlo...una fiesta vamos.

Total, que un día, nuestro jefe de proyecto decidió coger el toro por los cuernos y decretar que era necesario hacer una "reingeniería" de este módulo, con el único motivo de aumentar la productividad de los desarrolladores al evitar estos bloqueos que comentaba. Vamos, que el motivo de fondo no fue mejorar la calidad ni nada parecido, sino evitar que se produjera una situación que cada vez era más incómoda. Esta labor fue encomendada a uno de los líderes de equipo del proyecto, porque por supuesto los programadores junior debíamos dedicarnos exclusivamente a transcribir los diseños que nos pasaba el analista.

Diría que esta "reingenría" fue la primera vez que viví de cerca un proceso de refactoring, porque a fin de cuentas se trataba de eso. La productividad del equipo mejoró bastante cuando David, nuestro jefe de equipo, terminó su labor, dividiendo este módulo monstruoso en varios submódulos, y diría que la sostenibilidad del código también, aunque no fuera ese el objetivo inicial.

No obstante, a pesar de vivir los beneficios del refactoring de primera mano, no investigué mucho más allá, de momento.

## Liderando mi propia reingeniería

Pasaron los años, y me encontraba trabajando en una pequeña empresa donde yo era el responsable único de una aplicación web, bastante crítica por otra parte. La aplicación fue creciendo y creciendo hasta llegar a un punto parecido al sistema que comentaba en la etapa anterior (¿pero es que no aprendía nada?), con clases de 10.000 líneas, código dupicado por doquier, y cosas así.

Así que llegó un momento en que no era posible continuar de forma sostenible (o lo que yo creía que era sostenible) sin hacer una "reingeniería", y se lo propuse a mi jefe, quien aceptó. Me llevó cosa de dos meses reestructurar el diseño, y recuerdo perfectamente como tenía que probar el sistema completo de forma manual tras cada cambio importante...Otra cosa que recuerdo es que desconocía todas las funcionalidades que ofrecen los IDEs para llevar a cabo este tipo de tareas automáticamente...En fin, es lo que había. Como ya he comentado varias veces con anterioridad, si os encontráis en las primeras fases de vuestra carrera buscad a toda costa un mentor que os guíe y os evite meteros en este tipo de pozos de los que cuesta mucho salir.

## Refactoring tímido

En un momento determinado me di cuenta de que estaba llegando a un punto muerto en mi carrera, y por fin decidí hacer todas las cosas que no había hecho anteriormente, empezando por leer [Clean Code](https://www.amazon.es/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882/ref=sr_1_1), aprender algo de testing (poquito de momento), etc. De repente empecé a ver que el código que anteriormente escribía y me parecía bastante decente, era más bien una birria, y tenía mucho que mejorar.

Así que comencé a dar la importancia que se merecen los nombres de las cosas (variables, métodos, clases...), el tamaño de nuestros métodos...por entonces no tenía muy claros los [principios SOLID](https://es.wikipedia.org/wiki/SOLID), pero algo iba mejorando. Tampoco me atrevía a refactorizar (sí, por fin sabía que refactoring era una palabra que existía) con demasiada agresividad por miedo a romper cosas que hubieran hecho otros, y es que aún no podíamos sacar pecho de nuestra suite de tests en el proyecto en que estaba trabajando entonces.

## Refactoring constante (e irresponsable)

[Me mudé a Londres](/2015/04/trabajo-londres/), y en mi primer trabajo encontré un compañero obsesionado con la calidad, el [Single Responsibility Principle](https://es.wikipedia.org/wiki/Principio_de_responsabilidad_%C3%BAnica) (sobre todo), el clean code...Yo por mi cuenta me estaba leyendo el libro [Refactoring](https://www.amazon.es/Refactoring-Improving-Design-Existing-Technology/dp/0201485672/ref=sr_1_1) de Fowler, y si a todo esto sumamos que iniciamos un proyecto desde cero con tests en condiciones, [que aprendí a utilizar los IDEs como dios manda](/2015/01/entornos-integrados-desarrollo/)... el tema del refactoring se me subió a la cabeza, y no podía parar de refactorizar código por allá donde pasaba. ¿Por qué no hacerlo? Era increíblemente divertido, los tests me garantizaban que no rompía nada...en lo que no pensé con detenimiento fue en el coste que todo esto conllevaba a mi empresa.

## TDD

[Cuando conocí el TDD](/2015/08/primera-experiencia-tdd/), y lo practiqué de verdad, mi relación con el refactoring empezó a cambiar, e intentaba aplicarlo cuando le llegaba el turno dentro del ciclo Red / Green / Refactor, y si de verdad el diseño del código / sistema lo necesitaba. Ya no refactorizaba cada vez que ponía las manos sobre un nuevo proyecto como si me fuera la vida en ello, entre otras cosas porque comencé a entender que como profesionales debemos aportar un valor a nuestra empresa a cambio del dinero que ella invierte en nosotros, y en muchas ocasiones refactorizar como fin en sí mismo no está aportando tal valor. A fin de cuentas, no estamos añadiendo ninguna funcionalidad nueva, y el cuento de que el sistema será más mantenible en el futuro puede sonar a buena excusa, aunque se cae sola si pensamos en la frecuencia con que ese evento ocurrirá.

Una vez leí que las personas nos preocupamos en un alto porcentaje de ocasiones (que creo recordar rondaba el 80%) por cosas que nunca ocurren. Pues bien, exactamente lo mismo puede ocurrir con los refactorings si se hacen de forma irresponsable, con la consecuencia además de que nos hacen perder dinero.

## Refactoring responsable

[Este artículo de David Bryant Copeland](http://naildrivin5.com/blog/2013/08/08/responsible-refactoring.html) (autor del excelente libro ["The Senior Software Engineer"](/2017/02/senior-software-engineer-book/)) confirmó del todo las dudas que iba teniendo acerca de la conveniencia del "refactoring por el refactoring". No voy a repetir en este post casi nada de lo que cuenta el artículo, [de obligada lectura](http://naildrivin5.com/blog/2013/08/08/responsible-refactoring.html).

En resumen, lo que viene a decir es que dentro del ciclo TDD, cuando estamos desarrollando un nuevo sistema, el refactoring es perfecto, pero que cuando nos toca mantener un sistema que está en producción, hay que pensárselo dos veces, ya que podemos incurrir en bugs que no harán sino aumentar el coste de una tarea que hemos realizado y nadie nos ha pedido, por mucho que nos guste.

¡Ojo!, ni el artículo ni yo decimos que no haya que refactorizar, tan sólo que hay que hacerlo únicamente cuando sea la mejor forma de alcanzar un fin determinado, como puede ser arreglar un bug, añadir una nueva funcionalidad a un código extremadamente rígido, etc.

Como ejemplo de esto último, recientemente me encontré en un proyecto una historia de usuario que requería invertir el orden en que dos pasos de negocio se estaban realizando. Resulta que esos pasos estaban distribuidos en clases diferentes, con una herencia compleja, varios [acoplamientos temporales](/2015/07/acoplamiento-temporal/)...y el cambio no era para nada trivial. Decidí (más bien decidí junto a la persona con quien hacía pairing) que llevar a cabo un refactoring responsable, eliminando jerarquías innecesarias, [favorenciendo composición sobre herencia](https://en.wikipedia.org/wiki/Composition_over_inheritance), etc, era la mejor forma de preparar el código para el cambio. Tomada esta decisión, tras un día de trabajo (y con una buena suite de tests dándonos feedback de nuestros cambios, por supuesto), los pasos que había que intercambiar en nuestro código quedaron tal que así:

{% highlight java %}
public void method() {
    //...
    stepA();
    stepB();
    //...
}
{% endhighlight %}

Ya os podéis imaginar lo fácil que fue hacer que `stepB()` ocurriera antes que `stepA` con esta nueva versión del código.

Desconozco si en el futuro entraré en una nueva etapa dentro mi particular relación con el refactoring, pero de momento me encuentro en esta última :). Y vosotros, ¿qué opináis? ¿Habéis tenido experiencias similares?
