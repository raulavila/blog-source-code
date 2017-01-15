---
layout: post
title: Patterns of Enterprise Application Architecture (1)
description: Repasando el libro Patterns of Enterprise Aplication Architecture de Fowler, primera parte
permalink: 2015/09/patterns-eaa-1/
tags:
- desarrollo
- patrones
- arquitectura
- libros
comments: true
---

![Patterns of EAA, the book](/public/pictures/patterns-eaa.jpg)

Recientemente, y tras varios meses de lectura sosegada, he terminado [este libro](http://www.amazon.es/Enterprise-Application-Architecture-Addison-Wesley-Signature/dp/0321127420/ref=sr_1_1?ie=UTF8&qid=1443261882&sr=8-1&keywords=patterns+of+enterprise+application+architecture). Todo un clásico del Software, publicado hace más de una década. Su autor, Martin Fowler, del que [ya hemos hablado alguna vez](/2015/02/fowler-refactoring-1/), es una referencia imprescindible para entender la evolución de nuestra industria.

Mi intención con estos posts es resumir en mayor o menor medida el contenido de esta obra, y dar mi opinión sobre su vigencia en este mundo que evoluciona vertiginosamente.

<!--break-->

## Qué es una aplicación empresarial

Para ponernos en contexto, veamos qué define el libro como una aplicación empresarial (Enterprise Application). A grandes rasgos, para que podamos englobar un sistema en este ámbito, debe cumplir todas (o casi todas) las características de la siguiente lista:

* Manejar persistencia de datos
* Acceder a la información de forma concurrente
* Tener un gran número de pantallas o interfaces con las que el usuario final puede interactuar
* En ocasiones estos sistemas deben procesar gran cantidad de datos en lotes (batch processing)
* Necesidad de integración con otras aplicaciones empresariales
* La información es manipulada en modos muy diversos

Ejemplos de este tipo de aplicaciones los conocemos todos: seguros, facturación, tiendas online, contabilidad, CRM, etc. En la mayoría de los casos estos sistemas toman la forma de aplicaciones Web.

No profundizaré mucho más, pero creo que está claro a qué podemos considerar "Enterprise Applications" (o eso espero).

## Métricas de rendimiento (performance)

Existen varias medidas de rendimiento que debemos tener en cuenta cuando diseñamos o implementamos este tipo de aplicaciones. Estas medidas formarán además parte de los conocidos como "requerimientos no funcionales", que en ocasiones son tan importantes como los funcionales:

* Tiempo de respuesta (Response time): es el tiempo que le lleva al sistema procesar una petición desde el exterior (invocación a la API, presionar un botón, etc)
* Capacidad de reacción (Responsiveness): tiempo que el sistema tarda en reconocer la petición. Difiere del anterior en que reconocer la petición no significa enviar la respuesta. Esta métrica va enfocada a mejorar la experiencia de usuario, pensad que siempre es mejor saber que "se está trabajando en ello" :) que desconocer si el servidor ha recibido o no nuestra acción. Si la UI no muestra ningún mensaje a este respecto antes de recibir la respuesta, entonces los tiempos de respuesta y la capacidad de reacción son iguales
* Latencia (Latency): es el mínimo tiempo requerido para recibir algún tipo de respuesta, incluso si el trabajo que hay que hacer es cero. Esta medida está bastante relacionada con factores ajenos a nuestra aplicación, como el retraso añadido al realizar llamadas remotas si trabajamos en red, por ejemplo. En general Tiempo de respuesta > Latencia
* Producción (Throughput): también podría ser traducido como rendimiento, pero en ese caso podría confundirse con performance (siendo performance más genérico en inglés). Es una medida de la cantidad de trabajo que nuestro sistema puede hacer en una cantidad determinada de tiempo. Ejemplos pueden ser transacciones por segundo si procesamos operaciones, bytes por segundo si copiamos un fichero, etc

En este punto debemos plantearnos si nuestra medida más importante debe ser el tiempo de respuesta o la producción. De cara al usuario, además, en ocasiones es más importante la capacidad de reacción. Un lío, vaya. Sigamos con más métricas:

