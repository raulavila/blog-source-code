---
layout: post
title: Novedades en Spring 4 (y 2)
permalink: 2015/03/spring-4-novedades-2/
tags:
- Java
- Spring
- websockets
comments: true
---

Continuamos con el repaso a las novedades de la versión 4 de Spring, en esta ocasión más centrados en las relativas al desarrollo web, servicios REST, etc.

Cómo ya comenté en [la primera parte](/2015/02/spring-4-novedades), probablemente el proyecto más popular de Spring sea Spring MVC, y su mayor baza la claridad con la que podemos crear controladores web, bien sea manejando peticiones [Ajax](http://es.wikipedia.org/wiki/AJAX) o con un estilo más tradicional, redirigiendo a vistas JSP, etc. Es posible incluso configurar fácilmente y para la misma petición diferentes formatos de respuesta (JSON, XML, vista tradicional, CSV, lo que sea) mediante el uso de [view resolvers](http://examples.javacodegeeks.com/enterprise-java/spring/mvc/spring-mvc-view-resolver-example/), y se integra muy bien con librerías de plantillas como [Apache Tiles](https://tiles.apache.org/).

<!--break-->

Otra de las grandes ventajas de Spring MVC es lo rápido que permite desarrollar servicios REST, y por este lado vienen varias mejoras en la versión 4.

###Controladores REST

Antes de Spring 4, para crear un servicio REST era necesario el uso de dos anotaciones, `@Controller` para anotar la clase que escucha las peticiones, y `@ResponseBody`, que anota cada método "handler" en particular, y viene a significar "establece el retorno de este método como body del response", es decir, se transformará la estructura de datos a un formato JSON o similar como el cuerpo de la respuesta:

{% highlight java %}
@Controller
public class RestController32 {

    @RequestMapping("/greeting32")
    public @ResponseBody String getGreeting() {
    	return "Hello world - Spring 32";
    }

    @RequestMapping("/items32")
    public @ResponseBody List<String> getItems() {
    	return Arrays.asList("item1 - Spring 32","item2 - Spring 32");
    }
}
{% endhighlight %}

En el ejemplo, una petición GET a `/greeting32` (`@RequestMapping` manejará GET por defecto si no se indica lo contrario), devolverá como respuesta el código "200 OK" y el cuerpo de la respuesta será "Hello World - Spring 32". De forma similar el cuerpo de la respuesta de `/items32` será "["item1 - Spring 32","item2 - Spring 32"]" (formato JSON, transformación realizada mediante la librería [Jackson](http://jackson.codehaus.org/)).

Spring 4 introduce pequeñas mejoras para crear este tipo de controladores. Por un lado la anotación `@ResponseBody` pasa a ser anotación de método y clase (previamente lo era sólo de método), por lo que si la añadimos a nivel de clase, todos los métodos la heredarán, dejando el código mucho más limpio:

{% highlight java %}
@Controller
@ResponseBody
public class RestController32 {

    @RequestMapping("/greeting32")
    public String getGreeting() {
    	return "Hello world - Spring 4";
    }

    @RequestMapping("/items32")
    public List<String> getItems() {
    	return Arrays.asList("item1 - Spring 4","item2 - Spring 4");
    }
}
{% endhighlight %}

Mejor aún, la gente de Spring pensaría por qué no fusionar estas dos anotaciones en una sola, y así hicieron, dando lugar a `@RestController`. El resultado es bastante directo y auto-explicativo:

{% highlight java %}
@RestController
public class RestController4 {

    @RequestMapping("/greeting4")
    public String getGreeting(ZoneId zoneId) {
    	return "Hello world - Spring 4. TimeZone: "+zoneId;
    }

    @RequestMapping("/items4")
    public List<String> getItems() {
    	return Arrays.asList("item1 - Spring 4","item2 - Spring 4");
    }
}
{% endhighlight %}

En el ejemplo veréis además como el servicio `/greeting4` devuelve el time zone en su respuesta. He incluido esto en el ejemplo como muestra del soporte a la nueva API para fechas y horas que [incluye la versión 8 de Java](http://www.oracle.com/technetwork/articles/java/jf14-date-time-2125367.html). Aún no he profundizado en esta API lo suficiente para hablar con propiedad, pero parece ser que al fin se ha llegado a una solución en condiciones para manejar esta información (ya era hora, después de 7 versiones :) ).

###Clientes REST

En Spring existía la clase `RestTemplate`, que sirve para consumir API's REST de forma sencilla:

{% highlight java %}
@Test
public void testSyncRestTemplate() {
    RestTemplate restTemplate = new RestTemplate();

    ResponseEntity<String> response = restTemplate.getForEntity(
            "http://localhost:8080/items4", String.class);

    System.out.println("Received!:");

    assertThat(response.getStatusCode())
            .isEqualTo(HttpStatus.OK);
    assertThat(response.getBody())
            .isEqualTo("[\"item1 - Spring 4\",\"item2 - Spring 4\"]");
}
{% endhighlight %}

El problema de este cliente es que es síncrono, por lo que las llamadas son bloqueantes, y dificulta en gran medida el desarrollo de aplicaciones multithreading, más que nada porque nos obliga a desarrollar un wrapper asíncrono. Supongo que muchos desarrolladores se quejaron a la comunidad Spring de este asunto, porque la versión 4 viene con su propio cliente asíncrono, `AsynRestTemplate`. A grandes rasgos podemos utilizar este cliente de dos formas diferentes:

1- Bloqueando el thread mientras esperamos la respuesta:

{% highlight java %}
AsyncRestTemplate restTemplate = new AsyncRestTemplate();

ListenableFuture<ResponseEntity<String>> futureEntity =
		restTemplate.getForEntity(
				"http://localhost:8080/items4",
				String.class);

ResponseEntity<String> result = null;

try {
    //blocks waiting for the response
    result = futureEntity.get();

    System.out.println("Received: " + result.getBody());

} catch (InterruptedException | ExecutionException e) {
    e.printStackTrace();
}
{% endhighlight %}

La llamada a `futureEntity.get()` suspenderá el thread que ejecuta la petición REST hasta recibir la respuesta, por lo que otros threads se podrán seguir ejecutando mientras este está suspendido. Es decir, eliminamos esperas activas y  optimizamos recursos.

2- Configurando funciones callback:

{% highlight java %}
AsyncRestTemplate restTemplate = new AsyncRestTemplate();

ListenableFuture<ResponseEntity<String>> futureEntity = restTemplate
		.getForEntity("http://localhost:8080/items4",
				String.class);

futureEntity.addCallback(
		new ListenableFutureCallback<ResponseEntity<String>>() {
			@Override
			public void onSuccess(ResponseEntity<String> result) {
				System.out.println("Received!: " + result.getBody());
			}

			@Override
			public void onFailure(Throwable t) {
				System.out.println("Fail!");
			}
		});
{% endhighlight %}

Cómo habréis deducido, los métodos callback serán ejecutados en un thread paralelo una vez se obtenga respuesta (exitosa o no) del servicio. Opción más elegante en mi opinión, porque elimina la necesidad de tener que manejar en el thread principal las checked exceptions lanzadas por `futureEntity.get()` (`InterruptedException` y `ExecutionException`), y además separa claramente las acciones a realizar en caso de éxito o fracaso de la petición REST.

###Soporte para WebSockets

Siendo honestos, mi conocimiento de WebSockets se limita a lo que he trasteado con Spring 4 para escribir este post, así que es posible que meta algún gambazo en mi explicación. No obstante, el ejemplo que voy a mostrar (como siempre, [subido a mi repositorio GitHub](https://github.com/raulavila/spring-4-new-features)) funciona perfectamente.

WebSockets es un protocolo que permite el envío de mensajes en ambas direcciones, cliente => servidor y servidor => cliente, sin necesidad de petición previa por parte del cliente, más allá de la suscripción inicial a un topic ("handshake" inicial, operación realizada mediante HTTP). WebSockets es un protocolo que funciona sobre TCP (más allá del "apretón de manos" inicial), y a su vez necesita un sub-protocolo para gestionar los mensajes enviados. La opción principal en Spring es el protocolo [STOMP](http://en.wikipedia.org/wiki/Streaming_Text_Oriented_Messaging_Protocol).

Un problema con los websockets es su soporte por parte de determinados navegadores (adivinad [cuáles](http://imagenes.es.sftcdn.net/es/scrn/94000/94114/internet-explorer-9-35.png)). Para solventar esta situación, es necesario que exista un protocolo alternativo para el caso en que el navegador en cuestión no soporte WebSockets. Spring utiliza [SockJS](https://github.com/sockjs/sockjs-protocol) como "fallback option".

El ejemplo que mostraré está basado en [la documentación de Spring](http://spring.io/guides/gs/messaging-stomp-websocket/), pero sin utilizar Spring Boot. No explicaré paso a paso el proceso, ya que la documentación es bastante buena, sólo haré hincapié en los principales aspectos a tener en cuenta para configurar una aplicación con WebSockets.

#####Configuración de WebSockets en la aplicación

{% highlight java %}
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig
		extends AbstractWebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
    	config.enableSimpleBroker("/topic");
    	config.setApplicationDestinationPrefixes("/app");
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
    	registry.addEndpoint("/hello").withSockJS();
    }
}
{% endhighlight %}

Aquí estamos indicando que:

* ésta se trata de una clase de configuración de Spring (`@Configuration`)
* se activen el soporte para websockets (`@EnableWebSocketMessageBroker`).

En los métodos indicamos que:

* se cree un broker en memoria que se encargará de enviar mensajes a los clientes suscritos a topics con el prefijo /topic (`config.enableSimpleBroker("/topic")`). Si la palabra broker os suena a chino (como a mí hasta hace unos meses), echad un vistazo al [patrón broker](http://stackoverflow.com/questions/23830413/broker-architectural-pattern-in-plain-english)
* se manejarán mensajes en endpoints prefijados con "/app" (`config.setApplicationDestinationPrefixes("/app")`)
* se cree un endpoint "/app/hello" para manejos de mensajes STOMP desde clientes, y con SockJS como opción de respaldo (`registry.addEndpoint("/hello")).withSockJS()`)


#####Clase controladora

{% highlight java %}
@Controller
public class GreetingController {
    @MessageMapping("/hello")
    @SendTo("/topic/greetings")
    public Greeting greeting(HelloMessage message) throws Exception {
    	Thread.sleep(3000); // simulated delay
    	return new Greeting("Hello, " + message.getName() + "!");
    }

}
{% endhighlight %}

La clase está anotada como cualquier otra clase controladora de Spring MVC (`@Controller`). En el método greeting:

* Escuchamos mensajes enviados al endpoint "/app/hello" (`@MessageMapping("/hello")`). La falta del prefijo "/app" en la configuración puede resultar algo confusa
* Publicar el mensaje de retorno al topic del broker "/topic/greetings" (`@SendTo("/topic/greetings")`)

Por lo demás, el método espera recibir una instancia correcta de `HelloMessage`, y devolverá una instancia de `Greeting`

#####Configuración de cliente (Javascript)

(he omitido algunas partes del código que sí está subido a GitHub por claridad)

{% highlight javascript %}

<script src="js/sockjs-0.3.4.js"></script>
<script src="js/stomp.js"></script>

<script type="text/javascript">

    var stompClient = null;

    function connect() {
        var socket = new SockJS('/hello');
        stompClient = Stomp.over(socket);

        stompClient.connect({}, function(frame) {
            console.log('Connected: ' + frame);
            stompClient.subscribe(
                    '/topic/greetings',
                    function(greeting){
                        showGreeting(JSON.parse(greeting.body).content);
                    });
        });
    }

    function sendName() {
        var name = document.getElementById('name').value;
        stompClient.send("/app/hello", {}, JSON.stringify({ 'name': name }));
    }

    function disconnect() {
        stompClient.disconnect();
        console.log("Disconnected");
    }

</script>

{% endhighlight %}

Aquí vemos que:

* Es necesario incluir las librerías JS `sock-js` y `stomp`
* En el método `connect` vemos cómo hay que suscribirse a un topic, siendo necesario configurar el nombre completo del topic y la función callback cuando recibamos un mensaje publicado por el broker
* En el método `sendName` mandamos un mensaje al endpoint "/app/hello", que será manejado en el controlador. Nótese como en este ejemplo recibimos el mensaje de vuelta desde el broker en el mismo cliente, pero no tiene por qué ser así
* En el método disconnect eliminamos la suscripción al topic

En el siguiente video tenéis una demostración en vivo de la aplicación (siempre es más fácil ver algo en directo para entenderlo del todo :), disculpad la falta de estilos CSS en condiciones). Como se puede ver, se abren dos pestañas a la misma aplicación, y cuando se conecta cada cliente al websocket este empezará a recibir los mensajes publicados sin necesidad de enviar petición previa:

<iframe class="youtube" width="420" height="315" src="https://www.youtube.com/embed/OTvaIbVYPrU" frameborder="0" allowfullscreen></iframe>

###Conclusiones

Terminamos aquí el repaso a las principales novedades de Spring 4. Creo que hay mejoras sustanciales respecto a la versión 3, y no dudo que a este framework aún le queda vida para rato.
