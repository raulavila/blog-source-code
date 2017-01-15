---
layout: post
title: Las reuniones en proyectos de desarrollo
description: Mi opinión sobre las reuniones en proyectos de desarrollo software
permalink: 2015/04/reuniones/
tags:
- desarrollo
- gestion
comments: true
---

Vamos a abordar un tema polémico. ¿Son necesarias las reuniones? ¿Tienen un claro ROI ([retorno de la inversión](http://es.wikipedia.org/wiki/Retorno_de_la_inversi%C3%B3n)) o son el mayor ladrón de tiempo para un desarrollador de software? ¿Podemos rechazar alegremente una convocatoria de reunión si la consideramos innecesaria?

<!--break-->

Durante prácticamente toda mi carrera profesional, las reuniones han estado omnipresentes en mi rutina semanal, cuando no diaria. Trabajé bastante tiempo para la administración pública española, y en ese entorno no era extraño encontrarme una vez a la semana con reuniones que duraban 4-5 horas, y que finalizaban porque todos los asistentes estaban hambrientos y cerraba el restaurante (!!).

En los casi doce meses que llevo trabajando en UK, aunque mi empresa está intentando adoptar Scrum, aún perduran prácticas del pasado, y a las tradicionales "Scrum ceremonies" (planning, retrospective, daily scrum, etc) se le siguen sumando montones de reuniones de todo tipo, desde sesiones para decidir la convención a usar para poner nombre a las colas de mensajes hasta largas sesiones en varias entregas para escalar la jerarquía de seguridad de la empresa y conseguir una autorización para conectar con un Web Service externo, pasando por reuniones con el business (product owners) en las que involucran a desarrolladores para decisiones que no son en absoluto de desarrollo.

### ¿Cuál es el coste de una reunión?

El coste de una reunión es muy elevado. Sin concretar cifras a nivel económico, imaginemos un equipo de tres desarrolladores, un arquitecto, un jefe proyecto, un tester y un product owner. Vamos a obviar al Scrum Master, dado que el trabajo asociado con una reunión podría formar parte de las labores explícitamente asociadas al rol.

Bien, tenemos por tanto un equipo de 7 personas. Pongamos que la jornada semanal es de 40 horas, y que es habitual dedicar a reuniones una media de 8 horas semanales (ceremonias de Scrum aparte).

¿Diríamos que el impacto de estas reuniones es de 56 horas en el tiempo total del equipo? En este caso, calculando un total de 280 horas semanales (40 x 7), estaríamos dedicando un 20% del tiempo total.

El problema es que ese tiempo es aún mayor, por varios motivos:

* Las reuniones rompen el ritmo de trabajo, interrumpen tareas en curso, y posteriormente no es inmediato volver al punto en que lo dejamos
* Requieren de una preparación previa, y de una recapitulación posterior
* Son a la misma hora para todo el equipo, y no todo el mundo es igual de productivo a las mismas horas, por lo que afectan en mayor medida a unos sobre otros

Si sumamos el impacto total en el ejemplo que he puesto, diría que esas 8 horas de reunión semanal causan un descenso en la productividad de alrededor del 35% en el trabajo de todo el equipo.

### ¿Cómo minimizar el impacto?

Desde mi humilde punto de vista, el principal problema de las reuniones es que no se filtra adecuadamente a los asistentes. La cuestión es si ese filtro deben realizarlo los encargados de gestionar el proyecto, o ser nosotros mismos, como desarrolladores, los que dejemos constancia de la poca utilidad de nuestra presencia en determinadas sesiones.

La respuesta puede ser que ni unos ni otros, y debería ser una adecuada comunicación dentro del equipo la que lleve a que estas cosas acaben surgiendo de forma natural y "cayendo por su propio peso". Creo que es fundamental ser conscientes del impacto que las reuniones tienen en nuestro tiempo efectivo de trabajo, y me atrevería a afirmar que, pese a que en el pasado llegaba a protestar a mis jefes por el tiempo que perdía en las reuniones nunca llegué a poner sobre la mesa un desglose como el que he realizado en el punto anterior.

Nunca será posible "librarnos" de todas las reuniones, pero realmente ese no es el objetivo, sino utilizar las sesiones conjuntas con el equipo para conseguir objetivos claros, y evitar divagaciones innecesarias. Seguramente sea siempre necesaria la asistencia de un desarrollador, pero no de todos, por lo que una posible estrategia podría ser rotar a los desarrolladores del equipo en reuniones que no afectan a temas de desarrollo.

Otra línea de actuación puede ser establecer agendas claras con los puntos a tratar, y permitir ausentarse a los perfiles que no tengan nada que aportar en algunos de esos puntos. Enfocado al desarrollo, yo siempre ubicaría los temas de desarrollo al principio y una vez terminados ofrecer a los programadores la posibilidad de volver a sus puestos de trabajo.

### Las reuniones en Scrum

Si trabajamos con Scrum, existen una seria de reuniones que no es posible saltarse si realmente queremos obtener el máximo beneficio de esta metodología. Son:

* Sprint planning: se realiza al inicio del sprint y se desglosan las tareas a realizar, estimaciones, etc. Suele ocupar todo un día de trabajo para todo el equipo
* Spring retrospective: se hace retrospectiva de las cosas que fueron bien y mal en el sprint recién finalizado (entre una y dos horas)
* Sprint review: presentación del trabajo realizado en el sprint anterior (una hora)
* Daily stand-up: reunión diaria de corta duración en la que se comentan los progresos realizados por cada miembro del equipo, los impedimentos que se puedan encontrar, etc (quince minutos)

En [este enlace](https://www.atlassian.com/agile/ceremonies) se profundiza más en el cometido de cada reunión. Lo único a tener en cuenta aquí es tratar de atajar excesos en la reunión diaria. En bastantes ocasiones somos dados a profundizar más de la cuenta en las tareas que estamos realizando, o en los problemas que nos hemos encontrado, y entablar discusiones con todo el equipo presente que pueden llevar a prolongar la daily stand-up hasta más de 30 minutos, y en ocasiones hasta una hora. Esto hay que evitarlo a toda costa, en el 95% de las ocasiones es posible posponer la discusión para después de la reunión y comentarla con un miembro determinado del equipo (que puede ser otro desarrollador, el arquitecto, etc).

### Conclusiones

En última instancia, deberíamos recurrir al sentido común para conseguir el equilibrio necesario entre la comunicación y la máxima productividad. El diálogo en sí mismo no es bueno si no es productivo, y por tanto no deberíamos cortarnos a la hora de plantear este tipo de inquietudes a los responsables de gestionar nuestro proyecto (que para eso están las retrospectivas :)).