* Carga (Load): mide el estrés a que está sometido un sistema, por ejemplo, usuarios conectados al mismo tiempo. Este medida afecta a otras, así por ejemplo, una mayor carga puede derivar en tiempo de respuesta más altos (10 usuarios - 0.5 segundos, 100 usuarios - 1 segundo)
* Sensibilidad de carga (Load sensitivity): mide cómo el tiempo de respuesta se ve afectado por la carga. Viene relacionado con el punto anterior, y decimos que un sistema tiene menor sensibilidad de carga cuanto mejor soporta los aumentos de carga. El concepto degradación (degradation) está relacionado con esto mismo, y así decimos que un sistema A se degrada menos que el B si soporta mejor los aumentos de carga
* Eficiencia (Efficiency): rendimiento(o producción) / recursos. Un sistema que soporta 30 transacciones por segundo en 2 CPUs es más eficiente que uno que soporta 40 en 4 CPUs
* Capacidad (Capacity): es una medida de la máxima carga o producción que puede soportar un sistema
* Escalabilidad (Scalability): mide la forma en que añadir recursos a un sistema mejora su rendimiento. El escenario ideal sería el de un sistema que dobla su rendimiento si doblamos los recursos hardware, aunque esto siempre es muy complicado. En terminos de escalabilidad tenemos dos categorías, escalabilidad vertical (scaling up), en la que añadimos más hardware a un servidor, o escalabilidad horizontal (scaling out) en la que añadimos más nodos a un cluster, por ejemplo

Como véis, las medidas no son pocas. En general, a la hora de diseñar aplicaciones empresariales deberíamos intentar maximimar la escalabilidad, ya que de esta forma tendremos una gran flexibilidad para ajustar otras medidas.

## Los patrones

