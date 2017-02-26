---
layout: post
title: The Senior Software Engineer (el libro)
description: Un resumen del libro The Senior Software Engineer, de David Bryan Copeland
permalink: 2017/02/senior-software-engineer-book/
tags:
- libros
comments: true
---

![SSE](/public/pictures/sse.jpg)

Acabo de terminar el libro ["The Senior Software Engineer: 11 Practices of an Effective Technical Leader"](https://www.amazon.co.uk/Senior-Software-Engineer-Practices-Effective/dp/0990702804/ref=sr_1_1?), escrito por David Bryant Copeland, y me ha gustado tanto que he decidido dedicarle un breve post para animaros a leerlo.

El libro me lo recomendaron en el foro interno de mi empresa, como guía para enfocar correctamente el nuevo rol que me ha tocado este año, pero una vez finalizado, me atrevo a decir que más que una guía para Technical Leaders es un playbook para que un programador con algo de experiencia (pongamos un par de años) sepa encarrilar los siguientes pasos de su carrera.

<!--break-->

## Resultados

El autor comienza hablando de la importancia de entregar resultados en nuestro puesto de trabajo. Esto es algo con lo que me identifico de pleno, creo que se malgasta muchísimo tiempo en la oficina, con conversaciones infructuosas, [reuniones](/2015/04/reuniones/) interminables de las que no se saca ninguna acción en concreto, implementación de nuevas funcionalidades que introducen un montón de bugs y restan más que suman, etc.

Se describe cómo es mucho mejor tardar más tiempo en responder un email con una respuesta concreta que responder rápidamente con un "ya te miro esto". Quizás lo primera cause algo de ansiedad en la persona que nos está preguntando o pidiendo algo, pero tras comprobar la consistencia con la que respondemos de forma clara y concisa pasado un tiempo, nuestra reputación nos empezará a preceder, eliminando este factor.

## TDD

Los siguientes capítulos se centran en el proceso a seguir para solucionar bugs e implementar nuevas funcionalidades en un sistema existente. Para conseguir la máxima efectividad el proceso recomendado no es otro que [Test Driven Development](/2015/08/primera-experiencia-tdd/). La gran diferencia entre solucionar un simple bug y añadir una nueva feature al sistema es el papel de los tests de aceptación en el proceso, que en el caso de los bugs no suelen hacer acto de presencia, aunque yo añadiría que dependiendo del bug podrían hacerlo.

En segundo lugar es muy importante una comunicación constante con las diferentes partes implicadas en nuestro proyecto (stakeholders en inglés). Un canal defectuoso aquí puede tener consecuencias muy negativas en el resultado entregado.

Se hace también hincapié en la importancia de las revisiones de código antes de liberar el código. [Yo hablé de esto](/2015/03/code-reviews/) hace un par de años, aunque en este punto de mi carrera creo que el [Pair Programming](/2016/08/pair-programming/) es más efectivo.

Para terminar, conviene no perder de vista el riesgo de una optimización excesiva (over-engineering) en la fase de refactoring en el proceso TDD. Esto es algo en lo que yo he caído más de una vez. El refactoring debe ser un medio para conseguir nuestros objetivos, y no un fin. En uno de los apéndices se trata de forma explícita este tema de nuevo, bajo el título "Refactoring Responsable".

## Deuda técnica

Se describe la diferencia entre Deuda Técnica y "Código chapucero" (sloppy code), que en mi opinión es bastante importante. Todos sabemos cuando no estamos poniendo el suficiente empeño en generar código mantenible, pero en ocasiones es necesario incurrir en deuda técnica porque es la mejor decisión que se puede tomar en ese momento dadas las circunstancias.

Podéis encontrar cientos de referencias sobre [Deuda Técnica](https://martinfowler.com/bliki/TechnicalDebt.html) en Internet, y conviene que sepáis manejar convenientemente este concepto si os dedicáis profesionalmente al Software. En pocas palabras, deuda técnica es un compromiso que estamos tomando en forma de decisión técnica, para cumplir objetivos a corto plazo, y que sabemos tendremos que arreglar en el futuro a un coste mayor, de la misma forma que un préstamo bancario nos puede sacar de un apuro hoy pero se traducirá en una cantidad mayor de dinero desembolsado a la larga.

La diferencia entre Deuda Técnica y código chapucero, es que para este último hay pocas excusas. En mi artículo de [Ventanas Rotas](/2016/09/ventanas-rotas/) doy un repaso a varios ejemplos de esto.

## Comunicación

La comunicación con nuestros compañeros de trabajo, sean técnicos o no, es muy importante. Para conseguir una comunicación efectiva, la empatía es muy importante, y debemos adaptar nuestro discurso al oyente. Así, que por favor, nunca más le digáis a una persona de negocio algo como:

> Necesitamos refactorizar la capa de persistencia, porque existen problemas de latencia causados por Hibernate. Además, nuestro pool de threads no está correctamente dimensionado

Supongo que sabéis a lo que me refiero. Es necesario además distinguir las diferentes prioridades dentro de cada perfil con el que tenemos que tratar, y buscar un balance entre todas esas prioridades para encontrar una solución de consenso.

## Arrancar un proyecto

El capítulo sobre arrancar un proyecto desde cero es quizás uno de los mejores del libro, y trata los diferentes desafíos que nos encontraremos a la hora de iniciar un "greenfield project":

* Comprender el problema: aquí los "stakeholders" son fundamentales, y el mayor problema que nos vamos a encontrar es que muchas veces no saben exactamente lo que quieren.
* Analizar el nuevo rol de nuestro sistema dentro de la arquitectura global de nuestra empresa: las integraciones más sencillas pueden complicarse sobremanera, y hablo desde mi experiencia personal (una vez nos llevó un mes completo implementar un proceso de login para un aplicación nueva de una empresa financiera, debido a restricciones de seguridad).
* Elegir el stack tecnológico, cosa que puede resultar más complicada de lo que parece, ya que la decisión está condicionada por el conocimiento dentro del equipo, preferencias históricas en nuestra empresa, nivel de familiarización dentro del equipo de operaciones, etc.
* Esbozar una arquitectura a alto nivel del nuevo sistema.
* Crear una versión mínima de la aplicación y desplegarla, lo que el libro [GOOS](https://www.amazon.co.uk/s/ref=nb_sb_ss_i_1_14) define como "Walking Skeleton". En mi opinión, este paso es importantísimo, y hay que intentar tocar el mayor número posible de factores que puedan afectar a nuestra aplicación (Continuous Integration pipeline, third parties, librerías, persistencia, etc).
* Iniciar el desarrollo de las historias de usuario.

Personalmente añadiría la escritura de una versión inicial del fichero README, que no solo sirva de guía para los nuevos desarrolladores que se incorporen al sistema, también como punto de referencia para que nadie pierda de vista la importancia de mantener un fichero README en condiciones. [Jesús L.C. escribió de esto](https://jesuslc.com/2016/07/12/como-escribir-un-readme-que-mole/) hace poco.

## Escribir

Escribir de forma adecuada es imprescindible para avanzar en nuestra carrera, ya que se trata de una habilidad que deberemos utilizar en correos electrónicos, ficheros README, documentación, etc. Hay una sección muy interesante sobre cómo escribir emails de la forma más efectiva posible. En pocas palabras, viene a decir que la gente no suele leer los emails completos (¿os sorprende?), así que no pidáis o preguntéis más de una cosa por mensaje, y tratad de ofrecer alternativas ("¿prefieres la opción A o la opción B?") para que el lector tenga que pararse a pensar el menor tiempo posible. En mi TODO list tengo la escritura de emails como tema pendiente, y volveré a ello en algún momento.

## Entrevistar

Entrevistar a candidatos puede llegar a ser una parte muy importante en nuestro día a día. El libro describe una forma muy definida para realizar un proceso de selección (muy similar al estándar que se utiliza en muchas empresa de Londres, [y que describí aquí](/2015/04/trabajo-londres/)).

Personalmente, añado que entrevistar candidatos es de las labores más complicadas que nos pueden asignar. La demanda en nuestro sector es altísima, y encontrar gente muy buena se convierte en una empresa increíblemente dura. Los mayores dilemas que me he encontrado ocurrieron cuando no tenía claro si la persona era apta o no para el puesto. Para descargar de responsabilidad a los entrevistadores el libro recomienda intentar involucrar a varias personas en el proceso, de forma que todas deben estar de acuerdo en la decisión final. Esto se traduce en que existe "derecho de veto", lo cual puede ser también una carga si tú eres la única persona vetando al candidato :).

## Productividad

En este capítulo se describe un proceso para gestionar las interrupciones, tan presentes en los trabajos de oficina, de una manera efectiva. Recomienda no consultar el correo electrónico impulsivamente, sino hacerlo en momentos previamente decididos, y que encajen bien en el flujo de la metodología TDD, como por ejemplo cuando acabamos de refactorizar una nueva funcionalidad, o acabamos de definir un test fallido.

## Liderar equipos

Este capítulo es el más extenso de todos, y diría que peca un poco de no profundizar demasiado en determinados aspectos de esta labor tan compleja. Se comentan temas como:

* Estructura inicial del equipo: cuando todo es caos, y el mejor enfoque es constituir un equipo pequeño (dos desarrolladores, por ejemplo), y que vaya creciendo según se gane tracción.
* Afrontar el aspecto más problemático dentro del rol de Team Leader, que no es otro que la comunicación de estimaciones a la gente de negocio. En mi opinión, una de las mejores formas (y más divertidas) de aprender como tratar este tema es [viendo esta presentación de Uncle Bob](https://skillsmatter.com/skillscasts/8557-estimation-what-when-why-by-robert-martin) (en la que por cierto, podéis buscarme entre el público :D).
* Marcar un ritmo en el proyecto, mediante breves reuniones semanales donde el equipo presenta su trabajo (en mi empresa esto lo conocemos como [IPM](https://content.pivotal.io/blog/running-an-ipm)).
* Realizar revisiones de código efectivas, de manera que resulte productiva para todos los desarrolladores y no genere rencillas. El principal consejo aquí es comentar el código directamente sin mencionar o dirigirse directamente a la persona que lo escribió (por ejemplo: "este código podría ser más claro si extrayéramos este bloque a un método independiente" vs "este método te ha quedado demasiado largo").

## Producción

Finalmente tendremos nuestro sistema en producción, y en este punto es muy importante que nuestra aplicación genere información de manera adecuada para gestionar de manera correcta incidencias, buscar puntos de mejora, etc. Se describe el proceso para generar logs, alertas y estadísticas, y se hace hincapié en la importancia de escribir código tolerante a fallos.

## Conclusiones

No dejéis pasar este libro, es un compendio excelente de buenas prácticas a seguir para convertirnos en buenos programadores, y además incluye una lista de lectura con libros y artículos que sirven de complemento perfecto para continuar aprendiendo. Me extraña bastante no verlo más a menudo en recopilaciones de libros que todo desarrollador debería leer, y espero que el tiempo le haga justicia (se publicó inicialmente en 2014).
