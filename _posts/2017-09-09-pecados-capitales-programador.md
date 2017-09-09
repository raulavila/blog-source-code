---
layout: post
title: Los 7 pecados capitales del programador
description: Siete pecados capitales que los programadores cometen y pueden arruinar un proyecto
permalink: 2017/09/pecados-capitales-programador/
tags:
- desarrollo
- buenas-practicas
comments: true
---

Inspirado por el post [Los 7 pecados capitales del Product Owner](https://jeronimopalacios.com/2015/03/los-7-pecados-capitales-del-product-owner/) publicado hace unos años por [Jerónimo Palacios](https://twitter.com/giropa832) me he decidido a recopilar los que son, para mí, los siete pecados más importantes que he cometido o he visto cometer a programadores a lo largo de mi carrera.

Todos estos pecados pueden, por sí mismos, llevar a la ruina a un proyecto de desarrollo, algunos de forma más rápida que otros. Pero cuando son realmente dañinos es cuando se dan de forma conjunta.

<!--break-->

## 1. Descuidar los tests

A estas alturas espero que no sea necesario explicar por qué es necesario implementar tests automatizados en las aplicaciones o sistemas que desarrollamos, ¿verdad? Es responsabilidad única de los desarrolladores garantizar el correcto funcionamiento del software entregado, así como la falta de regresión según vayamos evolucionándolo, y la única herramienta que nos permite garantizar esto en todo momento son los tests.

Personalmente, no concibo entregar nada que no venga respaldado por su correspondiente suite de tests. [Practicar TDD](/2016/01/aprendiendo-TDD/) es una buena forma de no saltarse esta norma, pero aún con esas, siempre nos asaltará la debilidad en momentos puntuales de presión, en los que intentaremos recortar del lado más fácil, que suelen ser los tests. Por favor, no caigáis nunca en ese error porque siempre (siempre) lo acabaréis lamentando. Además, recordad la teoría de las [ventanas rotas](/2016/09/ventanas-rotas/) y lo que conlleva a largo plazo.

## 2. No hacerse responsables de nuestro sistema de integración continua

Desde el momento cero de un proyecto debe existir un pipeline de integración continua que chequee regularmente el estado de nuestro(s) repositorio(s) (tanto a nivel de tests como de [análisis estático](https://es.wikipedia.org/wiki/An%C3%A1lisis_est%C3%A1tico_de_software), etc) y despliegue la aplicación en los entornos que corresponda. El estado de dicho pipeline debe estar en todo momento visible para todos los miembros del equipo, a poder ser en una pantalla ubicada junto a nuestras mesas, y cualquier desviación de lo que sea considerado un estado aceptable ha de ser analizada inmediatamente por un desarrollador.

Es bastante frecuente que nadie se haga realmente responsable de esto que acabo de comentar. El motivo principal suele ser que siempre apetece más desarrollar nuevas funcionalidades, a fin de cuentas es más divertido. Pero del mismo modo que entregar software sin sus tests asociados se termina pagando, no rectificar una desviación en la estabilidad del pipeline CI puede pasar de ser un pequeño problemilla que ocuparía entre una y dos horas resolver, a una pelota que nadie sabe como atacar.

Así que, por favor, estableced una disciplina de trabajo donde esté claro quién se va a responsabilizar de mantener vuestro pipeline CI. Pueden utilizarse diferentes estrategias, desde la persona que realizó el último commit que "rompió" el pipeline, a un miembro fijo que rote cada día o semana.

## 3. Elegir tecnologías para engordar nuestro CV

Hemos de elegir la mejor herramienta o tecnología para el trabajo que tenemos entre manos, y nunca dejarse llevar por la curiosidad de "probar" algo que nos llama la atención porque lo hemos visto en una conferencia o leído en un newsletter, para así de paso meterlo en la lista de tecnologías de nuestro CV. Si hacemos algo así sin tener realmente claro que es la mejor opción posible, estaremos malgastando el dinero del cliente o de la empresa que nos está pagando para nuestro propio beneficio.

No estoy diciendo que haya que cerrar el abanico de opciones, en muchas ocasiones una tecnología que nos es desconocida será la adecuada, y tendremos que dedicarle tiempo para aprenderla. Pero antes de realizar la inversión debemos tener bien claro tras un análisis o comparativa previa que estamos haciendo lo correcto.

## 4. Intentar ser imprescindibles

Más de una y de dos veces, me he encontrado con compañeros que de alguna manera pensaban que era inteligente no compartir conocimiento con el resto del equipo. Esto se traducía en cosas como código escrito por ellos que era imposible de entender por nadie más, librerías añadidas de forma unilateral, decisiones de implementación no discutidas con el resto del equipo...No era extraño además que adoptaran una actitud defensiva cuando se les preguntaba por alguna de estas cosas.

En nuestra profesión es relativamente frecuente encontrarse con personalidades complicadas, y por eso mismo pienso que los procesos de selección deberían valorar la empatía de los candidatos al mismo nivel que las capacidades técnicas. Es importante recordar que un buen ambiente de equipo puede romperse de un día para otro por detalles como los que he mencionado en el anterior párrafo. Debemos además fomentar que nuestros equipos tengan la propiedad colectiva del código y el conocimiento compartido por bandera.

## 5. No refactorizar de forma responsable

Ya cubrí este tema recientemente en el post [Mis etapas con el refactoring](/2017/07/etapas-refactoring/). En mi opinión, todo refactoring que no entre dentro del apartado "Refactoring responsable" de dicho artículo tendrá consecuencias negativas tarde o temprano.

El refactoring es una actividad fundamental para conseguir software sostenible, pero también puede suponer una pérdida de tiempo si se hace cuando o donde no es necesario. Saber aplicarlo de manera efectiva es esencial.

## 6. No empatizar con el cliente

Los clientes, además de ser quienes en última instancia nos dan de comer, son personas como nosotros, y merecen el máximo respeto y empatía por nuestra parte. Quizás el sponsor de un proyecto no sea especialmente técnico y desconozca determinados aspectos del proyecto que estamos desarrollando para él, por lo que en ocasiones desbarrará un poco (o bastante) cuando cosas como los plazos de entrega salgan a discusión.

Cuando un cliente nos impone unos plazos que sabemos de antemano son completamente imposibles de llevar a cabo, y nosotros los aceptamos, sabemos que con ello comprometeremos los aspectos menos visibles de nuestros sistema, a saber: tests, calidad y mantenibilidad del código, seguridad, etc. Como profesionales debemos explicar a dicho cliente los riesgos que asumimos si seguimos adelante, y seremos transparentes al máximo para que no haya lugar a la duda, pero también nos pondremos en su piel para entender en qué situación se encuentra él para tratar de fijar esas fechas. Una vez conseguimos comprender sus motivos quizás sea más sencillo alcanzar un acuerdo de consenso.

He puesto el ejemplo de los plazos porque creo que todos lo hemos experimentado, pero existen otros muchos aspectos donde comprender el punto de vista del cliente es también necesario. No profundizo más porque el tema es denso y daría para una serie de posts :).

## 7. No pensar en quién vendrá después

¿A qué me refiero con esto? Me refiero tanto a futuros programadores que tendrán que lidiar con nuestro código para corregir bugs o añadir nuevas funcionalidades como al equipo de operaciones que tendrá que desplegar la aplicación en diferentes entornos, diagnosticar posibles problemas, etc. Es por tanto, fundamental, no descuidar aspectos ya mencionados anteriormente, como la legibilidad del código, pero también otros como proporcionar una documentación adecuada (a poder ser en forma de [README](https://jesuslc.com/2016/07/12/como-escribir-un-readme-que-mole/)), generar unas trazas (logs) de calidad, que en caso de problema permitan detectar qué está ocurriendo y facilitar la comunicación entre desarrollo y operaciones, encapsular parámetros de configuración en un solo lugar (y documentar cómo modificarlos), etc.
