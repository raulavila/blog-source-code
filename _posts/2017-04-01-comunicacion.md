---
layout: post
title: La comunicación en proyectos Software
description: Cómo comunicar de forma eficiente en proyectos Software
permalink: 2017/04/comunicacion-proyectos-software/
tags:
- comunicacion
- gestion-proyectos
comments: true
---

Esta semana hará tres meses que inicié mi andadura como Anchor en un proyecto de mi empresa. Diría que la experiencia está siendo muy buena en todos los sentidos, y me está sirviendo para aprender un montón de cosas. Quizás la más importante de todas ellas sea como gestionar la comunicación dentro de un equipo de forma eficiente, y en este post voy a repasar muchas de las conclusiones que he ido sacando en claro.

<!--break-->

## Comunicación síncrona

Comunicación síncrona es aquella en la que el intercambio de mensajes e impresiones ocurre en tiempo real, lo que incluye:

* Conversaciones directas en persona
* Conversaciones telefónicas
* Reuniones

### Conversaciones en persona

Es, en mi opinión, el canal más importante de comunicación, y el que debe funcionar con mayor fluidez. Me atrevería a afirmar con rotundidad que ninguna forma de comunicación supera al intercambio de impresiones entre dos personas realmente implicadas en la discusión, y es por eso que seguiré defendiendo a capa y espada el [Pair Programming](/2016/08/pair-programming/). Me resulta gracioso pensar ahora que el trabajar con Pair Programming casi el 100% del tiempo me hizo dudar un poco antes de unirme a una empresa que lo practicaba, mientras que en este momento de mi carrera creo que difícilmente aceptaría una oferta de desarrollador para una empresa donde no se practique.

