---
layout: post
title: ¿Programadores ágiles?
description: Las habilidades más importantes de un programador ágil
permalink: 2017/05/programadores-agiles/
tags:
- agile
comments: true
---

Supongo que a nadie le pillará por sorpresa si afirmo que el desarrollo Agile está en boca de toda las empresas hoy día. Incluso el tema se está llevando a otra dimensión con el deslumbrante nombre de "Transformación Digital". Por tanto, no creo que sea necesario explicar en este post qué es el desarrollo ágil.

El asunto ha llegado al nivel de que no es difícil encontrar ofertas donde se buscan "Agile Developers", y aquí es donde el asunto empieza a cobrar tintes algo cómicos. Personalmente, no creo que casi ninguna empresa que publica una oferta con ese título tenga mucha idea de las skills que diferencian a un programador ágil de uno que no lo sea, y si lo afirmo con tanta rotundidad es porque nunca he visto mención alguna a las que yo creo son las habilidades principales que todo desarrollador ágil debe tener.

<!--break-->

Fue Kent Beck quien [en una sesión de preguntas en Quora](https://www.quora.com/If-you-had-to-write-the-Agile-manifesto-again-would-you-change-something-on-it) comentó que el nombre "Agile" fuera quizás un error cuando un grupo de profesionales acuñaron el termino hace ya casi dos décadas. Lo que ha ocurrido con el tiempo es que las grandes corporaciones han comenzado a confundir "ágil" con "rápido", por tanto "Programador ágil" pasa a ser entendido como "Programdor rápido". Es decir, las empresas quieren gente que les entregue una mayor cantidad de trabajo en menos tiempo, pero sin tener bien claras las cualidades que un programdor capaz de hacer esto debe tener. Se piensa además, que con seguir las ceremonias de Scrum esta mayor velocidad se alcanzará mágicamente. Un poco absurdo todo.

En mi opinión, las principales cualidades del desarrollo ágil son el ritmo sostenible y el feedback constante. Para conseguir ambas cosas se necesita mejorar la comunicación, conseguir una máxima visibilidad del trabajo realizado, pero, por encima de todo, alcanzar una excelencia técnica en el proceso de desarrollo del software entregado.

Es esta excelencia técnica la que consigue, a la larga, incrementar la cantidad de funcionalidades entregadas por unidad de tiempo. En efecto, estoy hablando de abogar por la máxima calidad en lo que hacemos.

Dicho esto, ¿cuáles son las habilidades que debe tener un buen programador ágil? Pues, desde mi punto de vista, dos:

* Dominio del testing
* Dominio del [refactoring](https://martinfowler.com/books/refactoring.html)

Si un programador es capaz de entregar software perfectamente cubierto y descrito por tests, dicho software carecerá de regresión en el futuro (o tendrá muy poca), por lo que hacer crecer el sistema será mucho más sencillo, sin un decremento notorio en la velocidad. El hecho de tener los tests como "red de seguridad" permitirá además moldear nuestro código mediante el refactoring sin miedo a romper nada. Y es el refactoring constante, la adaptación de nuestro código para hacerlo más legible y mantenible lo que realmente tiene un impacto brutal en la calidad del software entregado.

¿Es más importante el testing o el refactoring? En realidad ninguna está por encima de otra, porque ambas se necesitan para crear un círculo virtuoso. Quizás os venga a la mente el [TDD](/2016/01/aprendiendo-TDD/) como consagración definitiva de este círculo. Y aunque personalmente considero que el TDD es increíblemente útil, en última instancia podría tolerar no utilizarlo siempre y cuando no se pierdan de vista la importancia de los tests y el refactoring en todo momento. Esta, y no otra, es la forma de conseguir el ritmo sostenible que mencionaba más arriba.

Por tanto, si en el futuro os topáis con una oferta donde se buscan "Agile Developers" pero no se mencionan una de estas dos habilidades, desconfiad :).
