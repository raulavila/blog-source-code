---
layout: post
title: Aprendiendo TDD
permalink: 2016/01/aprendiendo-TDD/
tags:
- TDD
- desarrollo
comments: true
---

Retomo el blog tras más de dos meses de parón. Los motivos han sido dos: mi búsqueda personal de un nuevo trabajo en Londres ([ya escribí acerca de lo exigente que puede llegar a ser esto](/2015/04/trabajo-londres)), y las merecidas vacaciones navideñas.

Excusas aparte, aquí estamos de nuevo, y el motivo principal de este post será recopilar diferentes recursos que he ido utilizando en el último medio año para profundizar en mi viaje personal dentro del TDD ([viaje que inicié aquí](/2015/08/primera-experiencia-tdd)). Dicho viaje tendrá su culminación en mi nuevo puesto de trabajo, donde utilizaré esta metdología a diario, ¡así que no creo que sea la última vez que hablo de TDD en el blog!

<!--break-->

En estos tiempos de información a espuertas, los recursos para aprender cualquier cosa son casi infinitos. Esto es bueno y malo, bueno porque ya no es necesario dejarse los cuernos buscando ese libro tan popular en las librerías (me estoy remontando a los noventa, dejando claro lo viejuno que soy :)). Pero también tiene su contrapunto negativo, y es que a veces es muy difícil seleccionar exactamente qué fuentes utilizar para sacar el máximo rendimiento de nuestro limitado tiempo.

Por eso mismo he decidido pasar revista a los recursos que personalmente he utilizado, y diría me han servido estupendamente para tener una buena base con el TDD. Algunos de ellos ya los mencioné [en mi primer post sobre el tema](/2015/08/primera-experiencia-tdd), pero creo que es ahora, medio año después, cuando realmente me veo con la confianza suficiente para describir un camino de aprendizaje que os dejará en posición de aplicar TDD en vuestro día a día una vez lo hayáis finalizado.

### Primer paso: El libro de Kent Beck

[Test Driven Development by Example](http://www.amazon.es/Driven-Development-Example-Addison-Wesley-Signature/dp/0321146530/ref=sr_1_1?ie=UTF8&qid=1438434705&sr=8-1&keywords=Kent+Beck) es considerado la obra fundamental por los "TDD practicioners". Tras haberla leído, diría que el libro se queda un poco a medias, pero su primera parte es **imprescindible**. Es en esta primera parte donde se desarrolla paso a paso un sistema específico (para realizar operaciones monetarias, básicamente) mediante el famoso proceso "Red-Green-Refactor". Este sería el primer ejemplo que yo recomendaría implementar a un completo novato en la materia para "romper el cascarón". Si os interesa echar un vistazo, [aquí está mi implementación](https://github.com/raulavila/tdd-kent-beck), aunque no os servirá de nada si no lo hacéis por vosotros mismos.

La segunda parte del libro peca un poco de ser algo teórica. Dejando de lado un segundo ejemplo donde se implementa un framework xUnit utilizando TDD (algo así como meta-TDD), y que personalmente creo que es algo confuso y no aporta conocimiento "de trincheras", el libro entra en varios capítulos donde se describen patrones de diseño y refactorings de forma demasiado teórica (sin código de por medio). A mi parecer, es complicado seguir algunas explicaciones sin ejemplos claros. Es algo que me costó a mí, incluso conociendo casi todos los patrones y refactorings (varios de los refactorings provienen del [libro de Fowler](/2015/02/fowler-refactoring-1)), así que imagino será mucho más confuso para programadores más noveles.

El último capítulo ("Mastering TDD") recupera el pulso, con un formato pregunta-respuesta donde quedan bien claro los beneficios de esta metodología, que en el momento de escribir el libro (hace más de 12 años) era totalmente desconocida.

En resumen, haceros con una copia de este libro y centraros en los capítulos "The Money Example" y "Mastering TDD" para tener una buena base de cara a los siguientes pasos.

### Segundo paso: la presentación de Sandro Mancuso

[Sandro Mancuso](https://twitter.com/sandromancuso) es uno de los líderes del movimiento [Software Craftmanship](http://manifesto.softwarecraftsmanship.org/) y fundador de la empresa [Codurance](http://codurance.com/), que aboga por el desarrollo basado en buenas prácticas (TDD, pair programming, diseño incremental, etc).

En una serie de tres vídeos, que juntos apenas sobrepasan la hora y media de duración, Mancuso desarrolla un pequeño sistema para gestionar una cuenta bancaria partiendo de una suite de tests de aceptación (acceptance tests), y añadiendo de forma incremental tests unitarios según lo va necesitando para guiar el desarrollo de cada componente.

Me parece asombroso lo precisa y clara que resulta esta presentación en un tiempo tan reducido. Enlazo a continuación las diferentes partes:

* [Parte 1](https://www.youtube.com/watch?v=XHnuMjah6ps)
* [Parte 2](https://www.youtube.com/watch?v=gs0rqDdz3ko)
* [Parte 3](https://www.youtube.com/watch?v=R9OAt9AOrzI)

### Tercer paso (y definitivo): World's best intro to TDD

[J.B. Rainsberger](https://twitter.com/jbrains) es otra de las figuras fundamentales en la actualidad cuando hablamos de Agile, TDD y Coaching en general. [Su web](http://www.jbrains.ca/) está plagada de recursos de todo tipo (además de ofrecer sus servicios :)), pero la piedra filosofal es el curso [World's Best Intro To TDD](http://online-training.jbrains.ca/courses/wbitdd-01). Su "modesto" nombre hace perfecta justicia a su contenido, y su precio (97 dólares) me parece barato para lo que ofrece.

En unas 25 horas de vídeo, Rainsbergerg desarrolla diferentes sistemas desde cero utilizando TDD. El primero de ellos es bastante sencillo, una calculadora de fracciones, pero me parece una perfecta introducción a lo que se avecina.

En la segunda parte del curso implementa un sistema end-to-end para escanear códigos de barras y mostrarlos en un display (utilizando periféricos reales). En líneas generales no es más que un modelo MVC, pero no os imagináis todas las reflexiones que se van originando durante el proceso. Introduce además las dos formas diferentes de afrontar el diseño mediante TDD, la primera sin utilizar Mocks y la segunda haciendo uso de ellos ("Client-First Design with Mock Objects").

A destacar los asombrosos procesos de refactoring que lleva a cabo en todo momento, siempre paso a paso y ejecutando la suite de tests después de cada cambio. Podría poner algún ejemplo en este post, pero no quiero desviarme demasiado del asunto principal (detallar un "learning path"). Seguramente en el futuro profundize en todo esto.

En mi repositorio de GitHub podéis encontrar [el código](https://github.com/raulavila/tdd-jbrains) que desarrollé en este curso, pero añado de nuevo que no os servirá de mucho si no lo implementáis vosotros mismos.

### Conclusiones

Las fuentes detalladas en este artículo están orientadas fundamentalmente al lenguaje Java. Si vuestro lenguaje principal es otro seguramente haya otros cursos ahí fuera, aunque realmente la metodología no cambiará en absoluto, y me atrevería a afirmar que Java es un lenguaje ideal para seguir las explicaciones aunque no seas un conocedor, debido a su verbosidad.

Si conocéis algún libro, curso online o vídeo en YouTube que creáis tenga un gran valor añadido, agradecería lo añadiérais a los comentarios. El camino del TDD es largo, y no termina una vez finalizados los tres pasos que aquí he reseñado. De hecho, en breve podré experimentar lo que es TDD a nivel corporativo, y por supuesto daré rendida cuenta en este blog.
