---
layout: post
title: Pair Programming
permalink: 2016/08/pair-programming/
tags:
- buenas-practicas
- me
comments: true
---

Hará algo más de cuatro años, hice una entrevista para una empresa en la que trabajaban utilizando Extreme Programming como metodología. En aquel momento, yo no tenía ni idea de lo que era eso, así que me contaron un poco en qué consistía, y fue algo así: "Todas las mañanas tenemos una reunión de pie, para que así sea más corta, donde hablamos de lo que hicimos el día anterior. Luego, nos ponemos entorno a ese tablón lleno de post-its, que representan tareas, y cada pareja, porque aquí programamos en pareja, elige una tarea y trabaja en ella".

Aquello me sonó a algo extremadamente diferente a todo lo que conocía, y, la verdad, me tiró un poco para atrás, sobre todo el hecho de estar todo el día trabajando con alguien en el mismo ordenador. "¡No podré ponerme música!", pensé. Eso, sumado al hecho de que las tecnologías que utilizaban no me llamaban en exceso, y que me encontraba bastante cómodo en mi puesto de trabajo, me hicieron rechazar su oferta.

<!--break-->

Estamos en 2016. Llevo viviendo en Londres desde 2014, y cuando tomé la difícil decisión de dar el salto tenía claro que si lo hacía intentaría sacar el máximo provecho a esta experiencia. Así que desde el primer día empecé a asistir a meetups, workshops, etc. En muchos de estos workshops se trabaja por parejas en problemas con el objetivo de familizarse con tecnologías o prácticas. El año pasado asistí al [Global day of code retreat](/2015/11/code-retreat-2015/), que me pareció una pasada, y donde básicamente te tiras todo el día trabajando en pareja con diferentes personas.

