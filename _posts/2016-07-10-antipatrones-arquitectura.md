---
layout: post
title: Antipatrones de Arquitectura Software
description: Antipatrones de Arquitectura Software
permalink: 2016/07/antipatrones-arquitectura/
tags:
- desarrollo
- arquitectura
- antipatrones
comments: true
---

Los patrones, sean [de diseño](https://es.wikipedia.org/wiki/Patr%C3%B3n_de_dise%C3%B1o), [de arquitectura](https://www.amazon.com/Pattern-Oriented-Software-Architecture-System-Patterns/dp/0471958697), etc, son una herramienta fundamental a la hora de desarrollar software. Pero en ocasiones, al mismo nivel de importancia encontramos los antipatrones, que no son más que prácticas equivocadas que a primera vista pueden parecer lo contrario.

Estos antipatrones surgen de la experiencia, y tenerlos presentes puede ser extremadamente útil para evitar perder tiempo innecesario debido a decisiones que otros han demostrado son fallidas. En este post pasaré revista a una serie de antipatrones de arquitectura software, o en otras palabras, antipatrones que aparecen en fases de definición y toma de decisiones para un nuevo producto o proyecto a desarrollar. Todos ellos están descritos más en profundidad en [este curso](http://shop.oreilly.com/product/110000195.do). Utilizaré su nombre original, en inglés, ya que generalmente es complicado encontrar traducciones que les hagan justicia y lleguen a ser acuñadas.

<!--break-->

### Architecture by implication

Siendo breves, esto no es otra cosa que tener desarrollado un sistema sin una documentación de su arquitectura. En estos tiempos donde las metodologías ágiles parecen coparlo todo, muchas empresas interpretan de forma errónea uno de los puntos más importantes del [Agile Manifesto](http://www.agilemanifesto.org/), que dice:

> Working software over comprehensive documentation

Es importante destacar que aquí no dice "Don't write documentation, write software instead". La clave de este punto es que hay que favorecer el desarrollo sobre la documentación, pero **sin olvidarse del todo** de dicha documentación, que por otro lado, debe ser sencilla y asequible para todas las partes implicadas en el desarrollo de un sistema.

Recuerdo que en mi primer trabajo como desarrollador, la documentación era tan detallada que los desarrolladores nos encontrábamos con especificaciones de este estilo:

{% highlight text %}
1. Leer el campo A de la tabla T
2. Actualizar el campo B de la tabla S con el valor obtenido del campo anterior
{% endhighlight %}

No voy a entrar a valorar la conveniencia de este tipo de documentación (que para empezar, ¡obliga a duplicar el trabajo utilizando diferentes medios de expresión!), pero el caso es que parece ser que al ser humano le gustan los extremos, y ahora se estila no tener NADA de documentación. Pues bien, ni una cosa ni la otra.

Cuando un sistema crece con el tiempo, es importante poder obtener una visión global de manera sencilla y rápida sin necesidad de leer el código. En un libro que recomiendo encarecidamente, ["Software Architecture For Developers"](https://leanpub.com/software-architecture-for-developers), [Simon Brown](https://twitter.com/simonbrown) nos presenta un modelo bautizado como [C4 Model](http://www.codingthearchitecture.com/2014/08/24/c4_model_poster.html) que mediante una serie de diagramas a cuatro niveles genera precisamente esta visión global. Os animo a profundizar en este modelo, que yo he encontrado extremadamente intuitivo y útil.

Por último, esta documentación básica debería contener información crucial para nuestro sistema como:

* Intregraciones con otros componentes
* Requisitos de rendimiento y mantenibilidad
* ¿Es nuestra solución factible dentro de nuestros límites de tiempo y presupuesto?
* Requisitos de seguridad
* Previsiones de escalabilidad

### Witches brew

Consiste en juntar a tantísimos roles (stakeholders) en la definición de la arquitectura, que el resultado final es una mezcla de ideas sin mucho sentido ni visión clara. Esto conducirá a lo que se conoce como "Analysis Paralysis", que bloquea el desarrollo de forma inevitable.

Para remediar esta situación se necesita, en primera lugar, una clara estructura de equipo, donde los ingenieros puedan tomar la última palabra (y para que esto ocurra dichos ingenieros deben tener un bagaje suficiente que les haga hacerse respetar). Además, las primeras fases de desarrollo deberán centrarse en el descubrimiento técnico, con un análisis conveniente de los diferentes pros y contras de cada decisión tomada, así como una revisión conjunta de dichas decisiones (peer reviews).

### Gold plating

Este es un problema que se da tanto en fases de arquitectura como en fases de desarrollo propiamente dicho. En toda tarea a realizar llega un momento en que añadir mayor esfuerzo no genera una cantidad de valor razonablemente acorde con el esfuerzo realizado.

Desde el punto de vista de la arquitectura, esto puede ocurrir si se comienzan a añadir demasiados detalles a los documentos de definición (ver ejemplo en el primer antipatrón :)). Tanto detalle además, puede llevar a ocultar los principios y estándares clave que se quieren comunicar, llevando, de nuevo, a un estado de parálisis.

Desde el punto de vista de la implementación, gold plating se da cuando comenzamos a refactorizar más de la cuenta, generando un diseño extremadamente flexible, pero que posiblemente no nos sea de demasiada utilidad (también conocido como [YAGNI](https://es.wikipedia.org/wiki/YAGNI)).

### Vendor King (o Vendor Locking)

Creo que por su nombre está claro de qué se trata. Tenemos un producto dependiendo de otro que condiciona enormemente nuestro diseño. Esto puede ocurrir si acoplamos nuestra solución a APIs propietarias, por ejemplo, de forma que si cambian en el futuro tendríamos que modificar un número enorme de líneas de código para adaptarse a la nueva versión. Por no hablar de que el producto desaparezca del mercado en favor de otras opciones.

La solución a este problema es utilizar patrones de diseño como [Adapter](https://es.wikipedia.org/wiki/Adapter_(patr%C3%B3n_de_dise%C3%B1o)), para desacoplar al máximo nuestra aplicación del producto externo. Otra solución puede ser el uso de colas de mensajes como punto de conexión entre ambos sistemas.

### Big Bang Architecture

Consiste en definir todos los detalles de la arquitectura al principio del proyecto ([Big Design Up Front](https://en.wikipedia.org/wiki/Big_Design_Up_Front)), que es el momento en que menos información tenemos. En estos días frenéticos que vivimos, los requisitos y las necesidades cambian constantemente, por lo que lo correcto es definir únicamente la arquitectura básica adecuada para arrancar el desarrollo, e ir refinando sucesivamente. Evidentemente, para que nuestro proyecto concluya de forma exitosa será necesario que diseñemos una arquitectura flexible y adaptable al cambio ([Architecting for Change](http://www.wmrichards.com/all-things-architecture/architecting-for-change.html)), así como respetar al máximo buenas prácticas de implementación (Clean Code, Testing, etc).

### Armchair Architecture

Esto ocurre cuando bocetos dibujados en fases iniciales pasan al equipo de desarrollo para ser implementados sin validación ninguna. En ocasiones, estos bocetos son incluso ¡fotografías tomadas a pizarras en salas de reuniones!

Yo he vivido esto en más de una ocasión. Para solventar el problema de las fotos está claro que es necesario establecer un proceso claro de normalización de estos bocetos en lugar de subir las fotos tal cual a nuestra Wiki. Jamás un boceto realizado en una reunión puede considerarse como el camino definitivo a seguir, ya que, con frecuencia, las reuniones llevan a confusiones debidas a fallos comunicativos que son descubiertas tan pronto como una persona revisa el resultado generado en dicha reunión.

En otras ocasiones, la causa de este problema es la contratación de arquitectos externos, que entregan su trabajo y desaparecen. Por fortuna, cada vez está más claro que este modelo no funciona, y que los equipos deben estar implicados de principio a fin en todas las fases de desarrollo. Pero si esto no ocurre en vuestro lugar de trabajo, por favor, tratad de remediarlo cuanto antes :)

### Playing With New Toys

Incorporar nuevas tecnologías a nuestro sistema sin verificar la conveniencia de su uso. Antes de añadir cualquier framework, librería o componente, deberíamos responder las siguientes preguntas:

* ¿Qué valor añade?
* ¿Es una tecnología probada y contrastada para el problema que tratamos de resolver?
* ¿Somos de los primeros en utilizar esta tecnología?
* ¿Existe soporte suficiente en la comunidad (StackOverflow...)?
* ¿Existe ya algo en nuestro sistema que proporciona esta funcionalidad o parecida?
* ¿Tenemos las habilidades necesaria en nuestro equipo, o deberemos considerar un tiempo adicional de aprendizaje?

Los desarrolladores somos curiosos por naturaleza, pero jamás debemos caer en el conocido como "CV Driven Development", es decir, añadir tecnologías a nuestros proyectos únicamente para poder ponerlas en nuestro CV.

### Spider Web Architecture

En estos días, la arquitectura de [Microservicios](http://www.javiergarzas.com/2015/06/microservicios.html) está omnipresente en cualquier conversación sobre software que se precie, pero no es la panacea. Tiene muchas ventajas, pero también grandes inconvenientes, empezando por el antipatrón ["Distributed Monolith"](https://www.infoq.com/news/2016/02/services-distributed-monolith) .

"Spider Web Architecture" no es más que una arquitectura plagada de servicios distribuidos, donde no nos hemos parado a pensar en todos los problemas que esto va a generar, empezando por el mantenimiento de cada uno de ellos por separado. OK, los microservicios están bien, pero pensad bien si es lo que necesitáis cuando vayáis por esa vía.

### Infinity Architecture

Consiste en crear arquitecturas extremadamente flexibles, para soportar requerimientos que seguramente tendremos en el futuro. El problema es que muchísimas veces estos requerimientos no llegan a materializarse, y nos vemos manejando unos niveles de abstracción que complican cualquier mínimo cambio o la solución de bugs.

### Groundhog Day

O "El día de la marmota", ocurre cuando las decisiones de arquitectura tomadas no son comunicadas de forma efectiva, lo que lleva a discutirlas de nuevo en futuras reuniones o en nuestras mesas de trabajo. Para evitar esto debemos centralizar todas estas decisiones en una Wiki o similar, de manera que ante la duda podamos remitir a nuestro interlocutor a ese lugar para finiquitar cualquier discusión innecesaria.

## Conclusión

Las fases de arquitectura y diseño son extremadamente complejas dentro del ciclo de vida del desarrollo software. El sentido común debe dictar cualquier decisión tomada, y muchos de estos patrones no son más que sentido común con un nombre vistoso, pero no está de más tenerlos presentes, ¿verdad?
