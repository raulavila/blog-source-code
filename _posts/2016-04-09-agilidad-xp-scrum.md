---
layout: post
title: Agilidad, XP y Scrum
permalink: 2016/04/agilidad-xp-scrum/
tags:
- desarrollo
- metdologías
comments: true
---

En los últimos años han cobrado relativa popularidad las conocidas como [metdologías ágiles](http://blog.leanmonitor.com/es/que-son-las-metodologias-agiles/), tanta que se llegan a ver ofertas de trabajo buscando "Desarrollador ágil", "Agile Java Developer" y similares. Cuando inicié mi [búsqueda de trabajo en Londres](/2015/04/trabajo-londres/) hace un par de años me sorprendió lo importante que todas las empresas consideraban tener experiencia "agile". Yo ya conocía algo, pues en mi última empresa española utilizábamos Scrum, así que quizás eso me facilitó encontrar mi primer puesto de trabajo en UK.

Tras mi reciente cambio de empresa a una que es [verdaderamente ágil](http://pivotal.io/) he tenido finalmente constancia de lo que supone esa palabra, por lo que finalmente creo que me veo en disposición de dar mi punto de vista sobre todo esto.

<!--break-->

##Scrum

Scrum es la metodología ágil más popular con bastante diferencia, tanto que ha dado lugar a un nuevo rol en el mundo empresarial conocido como "Scrum Master". No es mi intención explicarla en profundidad, para ello hay [varios sitios](https://proyectosagiles.org/que-es-scrum/), y [montones de referencias](http://scrumreferencecard.com/) . Es tal el alcance de Scrum, que existe una organización, [Scrum Alliance](https://www.scrumalliance.org), que ofrece (por un "módico" precio) varias [certificaciones](https://www.scrumalliance.org/certifications).

Personalmente, os recomendaría la lectura del libro "Scrum and XP from the trenches" (que está accesible de [forma pública](http://wwwis.win.tue.nl/2R690/doc/ScrumAndXpFromTheTrenchesonline07-31.pdf)), como una buena introducción a este framework. A un nivel más avanzado, la serie de artículos de Tobias Mayer recopilados en el libro [Por Un Scrum Popular](https://www.amazon.es/Por-Un-Scrum-Popular-Revolucion/dp/1937965228/ref=sr_1_1) es una forma ideal de comprender las ambiciones de Scrum, y lo complicado que es llevarlas a cabo.

En mi opinión, el principal defecto de Scrum es que se trata de un marco tan definido, con roles (Scrum Master, Product Owner, Developer), ceremonias (Standup, Retrospectiva, Planning, Review), y formalismos (Tablón, Historias de Usuario, Tareas, Sprints), que la mayor parte de las compañías, que llevan funcionado durante décadas de una forma totalmente tradicional y siguiendo el modelo de [Desarrollo en Cascada](https://es.wikipedia.org/wiki/Desarrollo_en_cascada), piensan que adoptando la jerga de Scrum, pero sin transformación interna real, se convierten por obra y gracia de dios en empresas ágiles. El libro de Mayer habla muchísimo de este problema (y también de cómo Scrum se ha ido convirtiendo en un negocio debido a las certificaciones).

Para transformar una empresa en ágil, se necesita mucho más que simular que se trabaja por iteraciones y poner post-its en las paredes. Se necesita un conjunto concretos de valores, principios y prácticas, que deben adoptarse de forma paulatina hasta desembocar en una forma de trabajar totalmente diferente.

##Extreme Programming

Extreme Programming (XP) es ese conjunto de valores, principios y prácticas que acabo de mencionar. Podríamos llamarlo metodología si queréis, pero es mucho más que eso.

Fue Kent Beck quien acuñó el término y le dió forma en el libro [Extreme Programming Explained - Embrace Change](https://www.amazon.es/Extreme-Programming-Explained-Embrace-Embracing/dp/0321278658/ref=sr_1_1). La primera versión del libro fue tan rupturista que cinco años después publicó una segunda algo suavizada :). Yo acabo de leerlo como parte de mi inducción en la nueva empresa, a la vez que estoy experimentando el día a día de una oficina que sabe realmente lo que significa ser ágil, y la verdad es que está siendo muy satisfactorio comprobar de primera mano lo que supone ser "agile".

Os recomiendo encarecidamente el libro, pero a modo de introducción, XP define un marco de trabajo donde la honestidad, el respeto y la comunicación es lo primero, donde el equipo es una entidad muchísimo más importante que el individuo, donde el conocimiento es compartido siempre, y donde el feedback es una constante que permite llevar a cabo una mejora permanente.

Para alcanzar estos ideales, XP propone una serie de prácticas que son ley y han de utilizarse. Se trata de:

* Pair Programming: se trabaja en parejas, todo el tiempo, todos los días. Esto, que a priori puede parecer una locura y una forma de desperdiciar recursos, es lo que en última instancia genera ese ambiente de honestidad, conocimiento compartido, respeto y comunicación que mencionaba más arriba. En el futuro hablaré más largo y tendido sobre mi experiencia con el Pair Programming
* Test Driven Development: el TDD no es nada nuevo en [este blog](/2016/01/aprendiendo-TDD/). Es el proceso de diseño en el que los tests guían la implementación de nuestro sistema, y genera unos resultados con un número de defectos bajísimo
* Diseño incremental: nuestro software no es diseñado en profundidad en las primeras fases del desarrollo. Es el día al día de la codificación quien, paso a paso, mediante incrementos pequeños, hacer emerger el diseño de nuestro sistema
* Integración Continua (Continuous Integration): cada pequeño incremento en nuestro código es subido al repositorio de control de versiones e integrado en el pipeline de integración continua para comprobar que en todo momento tenemos una versión operativa y desplegable de nuestro Software. Esto quiere decir, que el despliegue de una nueva versión en producción será una decisión de negocio y no tecnológica, porque en todo momento tendremos una versión válida desplegable con el trabajo más reciente
* 40 horas de trabajo a la semana: es muy importante no trabajar más de 40 horas a la semana. Pero ojo, 40 horas de trabajo real son muchas. Uncle Bob cambió el enfoque con una nueva denominación "8 hour burn", que viene a decir algo así como "trabaja durante 8 horas al día tan intensamente que no seas capaz de seguir trabajando más allá". Esto quiere decir que no es válido consultar las noticias, el Twitter o el correo personal durante ese tiempo (cosa que, por otra parte, ¡es imposible si trabajas en pareja!)

Todas estas prácticas, adoptadas de forma incremental, llevan a trabajar de una forma completamente diferente, y que una vez experimentada te hacen no querer volver atrás. Lo que más me ha llamado la atención, en estos dos meses que llevo trabajando con XP, es la confianza total que los managers tienen en el equipo, en sus estimaciones y en sus entregas.

##Scrum vs XP

Está muy claro con qué me quedo. Mi experiencia con Scrum no ha sido demasiado buena en general, por los motivos expuestos anteriormente. No digo que Scrum, bien utilizado, no pueda dar buenos resultados, pero, la verdad, para utilizarlo realmente bien habría que aplicar todas las prácticas de XP, por lo que al final tendriamos algo así como Scrum-Xp. De hecho, es curioso como el libro [Scrum and XP from the trenches](http://wwwis.win.tue.nl/2R690/doc/ScrumAndXpFromTheTrenchesonline07-31.pdf), que vuelvo a recomendar y puede considerarse una referencia fundamental de la comunidad ágil menciona ambas metdologías en el título.
