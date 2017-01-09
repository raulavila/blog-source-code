---
layout: post
title: TDD&#58; Hello World Web App
permalink: 2016/03/hello-world-tdd/
tags:
- TDD
- desarrollo
comments: true
---

Seguimos con  mi viaje dentro del mundo TDD. Uno de los mayores dilemas a los que nos enfrentamos a la hora de diseñar un nuevo sistema mediante tests es por dónde debemos empezar, existiendo como existen tantas modalidades distintas de tests (unitarios, integración, de navegador, etc).

He pensado que la mejor forma de responder a esta pregunta es liarme la manta a la cabeza y crear una aplicación web "Hello World" mediante TDD. Mostraré en este post el código generado tras cada paso, y la versión final la podéis encontrar [en GitHub](https://github.com/raulavila/hello-world-webapp-tdd).

<!--break-->

### El primer commit

Los ejemplos más comunes que podemos encontrar utilizando TDD en la red son [aplicaciones muy interesantes a nivel didáctico](https://www.youtube.com/watch?v=7RJM3pcMNyo), pero tras comprender los principios de esta metología se nos plantea la pregunta de cómo aplica esto al mundo real, a las aplicaciones que todos desarrollamos en nuestros puestos de trabajo. Y la verdad es que cuesta encontrar ejemplos de, por ejemplo, una aplicación web desarrollada mediante TDD.

Así que vamos a ello, y como buenos informáticos que somos desarrollaremos un "Hello World", que no es más que una web que cumple estos requisitos:

> Cuando abro en mi navegador la url http://[domain]/hello se abre una página con el texto "Hello World"

Con estos criterios de aceptación lo primero sería decidir las tecnologías a utilizar. En nuestro caso serán:

* Java
* [Spring Boot](http://projects.spring.io/spring-boot/) + [Spring MVC](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html)
* [Thymeleaf](http://www.thymeleaf.org/) como template engine para crear las vistas web
* [Maven](https://maven.apache.org/) para construir el proyecto
* Para los tests unitarios y de integración nos apoyaremos en las APIs expuestas por Spring para escribir tests en [JUnit](http://junit.org/)
* Para los tests de navegador o funcionales utilizaremos [Fluentlenium](https://github.com/FluentLenium/FluentLenium), API sobre [Selenium](http://www.seleniumhq.org/) para crear este tipo de tests

Una vez decidido todo esto deberíamos crear el esqueleto de un proyecto que contenga todas estas dependencias y compile sin problemas. Para esto podemos recurrir al uso de [arquetipos Maven](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html), pero más fácil incluso es utilizar la estupenda herramienta [Spring Initialzr](https://start.spring.io/):

![Spring Initialzr](/public/pictures/hello-world-webapp-tdd/spring-initializr.jpg)

Con la configuración que aparece en la captura, y tras pulsar "Generate Project" tendremos un proyecto Maven con las dependencias de Spring necesarias. Sólo necesitamos añadir la librería Fluentlenium.

El proyecto generado, aparte del layout básico tendrá dos clases, una en el código de producción y otra en el código de test. Ésta es la clase en el código de producción:

{% highlight java %}
@SpringBootApplication
public class HelloWorldWebappTddApplication {

    public static void main(String[] args) {
        SpringApplication.run(
                HelloWorldWebappTddApplication.class, args);
    }
}
{% endhighlight %}

No es más que el punto de entrada para ejecutar una aplicación Spring Boot, de hecho esta primera versión del proyecto es perfectamente compilable y ejecutable, aunque no haga gran cosa.

La clase de test es ésta:

{% highlight java %}
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(
    classes = HelloWorldWebappTddApplication.class)
@WebAppConfiguration
public class HelloWorldWebappTddApplicationTests {

	@Test
	public void contextLoads() {
	}

}
{% endhighlight %}

Así de primeras mucha gente piensa que estamos ejecutando un test vacío, por lo tanto no estamos haciendo gran cosa. Pero esta clase de test prueba mucho, a saber:

* Podemos ejecutar tests de JUnit
* Podemos ejecutar tests de integración de Spring utilizando el runner `SpringJUnit4ClassRunner`
* Los tests de integración pueden cargar la configuración completa de nuestra aplicación Spring Boot

Sólo vamos a cambiar un pequeño detalle sobre este esqueleto inicial, y es la nomenclatura utilizada en las clases de test. Utilizaremos `*Test` para los tests unitarios, y `*IT` para los tests de integración y navegador. Para poder ejecutar los tests de integración convenientemente en Maven necesitamos configurar el plugin failsafe, según se explica [aquí](http://www.petrikainulainen.net/programming/maven/integration-testing-with-maven/). Y puesto que el test generado por Spring Initialzr es realmente un test de integración vamos a renombrar ese test como `HelloWorldWebappTddApplicationIT`, y a crear un nuevo test unitario vacío:

{% highlight java %}
public class HelloWorldWebappTddApplicationTest {

    @Test
    public void junitDefaultRunner() throws Exception {

    }
}
{% endhighlight %}

Puede parecer ridiculo tener dos clases de test vacías, pero no lo es. En esta segunda clase estamos probando que el runner por defecto de JUnit funciona correctamente, y de momento debemos mantener ambas. Hora de hacer el primer commit en el repositorio Git, pues.

### El primer test

El primer test que tenemos que escribir (tests "vacíos" aparte), es, ni más ni menos, que el test de aceptación automatizado del requisito principal de nuestra aplicación expuesto más arriba. Es decir, el punto de inicio para escribir una aplicación web mediante TDD es el test de navegador, que es el de más alto nivel.

Cómo el objetivo de este post no es aprender Fluentlenium, ni el magnífico [Page Object Pattern](http://martinfowler.com/bliki/PageObject.html), vamos a escribir el test más sencillo posible:

{% highlight java %}
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(
    classes = HelloWorldWebappTddApplication.class)
@WebAppConfiguration
@IntegrationTest("server.port:8080")
public class HelloPageIT extends FluentTest {
    public WebDriver webDriver = new HtmlUnitDriver();

    @Override
    public WebDriver getDefaultDriver() {
        return webDriver;
    }

    @Test
    public void openHelloPage() throws Exception {
        goTo("http://localhost:8080/hello");

        assertThat(find(".message").getText())
                .contains("Hello World");
    }
}
{% endhighlight %}

Este test, además de las anotaciones que ya conocemos de nuestro primer test de integración,  se aprovecha de la anotación `IntegrationTest` para arrancar un servidor web en el puerto 8080 y configura el driver `HtmlUnitDriver`, sustituyendo al driver de Firefox, que es el que utiliza Selenium por defecto.

Nuestro método de test tan sólo realiza las acciones esperadas por nuestras condiciones de aceptación:

1. Ir al endpoint `/hello`
2. Verificar que se abre una página con un mensaje "Hello World" dentro de una sección de clase "message"

Al ejecutar este test fallará, y aunque la excepción que se produce es algo confusa, viene a decir que no se puede localizar el elemento `.message` ("Unable to locate element using css"). Esto sucede porque Spring Boot devuelve una página por defecto cuando el endpoint no es reconcido:

![Spring Boot Error](/public/pictures/hello-world-webapp-tdd/spring-boot-error.jpg)

Por tanto, el intento de localizar un elemento con la clase `message` fracasa.

### Siguiente paso, mapear el endpoint

Bien, acaba de emerger un requisito nuevo, y es que el endpoint `/hello` necesita ser mapeado en nuestra aplicación. Para esto necesitamos guiar por tests unitarios la creación de nuestra clase controladora de Spring MVC.

{% highlight java %}
public class HelloControllerTest {
    private HelloController helloController =
            new HelloController();

}
{% endhighlight %}

Aunque en el blog no se puede reproducir correctamente lo que pasa, el problema al escribir este test es que no existe tal clase `HelloController`, por lo que falla la compilación. Para arreglar esto tenemos que crear dicha clase:

{% highlight java %}
@Controller
public class HelloController {
}
{% endhighlight %}

Seguimos:

{% highlight java %}
public class HelloControllerTest {

    private MockMvc mvc;

    private HelloController pageController = new HelloController();

    @Before
    public void setUp() throws Exception {
        mvc = MockMvcBuilders
                .standaloneSetup(pageController)
                .build();
    }

    @Test
    public void showHelloPage() throws Exception {
        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(view().name("helloView"));
    }
}
{% endhighlight %}

La clase [MockMvc](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/servlet/MockMvc.html) pertenece a las utilidades de test de Spring, y permite simular peticiones HTTP al contexto web que hayamos arrancado para nuestro test. Este contexto puede ser desde el contexto completo de nuestra aplicación hasta uno pequeñito creado a partir de un controlador único (`standaloneSetup`). Este es el cámino que estamos siguiendo en nuestro ejemplo, ya que estamos interesados en dirigir el diseño de un solo controlador, y nuestro test será mucho más rápido y eficiente siguiendo este método.

Nuestro test espera que al procesar una llamada GET al endpoint `/hello` obtengamos como repuesta un código HTTP 200 y el nombre de la vista a mostrar sea `helloView`. Si ejecutamos este test obtenemos un bonito 404:

{% highlight text %}
java.lang.AssertionError: Status
Expected :200
Actual   :404
{% endhighlight %}

Esto se debe a que de momento no estamos mapeando ese endpoint, así que vamos a hacerlo:

{% highlight java %}
@RequestMapping(method = RequestMethod.GET, value = "/hello")
public String hello() {
    return "helloView";
}
{% endhighlight %}

Y ya tenemos el test en verde.

### Vuelta al test de navegador

Veamos qué pasa cuando ejecutamos de nuevo el test funcional:

{% highlight text %}
org.thymeleaf.exceptions.TemplateInputException: Error resolving template "helloView", template might not exist or might not be accessible by any of the configured Template Resolvers
{% endhighlight %}

La excepción es clara, Thymeleaf es incapaz de encontrar una vista que se corresponda con helloView, así que tendremos que crearla, en un fichero de nombre `helloView.html`:

{% highlight html %}
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
<head>
    <title>Hello world!</title>
</head>
<body>
</body>
</html>
{% endhighlight %}

¡Ojo! En TDD no tenemos que escribir más código que el necesario para eliminar el último error generado en nuestros tests, por lo que en este caso lo único que hacemos es crear el fichero de Thymeleaf que se corresponda con la vista devuelta por nuestra clase controladora.

En la siguiente ejecución nuestro test se queja de que no ha encontrado el elemento ".message" (aunque el assertionError podría ser mejor, concretamente indica que "Expecting actual not to be null"). Por tanto, ha llegado la hora de añadir nuestro saludo inicial:

{% highlight html %}
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
<head>
    <title>Hello world!</title>
</head>
<body>
    <div class="message">Hello World</div>
</body>
</html>
{% endhighlight %}

Y ahora sí, nuestro test de aceptación se ejecuta correctamente, por lo que hemos terminado de implementar nuestro Hello World al estar pasando las condiciones de aceptación.

Para rematar la faena, ejecutemos la aplicación con el plugin Maven de Spring Boot (`mvn spring-boot:run`), y vayamos a `http://localhost:8080/hello`:

![Hello World](/public/pictures/hello-world-webapp-tdd/hello-world.jpg)

La verdad es que el resultado no es muy espectacular :), pero espero que hayáis captado la idea principal del proceso. Al tener una funcionalidad perfectamente implementada es momento de hacer commit.

### ¡Refactoring!

¿Hemos terminado? ¡No! Nos falta un paso tan importante como todos los demás, la refactorización. Cada vez que añadimos algo nuevo a nuestro sistema es hora de reflexionar, ver si hay duplicidades en el código, si podemos mejorar nuestro diseño, etc. Estas duplicidades pueden estar tanto en nuestros tests como en nuestro código de producción.

Para empezar, tenemos dos clases creadas al principio del proceso, cuya presencia es completamente redundante, se trata de `HelloWorldWebappTddApplicationIT` y `HelloWorldWebappTddApplicationTest`. Puesto que ya tenemos tests unitarios y de integración, sabemos que el framework está correctamente configurado, por lo que no es necesaria la presencia de dos clases vacías que sí tuvieron un cometido claro antes de escribir nuestros primeros tests.

La segunda duplicidad está en la clase `HelloPageIT`:

{% highlight java %}
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(
    classes = HelloWorldWebappTddApplication.class)
@WebAppConfiguration
@IntegrationTest("server.port:8080")
public class HelloPageIT extends FluentTest {
    public WebDriver webDriver = new HtmlUnitDriver();

    @Override
    public WebDriver getDefaultDriver() {
        return webDriver;
    }

    @Test
    public void openHelloPage() throws Exception {
        goTo("http://localhost:8080/hello");

        assertThat(find(".message").getText())
                .contains("Hello World");
    }
}
{% endhighlight %}

¿La véis? Se trata del puerto `8080` que está mencionado en dos sitios. Como ahora tenemos una sola clase con tests de navegador no parece un gran problema, pero pensad lo que ocurriría si nuestra suite contuviera decenas de tests...es mejor encapsular el valor del puerto en una constante, por ejemplo:

{% highlight java %}
public class EnvironmentConstants {
    public static final int SERVER_PORT = 8080;
}

@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(
    classes = HelloWorldWebappTddApplication.class)
@WebAppConfiguration
@IntegrationTest("server.port:" + SERVER_PORT)
public class HelloPageIT extends FluentTest {
    public WebDriver webDriver = new HtmlUnitDriver();

    @Override
    public WebDriver getDefaultDriver() {
        return webDriver;
    }

    @Test
    public void openHelloPage() throws Exception {
        goTo("http://localhost:" + SERVER_PORT + "/hello");

        assertThat(find(".message").getText())
                .contains("Hello World");
    }
}

{% endhighlight %}

De momento parece suficiente. En realidad se podrían discutir muchos otros detalles, por ejemplo parece fea la forma en que configuramos el parámetro del método `goTo`...pero no es este el momento de abstraer código que sólo se utiliza en un sitio.

Si nuestro sistema siguiera creciendo, a nada que añadiéramos una segunda clase con tests de navegador surgirían nuevas ideas para encapsular configuración común, pero no es ese el propósito de este post.

### Conclusión

Comenzamos el post planteando la cuestión de cuál sería el punto de partida para escribir una aplicación siguiendo la metodología TDD. Espero haber dejado clara la respuesta, que no es otra que "por los tests de aceptación, y siguiendo una aproximación top-down para escribir tests en sucesivos niveles".