Fuera del Pair Programming, las conversaciones en persona deben ocurrir siempre que sea necesario, y para ello es fundamental tener al equipo trabajando en el mismo espacio, a poder ser con pizarras compartidas donde se puedan esbozar ideas y diagramas con facilidad. Es importante ser respetuosos con el trabajo de los demás si nos dirigimos a una persona para discutir o preguntar algo, con esto quiero decir que nunca interrumpáis directamente sin antes preguntar con educación "¿Tienes un segundo?" o similar. Si no es el momento adecuado, debemos entenderlo y esperar. La [Técnica Pomodoro](https://es.wikipedia.org/wiki/T%C3%A9cnica_Pomodoro) puede ser realmente útil en este punto, de forma que durante un pomodoro no es posible interrumpir lo que alguien esté haciendo.

También creo que es importante no desviarse demasiado del tema de discusión principal. Ocurre en ocasiones que se empieza tratando un punto A, y en mitad de la conversación saltamos a B, luego a C, y terminamos en F después de una hora. Esta conversaciones no suelen ser excesivamente productivas (¡aunque siempre hay excepciones!), y es importante rectificar el curso si detectamos una pérdida de rumbo excesiva.

### Conversaciones telefónicas

No me extenderé mucho aquí, ya que el uso del teléfono en mi empresa es CERO. Esto es posible porque siempre presionamos para tener al equipo completo en el mismo espacio (como acabo de mencionar). En cualquier caso, por mi experiencia pasada sí tengo algo que aportar a este punto, y es que se deberían evitar tanto como sea posible las llamadas telefónicas "a puerta fría", es decir, sin aviso previo, sobre todo si es para tratar un tema más o menos en profundidad (como por ejemplo realizar un seguimiento por parte de un manager que esté en otra oficina). Este tipo de llamadas tienen un impacto muy grande en el trabajo que el receptor de la llamanda pueda estar realizando, y creo que deberían programarse en el calendario de la misma forma que una reunión presencial de seguimiento.

Otro de los grandes problemas de las conversaciones telefónicas es que son efímeras y no suelen dejar ningún rastro. La memoria humana no es perfecta y es caprichosa, pudiendo crear todo tipo de confusiones.

El único caso donde creo que se pueden justificar las llamadas telefónicas repentinas es cuando está ocurriendo una incidencia grave que requiere atención inmediata.

### Reuniones

[Ya toqué este tema](/2015/04/reuniones/) hace un par de años, pero creo que estoy en posición de añadir nuevas puntualizaciones a lo que entonces dije.

Siendo franco, yo siempre he detestado las reuiones. Opinaba que en la mayoría de los casos no sirven para gran cosa, pero esa perspectiva ha cambiado ligeramente desde que ya no trabajo en desarrollo el 100% del tiempo. Mi punto de vista en la actualidad es que las reuniones son necesarias **mientras sean productivas**.

Empezaré con unas puntualizaciones rápidas sobre las reuniones fijas que suelen existir en casi todos los procesos ágiles:

* Stand-up: debería tener lugar a primera hora de la mañana. Quizás no a las 9 en punto, para no forzar una puntualidad excesiva y el consiguiente estrés asociado en las grandes ciudades, pero definitivamente antes de las 10. Esta reunión debe ser **corta**, así que por favor, evitad dar detalles técnicos innecesarios sobre lo que hicísteis el día anterior, y que dependiendo del tamaño del equipo puede estirar la duración de la stand-up hasta unos 40 minutos (!!!). El principal cometido de la stand-up debería ser compartir a grooso modo el trabajo que estamos haciendo, y lanzar preguntas sobre determinados temas que nos puedan estar bloqueando.
* Reunión de planificación: en mi empresa las iteraciones son de una semana, y la reunión de planificación dura una hora. Las historias de usuario deberían llegar tan maduradas como sea posible en cuanto a la descripción y los criterios de aceptación, de forma que el equipo pueda puntuar el máximo número posible. En mi experiencia, lleva un tiempo agilizar este proceso.
* Retrospectivas: dura otra hora, y repasamos las cosas buenas, malas, y reguleras que han ocurrido durante la semana, centrándonos sobre todo en sentimientos (lo cual representamos con tres columnas: cara sonriente, cara triste, y meh). En esta reunión debemos ser tan abiertos y claros como sea posible, y lo más importante de todo, **debemos extraer acciones a tener en cuenta para la siguiente iteración**.

Todas estas reuniones no deberían llevar más de 3 horas por semana a todo el equipo. El principal problema de muchos proyectos, y donde creo que hay que poner el foco, son las reuniones que caen fuera de este grupo. Intentaré resumir los anti-patrones que, a mi parecer, debemos evitar:

* Demasiada gente convocada: no, no hace falta que todos los desarrolladores estén presentes en reuniones para discutir temas como las proyecciones de futuras fechas de entrega (el típico roadmap), el descubrimiento de nuevos requisitos, la toma de decisiones con la división de seguridad da empresa, etc. El tiempo que los desarrolladores están, ejem, desarrollando, debería maximizarse tanto como sea posible, y para eso existen roles (llámese Team Leader, Anchor, o lo que sea), cuya labor es proteger al equipo de estas conversaciones.
* Reuniones sin decisión de acciones asociada: toda reunión debe terminar con acciones a seguir por una o más personas (que puede incluso ser todo el equipo). Estas decisiones deberían quedar reflejadas en algún sitio (JIRA, Pivotal Tracker, Trello), y no sólo en los cuadernos de los asistentes. En ocasiones, dependiendo de la audiencia, es recomendable enviar un correo a modo de resumen con estas acciones.
* Reuniones de larga duración sin breaks: sinceramente, yo soy incapaz de mantener la atención de manera continuada durante más de 70 minutos. No digo esta cifra al azar, en mi experiencia, a la hora y cuarto no puedo seguir conectado. Los descansos son necesarios, y ni siquiera necesitamos que sean muy largos, 5-10 minutos cada hora, no solo no restan, sino que suman efectividad a la reunión.
* Reuniones sin preparación previa: nunca debemos ir a una reunión sin preparar por encima el tema que se va a tratar. Sin preparación, alguno de los presentes tendrá mucho más contexto que nosotros, tomará la batuta de la discusión, y la reunión avanzará sin discusión real, puesto que no nos atreveremos a rebatir sus argumentos por miedo a quedar en ridículo.
* Reuniones con varias personas en remoto: esto no es fácil de evitar en muchas ocasiones, pero el hecho de tener gente en otra localización diferente (presente al teléfono o Skype) dificulta mucho determinadas conversaciones, además de hacer imposible el uso de la pizarra física si fuera necesaria. Repito, un punto difícil de evitar, pero que conviene mencionar.
* Reuniones que se desvían del tema principal: aspecto que ya comenté en la comunicación entre personas de este mismo artículo, pero que cobra una dimensión incluso mayor en las reuniones. Si nos desvíamos en exceso del tema principal, pueden que gastemos el tiempo de la reunión por completo sin llegar a ningún lado. De ahí la importancia de establecer una agenda clara, y mantener la conversación enfocada.
* Gente con el portátil / móvil en la sala: aspecto peliagudo de evitar, debido al funcionamiento de muchas empresas. El mayor problema que veo a esto es que, quizás debido a las [neuronas espejo](https://es.wikipedia.org/wiki/Neurona_especular), cuando alguien empieza a interactuar con su ordenador o teléfono, siempre hay gente que va detrás :), lo cual nos lleva a la pregunta de si es realmente necesario que esas personas estén presentes en la reunión.
* Personas presentes la reunión entera que sólo son necesarios para un punto en concreto: esta guerra la llevo luchando desde que trabajaba en España hace años. Por aquel entonces, mi empresa se dedicaba a la consultoría medioambiental, y yo era el programador de las webs que desarrollábamos. Pues bien, llegué a tragarme reuniones de 5 horas donde las 4 primeras se centraban en normativas y legislaciones, temas que ni me iban ni me venían, y en la última hora pasábamos a discutir los aspectos de la web. Hay que intentar optimizar el tiempo que la gente pasa en reuniones, y no debería estar mal visto para nada el hecho de que una persona empiece en la reunión y la abandone a los 10-20 minutos de empezar, cosa que, por algún extraño motivo, parece realmente complicado de implementar.
* Timing: las reuniones deben convocarse con un tiempo razonable, para permitir prepararla a los asistentes, y si la reunión pasa a ser innecesaria por algún motivo debe cancelarse tan pronto lo sepamos. La cancelación tardía de reuniones (no es infrecuente recibir cancelaciones una hora o incluso cinco minutos antes de la reunión) puede resultar muy contraproducente para todos los invitados, que quizás hayan planeado su mañana o tarde en torno a ese evento.

## Comunicación asíncrona

A diferencia de la comunicación síncrona, en la asíncrona enviamos un mensaje sin esperar respuesta inmediata. Esta forma de comunicación es quizás menos fluida, pero sin embargo no tiene un impacto tan directo en la productividad, ya que queda en cada persona como manejar estos canales. Las dos herramientas principales dentro de esta categoría son:

* Mensajería instantánea
* Email

### Mensajería instantánea

No hay empresa que no tenga un sistema de este tipo, llámese Lync, Hipchat o Slack, aunque parece que este último está monopolizando esta categoría últimamente.

No tengo gran cosa que aportar en esta categoría, quizás resaltar la necesidad de crear canales específicos dentro de un proyecto determinado, de forma que los desarrolladores no discutan temas técnicos en el canal principal (a la vista de Product Managers y gente de negocio), por ejemplo.

Personalmente, no utilizo la mensajería instantánea muy a menudo, ya que donde realmente es útil es en proyectos donde el equipo está distribuido, lo cual nunca ha sido mi caso.

Un último consejo: todas las empresas tienen acceso al registro de las conversaciones que dejéis en este tipo de sistemas, así que cuidado con las cosas que digáis :D.

### Email

Existen varias consideraciones a tener en cuenta sobre el uso del correo electrónico. La verdad es que este tema daría para un libro, pero intentaré condensar los puntos más importantes:

* No asumas que la gente va a leer tu correo: en efecto, la gente está muy ocupada, recibe muchos correos, y muchas veces no tiene tiempo que dedicarle a ese mensaje que tú crees es de importancia capital. El mayor riesgo en este punto es que un correo se quede marcado como leído en la bandeja de entrada sin que ese hecho haya ocurrido realmente. Para mitigar el riesgo de que tu correo sea ignorado, es crucial intentar ser conciso, el efecto "correo marcado como leído sin leer" es más probable cuanto más largo sea el correo. Otra forma de mitigar esto es marcar en negrita una pregunta o aspecto concreto que sea de importancia capital y/o necesite una respuesta en algún momento.
* La gente reenvía los correos: no creo que os pille por sorpresa, pero lo menciono por si acaso, para que no lo olvidéis. Las consecuencias más nefastas que he experimentado en mi carrera a este respecto es cuando alguien reenvía una conversación kilométrica para que el destinatario eche un vistazo al email más reciente de la cadena, y se da la situación de que en algún punto de la discusión se hablan de temas, digamos "delicados". Es por esto que debemos ser extremadamente cuidadosos con lo que escribimos en nuestros emails, evitando en la medida de lo posible la ironía, el sarcasmo, y menciones a terceros que se salgan de la objetividad y contengan un componente emocional.
* Si tienes tres preguntas que hacer, envía tres correos: los correos son leídos normalmente a gran velocidad, y si el correo contiene varias preguntas es probable que no todas sean respondidas (si es que alguna es respondida). Por ello debemos centrarnos en un aspecto muy concreto del que necesitemos aclaración en cada correo, y si queremos incrementar la probabilidad de respuesta, podemos marcarlo en negrita. Una vez leí en un artículo que la mejor forma de incrementar las probabilidades de que el destinatario de un correo responda a una petición es finalizando con "Gracias por adelantado". No estoy en posición de defender la fiabilidad de esto, pero ahí lo dejo :).
* Relee y relee antes de escribir, a poder ser en voz alta, o mejor, pide a otra persona que lo relea contigo. Esto ayuda mucho a detectar frases poco claras, falta de información, o a corregir el tono incorrecto en algún punto tratado.
* Si necesitamos adjuntar un documento importante que pueda evolucionar en el tiempo, plantearos la necesidad de poner ese documento en un espacio compartido (Confluence, Google Drive...), de forma que no se generen confusiones con diferentes versiones circulando. Si no queda otra que adjuntarlo, añadid un número de versión, y madurad bien el documento antes de pulsar "Send", quizás dejando el correo escrito una tarde y revisándolo en la mañana siguiente antes de enviar.
* El correo electrónico no es un chat, nunca esperes que la gente responda inmediatamente. Evita en la medida de lo posible el envío de correos para preguntar si un correo anterior ha sido leído, y de hacerlo, deja pasar un tiempo prudencial (no menos de una semana / 10 días).
* Nunca dejes un tema de gran importancia a expensas de recibir respuesta a un correo, utiliza otros canales para esto.

Con esto termino mi repaso a las diferentes formas de comunicación en proyectos de empresa. Si pensáis que me he dejado alguna, o si creéis necesario puntualizar algún aspecto de los tratados, estaré encantado de seguir discutiendo en los comentarios.