En este primer post comentaré muy por encima los patrones desglosados en la primera parte del libro, y que en su mayoría fueron la fundación de muchos de los frameworks de acceso a datos que todos utilizamos actualmente. No traduciré los nombres de los patrones, ya que personalmente me suena muy raro hacerlo, y no creo que casi nadie lo haga, honestamente. Añadiré los links a cada patrón mencionado en la web oficial de [Fowler](http://martinfowler.com/).

### Domain Logic patterns

Bajo este grupo se agrupan patrones que sirven para organizar el procesamiento de una lógica de negocio determinada. Son cuatro, aunque diría que dos de ellos han quedado algo desfasados:

* [Transaction Script](http://martinfowler.com/eaaCatalog/transactionScript.html): organiza la lógica de negocio para gestionar una petición determinada (ejemplo: añadir un nuevo contrato a un sistema) en un solo procedimiento. Siendo honestos, no veo un uso claro para este patrón en un sistema diseñado con orientación a objetos. Sí le veo algo de salida si encapsulamos lógica compleja en procedimientos almacenados SQL, aunque yo personalmente no soy nada fan de esta opción
* [Domain Model](http://martinfowler.com/eaaCatalog/domainModel.html): es el contrapunto OO de Transaction Script. Consiste en encapsular lógica de negocio en los objetos de dominio. El problema que veo yo en este enfoque es que si utilizamos POJO's como objetos de dominio y les añadimos lógica de negocio, dejan de ser POJO's. En [esta página](https://dzone.com/articles/business-logic-domain-objects)) se menciona el problema, y hoy día, al utilizar frameworks como Hibernate lo normal es crear POJO's como estructuras planas sin lógica asociada
* [Table Module](http://martinfowler.com/eaaCatalog/tableModule.html): organiza la lógica de negocio en una clase por tabla de base de datos. Mientras que en Domain Model tenemos una instancia por registro de BDD, en este patrón tendríamos una instancia por tabla
* [Service layer](http://martinfowler.com/eaaCatalog/serviceLayer.html): capa de servicios (sic) que sirve como wrapper para el Domain Model. Expone las operaciones que se pueden hacer en el back end para su uso por el front end, realiza labores de coordinación, etc

Desde la perspectiva actual, mi enfoque preferido sería Domain Model + Service Layer, siendo el Domain Model una capa de estructuras de datos sin más. Por supuesto, para poblar dichas estructuras tendremos que utilizar una DAO layer ([Data Access Object](https://en.wikipedia.org/wiki/Data_access_object)), que no se menciona en este libro, aunque sí un patrón muy parecido de nombre Repository (nombre que a su vez fue extrapolado por Spring en la anotación [@Repository](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/Repository.html)), por lo que podríamos considerar al patrón Repository de este libro el precendente de DAO.


### Patrones de acceso y estructuración de datos

En este grupo englobo tres secciones del libro: Data Source Architectural Patterns, Object-Relational Behavioral Patterns y Object-Relational Structural Patterns.

Desde mi humilde opinión, muchos de los patrones agrupados en estas tres secciones han quedado algo desfasados tras la irrupción de todo tipo de frameworks de acceso a datos (sean JPA / Hibernate, MyBatis o jdbcTemplate), así que obviaré varios en el post. No obstante, he de añadir que lo que ha ocurrido en realidad es ¡que dichos frameworks implementan varios de estos patrones! Así, por ejemplo, tenemos un patrón llamado [Metadata Mapping](http://martinfowler.com/eaaCatalog/metadataMapping.html), del que es fácil deducir su uso: se encarga de mapear una tabla de base de datos con un objeto de dominio. [¿Os suena?](http://www.tutorialspoint.com/hibernate/hibernate_mapping_files.htm). Por lo tanto, quizás no sea justo hablar de obsolescencia, sino de desuso en el lado del desarrollo de estas aplicaciones (que no del lado de los desarrolladores de frameworks de acceso a datos).

Los patrones que sí considero importante mencionar por uno u otro motivo son:

* [Unit of Work](http://martinfowler.com/eaaCatalog/unitOfWork.html): es un objeto que encapsula los objetos / acciones afectados por una transacción de negocio (Business Transaction), y se encarga de ejecutar los commits y resolver problemas de concurrencia. En un ORM una Unit of Work sería una transacción, pero yo encuentro este patrón extrapolable a otro tipo de ambitos, como envoltorio de una serie de acciones, por ejemplo, o como container de los diferentes estados por los que atraviesa un objeto antes de finalizar una petición
* [Identity Map](http://martinfowler.com/eaaCatalog/identityMap.html): asegura que un objeto es transferido desde base de datos una sola vez en una transacción, almacenándolo en un mapa (id=>object), a modo de caché. Esto es un poco lo que ocurre en Hiberante cuando cargamos un objeto en una sesión y luego volvemos a cargarlo (que se cargará desde la sesión, sin ir a base de datos)
* [Lazy Load](http://martinfowler.com/eaaCatalog/lazyLoad.html): consiste en crear un objeto que no contiene toda la información que realmente expone, cargándola bajo demanda. Otro patrón implementado en [Hibernate](http://howtodoinjava.com/2014/09/26/lazy-loading-in-hibernate/)
* [Query object](http://martinfowler.com/eaaCatalog/queryObject.html): encapsula una query a BDD. La [API Criteria](https://docs.jboss.org/hibernate/orm/3.2/api/org/hibernate/Criteria.html) de Hibernate lo implementa, por ejemplo. Me gusta la idea de este patrón para facilitar el acceso a otro tipo de estructuras de datos, o incluso Web Services, por eso lo menciono aquí
* [Repository](http://martinfowler.com/eaaCatalog/repository.html): el precedente de DAO que ya comenté anteriormente

Otro tipo de patrones mencionados en el libro tratan temas como la creación de una clave única para nuestros objetos ([Identity Field](http://martinfowler.com/eaaCatalog/identityField.html)), la creación de asociaciones entre objetos ([Foreign Key Mapping](http://martinfowler.com/eaaCatalog/foreignKeyMapping.html), [Association Table Mapping](http://martinfowler.com/eaaCatalog/associationTableMapping.html), [Dependent Mapping](http://martinfowler.com/eaaCatalog/dependentMapping.html)...), o las diferentes estrategias para modelar un grupo de objetos de dominio y sus tablas asociadas cuando existe herencia en los objetos de dominio. Todos estos patrones es mejor extrapolarlos a la tecnología de acceso a dato concreta que utilicemos, y utilizar las soluciones expuestas en cada framework. Además, sería demasiado farragoso profundizar en todos los detalles en un modesto blog :).

Lo dejamos aquí por hoy. [En el siguiente post](/2015/10/patterns-eaa-2) resumiré los patrones detallados en la segunda parte del libro.
