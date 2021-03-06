---
layout: post
title: Patterns of Enterprise Application Architecture (y 2)
description: Repasando el libro Patterns of Enterprise Aplication Architecture de Fowler, segunda parte
permalink: 2015/10/patterns-eaa-2/
tags:
- desarrollo
- patrones
- arquitectura
- libros
comments: true
---

Continuamos con el repaso a [la obra de Fowler](http://www.amazon.es/Enterprise-Application-Architecture-Addison-Wesley-Signature/dp/0321127420/ref=sr_1_1?ie=UTF8&qid=1443261882&sr=8-1&keywords=patterns+of+enterprise+application+architecture). [En la primera entrega](/2015/09/patterns-eaa-1) nos centramos en describir qué es una aplicación empresarial, pasar lista a una serie de métricas de rendimiento, y destacar algunos de los patrones descritos en la primera parte del libro. En este nuevo post, que espero no quede demasiado farragoso, intentaré resumir los que considero los patrones más interesantes de la segunda parte.

<!--break-->

## Web Presentation Patterns

La mayoría de estos patrones tenían más sentido cuando fue escrito el libro, es decir, antes de que se existieran todos los frameworks web que nos hacen la vida tan fácil ahora ([Spring MVC](http://docs.spring.io/spring-framework/docs/current/spring-framework-reference/html/mvc.html), [Java Server Faces](https://es.wikipedia.org/wiki/JavaServer_Faces)). No obstante, creo que merece la pena destacar alguno:

* [Model View Controller](http://martinfowler.com/eaaCatalog/modelViewController.html): el clásico de los clásicos, la rockstar de los patrones, diría yo. No creo que se acuñara en este libro (aunque no estoy seguro), y no seré yo quien lo explique de nuevo en el blog, ya que internet está lleno [de referencias](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller), [video tutoriales](https://www.youtube.com/watch?v=qXRcVhWxuaU), e ¡incluso tiene [su propia canción](https://www.youtube.com/watch?v=YYvOGPMLVDo)!
* [Front Controller](http://martinfowler.com/eaaCatalog/frontController.html): se trata de un controlador que maneja todas las peticiones (requests) de una aplicación web, algo que recuerda sospechosamente al [DispatcherServlet](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html) de Spring MVC :). Su contrapunto es el patrón [Page Controller](http://martinfowler.com/eaaCatalog/pageController.html), que encapsula en un objeto toda la lógica que recoge las peticiones para una página determinada, y que, volviendo de nuevo a Spring, podríamos decir que la anotación `@Controller` debidamente utilizada nos permite implementar este patrón. De hecho, yo diría que Spring MVC es, entre otras muchas cosas una implementación de estos dos patrones
* [Application Controller](http://martinfowler.com/eaaCatalog/applicationController.html), patrón similar a Front Controller, pero que además maneja el flujo de la navegación en nuestra aplicación, es decir, conoce de antemano cuál será la siguiente página que hay que mostrar al usuario al finalizar la acción en curso, y en base a ciertas condiciones. [Spring WebFlow](http://projects.spring.io/spring-webflow/) es el módulo de Spring que implementa este patrón
* Patrones de presentación: a destacar el que más ha sobrevivido el paso del tiempo, [Template View](http://martinfowler.com/eaaCatalog/templateView.html). Es, como indica su nombre, una plantilla HTML a rellenar (lo que alguna vez hemos hecho todos con JSP o [FreeMarker](http://freemarker.org/), vaya)

## Distribution Patterns

Las llamadas remotas a sistemas o procesos ubicados en diferentes servidores siempre serán costosas, por mucho que evolucione el hardware o las velocidades de conexión. Para minimizar el impacto de estas llamadas tenemos dos patrones:

* [Remote Facade](http://martinfowler.com/eaaCatalog/remoteFacade.html): es una fachada remota pesada que encapsula llamadas a una API más ligera. El objetivo es minimizar el número de conexiones / peticiones. Así, en el ejemplo del link (y del libro), para actualizar una dirección, tarea que en la API original requeriría de varias llamadas a los métodos `setStreet`, `setCity`, etc, en la fachada lo dejamos como un solo método, `setAddress(street, city, zip)`, que recibirá todos los parámetros
* [Data Transfer Object](http://martinfowler.com/eaaCatalog/dataTransferObject.html): como su propio nombre bien indica, encapsula un conjunto de datos para su transferencia. Viene a ser el complemento a Remote Facade cuando necesitamos devolver un conjunto de datos, de forma que en lugar de invocar varios getters, hacemos una sola llamada que nos traerá de vuelta toda la información

## Offline Concurrency Patterns

La concurrencia es uno de los temas más complicados de comprender y utilizar correctamente en el desarrollo software. En este modesto blog [ya hablamos de ello](/2015/05/multithreading-1) desde un punto de vista más o menos práctico, pero a más bajo nivel.

Haciendo una pequeña digresión, una de las preguntas más típicas en las entrevistas de trabajo (al menos en Londres) es la diferencia entre "Optimistic Locking" y "Pessimistic Locking". El libro de Fowler utiliza un ejemplo que me parece totalmente perfecto para entender las diferencias, y es el trabajo en un equipo de desarrollo utilizando control de versiones. En este contexto, "Pessimistic Locking" sería una estrategia en la que una persona bloquea el fichero que va a modificar, y hasta que no lo libere, nadie puede modificarlo a la vez, y "Optimistic Locking" es la estrategia utilizada con herramientas como SVN o Git, donde tu modificas un fichero, y a la hora de hacer commit (o push en Git), si otra persona ha modificado las mismas líneas surge un **conflicto**, que rechaza el commit y hay que resolver, mientras que si otra persona ha modificado el mismo fichero en diferentes líneas no pasa nada. Otra forma de denominar estas dos estrategias es "detección de conflicto" (optimistic) y "prevención de conflicto" (pessimistic). Con esta base tenemos los siguientes patrones de concurrencia en el libro:

* [Optimistic Offline Lock](http://martinfowler.com/eaaCatalog/optimisticOfflineLock.html): lo dicho, si una transacción genera un conflicto con otra que ha tenido lugar en el mismo tiempo, este patrón detecta el conflicto y hace rollback a la transaccción que intenta completarse más tarde. Para implementar este patrón en una base de datos es necesario añadir una columna de versión que se incrementa por cada transacción realizada, de forma que la transacción sabe cuál fue el número de versión original, y si este cambia al ejecutar el commit es porque ha habido modificación concurrente
* [Pessimistic Offline Lock](http://martinfowler.com/eaaCatalog/pessimisticOfflineLock.html): implementa la estrategia de pessimistic locking. Necesita para ello un Lock Manager que controle y bloquee el acceso concurrente a información que ya está siendo utilizada. Dentro de este patrón existen una estrategia más ligera (read/write lock) que permite el acceso concurrente para lectura, pero si alguien adquiere el lock de escritura se bloquean todos los accesos posteriores. En general este patrón es menos recomendado que el anterior, porque crea una alta contención (usuarios peleando por recursos) y reduce muchísimo la concurrencia

Aunque este tema es muy complejo de implementar correctamente, tenemos la suerte de que [JPA lo ha definido en su estándar](https://blogs.oracle.com/carolmcdonald/entry/jpa_2_0_concurrency_and), por lo que cualquier framework que implementa JPA expone en su API el uso de estas estrategias. Aunque eso no quita que tengamos que seguir siendo extremadamente cuidadosos para evitar deadlocks y demás

* [Coarse-Grained Lock](http://martinfowler.com/eaaCatalog/coarseGrainedLock.html): se trata de utilizar un lock común para un grupo de objetos que deben ser tratados como unidad (o aggregate). Así por ejemplo, si un objeto `Person` tiene un objeto `Address` asociado, aunque solo queramos modificar la dirección vamos a bloquear también a la persona. Esta estrategia puede implementarse de forma pesimista u optimista
* [Implicit Lock](http://martinfowler.com/eaaCatalog/implicitLock.html): encapsula la estrategia de locking en un objeto determinado, haciéndolo transparente para los objetos que lo usan. Si entendemos el concepto de Thread Safe, esto viene a ser lo mismo pero ampliado al ámbito de una transacción de negocio. De nuevo, hay que ser muy cuidadosos para implementar esta estrategia

## Session State Patterns

A la hora de plantearnos la escalabilidad de un sistema que desarrollemos, el mayor impedimento será siempre el mantenimiento del estado de las sesiones de usuario. Los tres patrones descritos en esta sección responden a la pregunta, ¿dónde guardo el estado de las sesiones?

* [Client Session State](http://martinfowler.com/eaaCatalog/clientSessionState.html): almacena el estado completo de la sesión en cliente, haciendo al servidor totalmente stateless. Los problemas de esta estrategia es que las opciones para guardar información en cliente no son demasiadas (cookies, parámetros en la URL, hidden fields), y la información a almacenar ha de ser limitada. También tenemos consideraciones de seguridad, tanto desde el punto de vista de protección de datos de nuestros clientes, como seguridad en nuestro propio servidor al recibir información desde el cliente (cosa que obliga a validar esta información en cada request). Lo bueno de esta estrategia es que la escalabilidad es ilimitada
* [Server Session State](http://martinfowler.com/eaaCatalog/serverSessionState.html): pues eso, almacenar la sesión en el servidor. Conviene resaltar que, utilizando esta estrategia necesitaremos al menos almacenar una mínima información en cliente debido a la naturaleza stateless del protocolo HTTP (la famosa cookie JSESSIONID en Java, por ejemplo). A grandes rasgos, esta cookie sería la clave en un mapa que localiza los datos de la sesión para un usuario determinado a lo largo de una transacción que comprende varias requests. El mayor problema de este patrón es la escalabilidad, que se resuelve con estrategias como [sticky sessions](http://stackoverflow.com/questions/10494431/sticky-and-non-sticky-sessions) (en base a la IP, por ejemplo, todas las requests de un usuario irían a parar al mismo nodo), o migración de datos de sesión entre nodos del cluster (si se detecta que una petición de una transacción en curso va a parar a otro nodo)
* [Database Session State](http://martinfowler.com/eaaCatalog/databaseSessionState.html): es una modalidad de Server Session State que mantiene la información de sesión en base de datos. Más fácilmente escalable, debido a la naturaleza centralizada de una base de datos, un problema a considerar es la limpieza de sesiones longevas (para lo que necesitaríamos un demonio que lo compruebe a determinados intervalos). Otro problema es que su rendimiento respecto a Server Session State es menor, al introducir las conexiones a BDD en la ecuación

## Base Patterns

La última sección del libro describe una serie de patrones más genéricos, y relativamente sencillos (sobre todo tras asimilar todo lo que venía antes). Los más importantes son:

* [Gateway](http://martinfowler.com/eaaCatalog/gateway.html): es un objeto que encapsula el acceso a un sistema o recurso externo. La diferencia con el patrón Remote Facade mencionado anteriormente es que Gateway es implementado por el cliente para simplificar el acceso a un sistema, mientras que Remote Facade es expuesto por el sistema para reducir el número de llamadas de los clientes que lo utilicen
* [Separated Interface](http://martinfowler.com/eaaCatalog/separatedInterface.html): define una interfaz en un paquete diferente al de la implementación. Esta patrón está estrechamente relacionado con el [Dependency Inversion Principle](/2015/03/principios-dependencias)
* [Registry](http://martinfowler.com/eaaCatalog/registry.html): viene a ser un punto de acceso para localizar servicios u objetos de uso general. Su implementación más común es mediante métodos estáticos, y el propio Fowler desaconseja su uso. Las dependencias estáticas no son nada recomendables para desarrollar tests unitarios solidos, y en estos días de frameworks de inyeccción de dependencias seguro que encontramos mejores alternativas. Así que básicamente, cito este patrón como anti-pattern :)
* [Money](http://martinfowler.com/eaaCatalog/money.html): representa un valor monetario y permite realizar cálculos con dinero (encapsulando toda la lógica de esos cálculos, el redondeo, etc)
* [Special Case](http://martinfowler.com/eaaCatalog/specialCase.html): es una clase que implementa un caso especial de uso y que nos evita plagar nuestro código de comprobaciones para ese caso especial. Un buen ejemplo sería el [Null Object Pattern](https://sourcemaking.com/design_patterns/null_object)
* [Plugin](http://martinfowler.com/eaaCatalog/plugin.html): otro patrón relacionado con el Dependency Inversion Principle. Viene a decir que el linkado de las clases se realiza en tiempo de configuración en lugar de en tiempo de compilación. O en otras palabras: programa con interfaces y utiliza inyección de dependencias
* [Service Stub](http://martinfowler.com/eaaCatalog/serviceStub.html): encapsula el acceso a un servicio externo mediante una interfaz para facilitar los tests unitarios, mediante mocks de dicha interfaz. Nada que las buenas prácticas y los [principios SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) no nos dicten, vamos

## Conclusión

Este libro es un obra monumental, de importancia capital para entender el desarrollo software tal y como lo entendemos hoy día. Aunque el tiempo siempre pasa factura a este tipo de libros, debido al ritmo de evolución de nuestra industria, me atrevería a decir que sigue siendo vigente, con la excepción de algunos patrones, que más que haber perdido vigencia han perdido visibilidad gracias a los frameworks que todos conocemos.

Espero que estos dos posts empujen a alguien a profundizar en el libro, o que al menos sirvan como referencia para considerar el uso de patrones que antes eran más desconocidos.