Comencé a pensar que el pair programming no estaba tan mal. Lo disfrutaba mucho, notaba que aprendía un montón, y eso sumado a mi creciente interés por el [TDD](/2015/08/primera-experiencia-tdd/), el Spring Framework y las plataformas Cloud se materializó en mi candidatura para trabajar en [una empresa](https://pivotal.io/) que sumaba todo esto y más.

Seis meses después de haber empezado a trabajar aquí, creo que me siento en posición de dar mi opinión sobre lo que supone trabajar con Pair Programming **full time**. Sí, habéis leído bien, trabajar de 9 a 6 compartiendo ordenador con una persona. ¡Ojo!, compartimos ordenador, pero no teclado y ratón, ya que cada miembro de la pareja tiene uno propio. Lo habitual es que una persona conduzca la codificación, mientras la otra intenta abstenerse y limitarse a revisar y hablar sobre lo que se está haciendo. Para evitar el aburrimiento ambos roles se van turnando con frecuencia.

##Los pros

Son muchos, y con un peso específico determinante sobre los contras. A saber:

* Al trabajar dos personas en el mismo problema, y tener que discutir cada aspecto de la codificación, no se cometen el mismo número de fallos "tontos" que cuando se trabaja en solitario. Este tipo de fallos son muchas veces los que nos hacen perder más tiempo cuando algo no funciona, y seguramente todos tengáis alguno reciente en mente
* Aprendes un montón: esto ocurre porque tu pareja siempre sabrá algo que tú no, desde cosas sencillas como herramientas de líneas de comandos, hasta cosas más genéricas como patrones, pasando por funcionalidades del lenguaje que se esté utilizando. Este punto alcanza su máxima expresión cuando tienes que programar en un lenguaje que es nuevo para tí pero no para el otro. No creo que haya mejor forma de aprender un nuevo lenguaje de programación que mediante el pair programming. Creedme, la experiencia es tremenda
* Enseñas un montón: de la misma forma que el otro sabrá cosas que tú no, la inversa también es cierta, y te verás en muchos momentos explicando cómo hacer determinadas cosas. Dejando aparte lo satisfactorio que es esto, también notarás que enseñar es una buena forma de revisar tus conocimientos y descubrir lagunas en lo que sabes
* El conocimiento es compartido: la rotación de las parejas es frecuente, por lo que se elimina casi por completo el problema del "single point of failure", donde la ausencia de un miembro del equipo hace temblar a todo el mundo. En efecto, mediante esta técnica es muy complicado que alguien acapare conocimiento y se haga imprescindible. Esto puede romper los esquemas de mucha gente, ya que desafía bastante el modelo tradicional de escalera corporativa, donde hay que crearse un nicho de conocimiento para ir ascendiendo y demás. Si estás en este grupo, párate a pensar si en tu oficina buscas un beneficio personal o el beneficio de tu equipo, y por consiguiente, de tu empresa
* Es divertido: las personas somos seres sociables, nos gusta hablar con la gente, discutir de cosas, gastar bromas. Pair programming fomenta esto al máximo, por lo que la idea de programador aislado con sus cascos la mayor parte del día desaparece por completo, repercutiendo en una mayor salud mental individual :)
* Ayuda a conseguir equipos más cohesionados: en el libro [Peopleware](https://www.amazon.com/Peopleware-Productive-Projects-Teams-3rd/dp/0321934113) se habla de los "jelled teams", equipos que alcanzan velocidad de crucero y se convierten en imparables. Llegar a ese punto no es cosa de un día, y se me ocurren pocas formas más rápidas de lograrlo que el pair programming
* El código generado tiene menos fallos: cuando se trabaja con pair programming se está llevando a cabo una [revisión de código](/2015/03/code-reviews/) a tiempo completo, por lo que el porcentaje de bugs desciende considerablemente.

Seguramente me esté dejando algo en el tintero, pero creo que tras leer todos estos puntos deberiáis tener una buena idea de la experiencia.

##Los contras

Como todo en esta vida, también hay contras, aunque creo que se pueden mitigar en su mayoría:

* Es agotador: las primeras semanas trabajando con pair programming salía de la oficina cansadísimo, y no porque sea un endeble, todo el mundo me comentaba que seguramente estuviera pasando por eso. El hecho de estar todo el día hablando con otra persona, y completamente enfocado en el trabajo (ya que no te puedes distraer tanto como trabajando en solitario), hace que tu mente esté en las últimas al final del día. Sin embargo, como las personas somos tan adaptables, tras tres o cuatro semanas te acostumbras a ese ritmo de trabajo y no es tan duro ni de lejos
* Colisiones de ratón y teclado: en ocasiones, la persona que en teoría no debería tocar el teclado y ratón, no puede evitar hacerlo para comentar una cosa que es complicada de señalar sin tomar el mando. La mayoría de las veces se toma el control por impulso, y puede resultar molesto para el otro, pero es algo inevitable. Cuando pasa, lo mejor es tomárselo con humor si tú eres el interrumpido, y si eres el que interrumpe, intenta avisar al otro educadamente ("¿puedo tomar el ratón un momento, por favor?")
* Buscar algo en internet es algo incómodo: cuando estamos bloqueados con algo y nos vamos a Google a buscar una solución, personalmente siempre he notado que cada persona lee de forma aleatoria diferentes contenidos, así que realmente se están siguiendo dos líneas de pensamiento. Desde cierto punto de vista puede ser beneficioso, ya que quizás se llegue a la solución de forma más rápida, pero personalmente, muchas veces pienso que lo haría mejor en solitario. Cuando esto ocurre, es perfectamente válido proponer la alternativa de investigar en paralelo utilizando dos ordenadores diferentes
* Choques de personalidad: esto es algo inevitable, cada persona es un mundo, y tantas horas juntos lleva a momentos de relativa tensión cuando hay desacuerdos. Esto lo he vivido más veces trabajando con programadores de nuestros clientes que con programadores de nuestra empresa, ya que en nuestro proceso de selección buscamos a gente que se adapte bien al proceso, pero no tenemos este poder con la gente de fuera que viene a trabajar con nosotros. La paciencia es muy importante cuando se trabaja con pair programming, y nunca, nunca, se debe ser maleducado o grosero con el otro
* Desconexión de una mitad: si en un momento determinado notas que estás desconectando de la tareas entre manos, o que le está ocurriendo al otro, lo mejor que se puede hacer es proponer un break. En nuestras oficinas tenemos mesas de ping pong para facilitar esto, y viene muy bien para cambiar radicalmente de actividad durante diez minutos y desconectar

##Conclusiones

No creo que la lectura de este artículo convenza a nadie para introducir pair programming en su día a día, pero sí espero que haya quedado claro por qué es una práctica muy recomendable. Para profundizar más, echad un vistazo al [libro de Kent Beck sobre Extreme Programming](https://www.amazon.es/Extreme-Programming-Explained-Embrace-Embracing/dp/0321278658/ref=sr_1_1).
