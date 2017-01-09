---
layout: post
title: Code Reviews (revisiones de código)
permalink: 2015/03/code-reviews/
tags:
- desarrollo
- buenas-practicas
- agile
comments: true
---

¿Son necesarias las revisiones de código? ¿No revisamos ya el código al mismo tiempo que desarrollamos? Si ya utilizo el refactoring como práctica habitual, ¿qué sentido tiene revisar el código de nuevo?

Son muchas las preguntas que pueden surgir en un equipo de desarrolladores en referencia a las revisiones de código. Pero peor aún que las preguntas son las inquietudes o recelos ("¡si se revisa mi código tirarán por tierra mi trabajo!"). Antes de nada, empezaremos definiendo qué es una revisión de código (*Code review*).

<!--break-->

## Definición de Code Review

Partamos de un escenario en el que trabajamos en un [marco ágil](http://es.wikipedia.org/wiki/Desarrollo_%C3%A1gil_de_software), y dentro de las diferentes opciones, elijamos [Scrum](http://es.wikipedia.org/wiki/Scrum). No es el objetivo de este post profundizar en el funcionamiento de estas metodologías. Aunque es posible que en un futuro hable más sobre ellas, Internet está plagado de información al respecto; yo concretamente recomiendo el blog de [Javier Garzás](http://javiergarzas.com/).

Es necesario establecer un marco inicial para ubicar la forma en que afrontamos nuestro desarrollo de software. En Scrum la "unidad de desarrollo" es la [historia de usuario](http://es.wikipedia.org/wiki/Historias_de_usuario), y la unidad de tiempo el [sprint](http://searchsoftwarequality.techtarget.com/definition/Scrum-sprint).

Imaginemos pues, que un equipo de 4 desarrolladores tiene que repartirse el desarrollo de varias historias de usuario a lo largo de un sprint. Imaginemos que a un desarrollador en solitario (llamémosle Luis) se le asigna la historia de usuario X (por ejemplo, "Creación de capa de persistencia de datos").

Una revisión de código es el proceso mediante el cual, el resto del equipo revisa en conjunto la implementación de Luis para esa historia de usuario, aportando ideas sobre como mejorarlo, quizás refactorizarlo, descubriendo posibles bugs, errores de arquitectura, falta de cobertura de tests para determinados casos, y un sinfín de cosas más que puedan surgir.

#### Reticencias...

Seamos directos: en nuestra profesión abundan los perfiles "complicados". Sin entrar en demasiados detalles, determinados desarrolladores no están abiertos a crítica, y mucho menos a modificar un código que han finalizado y está, en teoría, funcionando. Si alguno de los lectores pertenece a este grupo yo le lanzaría las siguientes preguntas:

* ¿Tú código es fácil de entender por otra persona que no seas tú?
* ¿Tú mismo serías capaz de entender ese código de aquí a un par de años?
* ¿Has tenido en cuenta TODOS los factores que afectan o pueden afectar al sistema a la hora de desarrollar tu código?
* ¿Si tú descubrieras un error flagrante en el código de otro no crees que es tu responsabilidad como profesional arreglarlo? En tal caso, ¿no es mejor detectar ese tipo de errores cuanto antes?

#### Colectividad del codigo. El código no es mío

Efectivamente, cuando trabajamos en un proyecto empresarial, estamos programando código que seguirá siendo desarrollado, mejorado y mantenido por un buen puñado de desarrolladores en el futuro. Creo que en semejante ámbito no tiene ningún sentido hablar de propiedad individual del código, más bien todo lo contrario.

La propiedad colectiva del código fomenta que un equipo ágil asuma la responsabilidad del funcionamiento de absolutamente todo el sistema entregado. Nada de apuntar con el dedo a culpables cuando algo no funciona, o sugerir que "yo no he sido" o "yo no estaba allí". Diría que algo así es evidente...pero no lo es en absoluto.

La experiencia me ha enseñado que las revisiones de código son la manera más efectiva de alcanzar ese ideal.

## El proceso

El proceso de una revisión de código comienza cuando un desarrollador va a subir, o ya ha subido, al repositorio central (damos por hecho que estamos utilizando Git o herramienta de control de versiones similar) todos los cambios asociados con la implementación de la historia de usuario asignada.

El resto del equipo (en nuestro ejemplo, los otros tres desarrolladores), o más de un 50% restante al menos (en el ejemplo, un mínimo de 2) repasarán todos los cambios en el código que serán añadidos al sistema, y mediante comentarios en un entorno amistoso e informal transmitirán sus dudas respecto a ciertas decisiones, propuestas de mejora, y por supuesto "thumbs up" ante ideas que les puedan resultar brillantes o especialmente acertadas. Estos comentarios deberán ir enfocados a cubrir varias de las cuestiones que he ido plateando en este post, y los beneficios son varios y evidentes:

1. Descubrir herramientas, API's, maneras de afrontar un problema...que desconocíamos
2. Obtener un conocimiento global del sistema (no sólo de las partes que YO desarrollo)
3. Aportar puntos de vista externos, y sacar a relucir cuestiones que se le pueden haber pasado por alto al desarrollador principal
4. Mejorar la calidad y legibilidad. En ocasiones el funcionamiento de un fragmento de código que puede resultar muy evidente a su creador no lo es tanto para otros
5. Conseguir la tan ansiada "colectividad del código"

Por supuesto, no todos los comentarios del resto del equipo pueden resultar acertados, y es tarea de **todo el equipo** alcanzar un consenso sobre las acciones que hay que llevar a cabo una vez finalizada la revisión. En general, siempre surgirán cambios a aplicar. En mi actual empresa damos por hecho que tras la "Revisión de código" siempre habrá una tarea de "Cambios tras revisión de código", y siempre añadimos una estimación de estas tareas al conjunto de la historia de usuario. De hecho, nunca damos por cerrada (**done**) una historia de usuario hasta que ha pasado por la revisión de código.

Conviene dejar los egos aparcados a la hora de realizar una revisión de código. No estamos en un concurso para encontrar al programador más brillante, sólo buscamos que nuestro producto sea de la máxima calidad.

## El resultado

Me gusta mucho esta fórmula (la descubrí en un vídeo de Atlassian que no he conseguido localizar de nuevo):

![Formula Code Review](/public/pictures/code_review_formula.png)

* G: nivel de culpabilidad individual de un bug encontrado en producción
* R: número de personas que revisaron el código afectado

Deducimos por tanto que si un código no es revisado por nadie, **solamente una persona** será responsable de un fallo en producción. Mientras que si todo el equipo revisa el código, la culpabilidad se reparte entre todo el equipo. ¿No hemos conseguido esa "propiedad colectiva del código" buscada?

## Herramientas

La revisión de código puede hacerse reuniendo al equipo en una sala o utilizando una herramienta creada con ese cometido. Existen varias en el mercado, tanto Open Source como comerciales, para conducir Code Reviews. En [este link](http://en.wikipedia.org/wiki/List_of_tools_for_code_review) se listan varias de ellas. Personalmente he trabajado con [Crucible](https://www.atlassian.com/software/crucible/overview) de [Atlassian](https://www.atlassian.com/), que se integra estupendamente con el resto de la suite Atlassian (JIRA para gestión de tareas, Stash como repositorio de código Git, etc), y permite iniciar conversaciones a partir de una línea de código, bloque, añadir comentarios generales, etc:

![Crucible](/public/pictures/code-review-hero.png)

<div class="font-small" style="text-align:center;">
(imagen descargada de la propia web de Atlassian)
</div>

## Mi experiencia personal

Me atrevería afirmar que soy mucho mejor profesional desde que formo parte de un equipo que utiliza de forma constante las revisiones de código. No solamente por todo lo que se aprende cuando tu código es puesto a prueba o cuando te toca revisar el código de otros, haciéndote desarrollar un espíritu crítico y analítico que no utilizaríamos de forma tan clara en otras circunstancias.

Hay otro factor más "incómodo" de decir, y es que, sabiendo de antemano que tu código va a ser revisado programamos mejor :). Ciertos vicios o prácticas adquiridas, que sabemos fehacientemente que no están bien, pero que da algo de pereza esquivar son definitivamente arrinconados cuando tenemos una revisión de código en el horizonte.

## Conclusiones

Ayer mismo terminé un proyecto en el que he estado trabajando durante 11 meses (de hecho hoy estoy iniciando mis vacaciones :)), habiendo completado 75 revisiones de código a lo largo de este tiempo. Creo que es, con diferencia, el producto de mayor calidad que he entregada hasta ahora, y pondría totalmente la mano en el fuego por él. Seguro que el resto de desarrolladores del proyecto piensan exactamente lo mismo que yo. Sin "Code Reviews" jamás habríamos conseguido estándares tan altos ni por asomo.

Dadle una oportunidad a las revisiones de código, nunca volveréis atrás.
