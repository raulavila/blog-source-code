---
layout: post
title: Novedades en Spring 4 (parte 1)
description: Novedades en Spring Framework 4, primera parte
permalink: 2015/02/spring-4-novedades/
tags:
- Java
- Spring
comments: true
---

De todos los frameworks que existen dentro del mundo Java, posiblemente Spring sea el más popular. Rara es la oferta de trabajo en la que busquen Desarrolladores Java y Spring no aparezca en el stack requerido para el puesto.

Lo que empezó principalmente como un motor de [inyección de dependencias](http://en.wikipedia.org/wiki/Dependency_injection), ha ido derivando en un ecosistema de diferentes plataformas que facilitan el desarrollo de aplicaciones de todos los tipos imaginables. Entre los módulos más populares de Spring se encuentran:

<!--break-->

* [Spring MVC](https://spring.io/guides/gs/serving-web-content/), que facilita la creación de aplicaciones web, API's REST, etc. Suele ser el punto de entrada a Spring para casi todo el mundo
* [Spring Data](http://projects.spring.io/spring-data/), permite la creación de capas de persistencia de forma extremadamente sencilla, ¡llegando incluso a evitarnos la implementación de los métodos!, que es generada por Spring siguiendo convenciones en la nomenclatura. Personalmente creo que la mayor baza de este módulo es su uso con bases de datos [NoSQL](http://es.wikipedia.org/wiki/NoSQL)
* [Spring Batch](http://projects.spring.io/spring-batch/), para la creación de, sorpresa, procesos Batch. Trabajé con Spring Batch para un proyecto, y creo que el enfoque que obliga a dar a la arquitectura de estos procesos es perfecto
* [Spring Social](http://projects.spring.io/spring-social/), para generar aplicaciones que se conecten con redes sociales
* [Spring Security](http://projects.spring.io/spring-security/), para añadir sistemas de autenticación (login) y autorización (permisos por roles, etc) a nuestras aplicaciones web
* [Spring Boot](http://projects.spring.io/spring-boot/), proyecto relativamente más reciente, con el que apenas he tomado contacto, hasta que hace poco asistí a un Webcast impartido por [Josh Long](https://twitter.com/starbuxman) y quedé gratamente sorprendido. Creo que profundizaré en Spring Boot durante este año, pero en resumen permite el desarrollo rápido de aplicaciones mediante la filosofía [convention over configuration](http://en.wikipedia.org/wiki/Convention_over_configuration)

La lista podría seguir, pero creo que basta para hacerse una idea de todo lo que Spring abarca.

## Spring 4

A primeros de 2014 se liberó la versión 4 de Spring, que introduce varias mejoras sobre la última release dentro de la versión 3 (3.2.13). En este post revisaré las más importantes en el Core de Spring, como siempre con ejemplos prácticos que [podéis descargar de GitHub](https://github.com/raulavila/spring-4-new-features).

### Expresiones Lambdas

Las [Lambda Expressions](http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) son una de las grandes mejoras de Java 8. Spring 4 por supuesto ha adaptado sus API's para soportarlas, permitiéndonos ser más concisos en el código. Un ejemplo muy claro de la diferencia entre utilizar y no utilizar expresiones Lambda lo tenemos en el siguiente ejemplo, donde se consulta una base de datos mediante [JdbcTemplate](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html) sin y con expresiones lambda:

{% highlight java %}
private static void queryDB(JdbcTemplate jdbcTemplate) {
    System.out.println("Java 7 Querying for customer records " +
                            "where first_name = 'Bart':");

    List<Customer> results = jdbcTemplate.query(
            "select * from customers " +
                    "where first_name = ?",
            new Object[] { "Bart" },
            new RowMapper<Customer>() {
                @Override
                public Customer mapRow(ResultSet rs,
                                       int rowNum) throws SQLException {
                    return new Customer(rs.getLong("id"),
                                        rs.getString("first_name"),
                                        rs.getString("last_name"));
                }
            });

    processCustomers(results);
}

private static void queryDB_Java8(JdbcTemplate jdbcTemplate) {
    System.out.println("Java 8 Querying for customer records " +
                            "where first_name = 'Bart':");
    List<Customer> results = jdbcTemplate.query(
            "select * from customers " +
                    "where first_name = ?",
            new Object[] { "Bart" },
            (rs, rowNum) -> new Customer(rs.getLong("id"),
                                    rs.getString("first_name"),
                                    rs.getString("last_name")));

    processCustomers(results);
}

{% endhighlight %}

Hemos eliminado toda la verbosidad que conlleva la creación de una instancia de RowMapper, y la expresión queda mucho más clara. El método query seguirá recibiendo una objeto del tipo RowMapper, pero nos quitamos la responsabilidad de crearlo nosotros (nos quitamos [boilerplate code](http://en.wikipedia.org/wiki/Boilerplate_code) vaya).


### Generics con @Autowired

La anotación `@Autowired`es omnipresente en proyectos con Spring, y se utiliza para inyectar dependencias de forma automática. Es decir, Spring busca en su contexto un bean que encaje con el tipo del atributo que está asociado a la anotación:

{% highlight java %}
@Autowired
PersonDAO personDAO;
{% endhighlight %}

En el ejemplo, sería necesario que existiera en el contexto un bean del tipo PersonDAO, o Spring fallaría al intentar levantar el contexto. Personalmente considero que en ocasiones es mejor configurar el contexto de Spring con sus dependencias en ficheros XML, pero eso es discusión para otro momento.

Hasta la versión 4 de Spring, si el tipo del campo al que queremos inyectar la dependencia era genérico no podíamos hacerlo:

{% highlight java %}
public interface Store<T> { /*..*/}

@Component
public class IntStore implements Store<Integer> { /*..*/}

@Component
public class StringStore implements Store<String> { /*..*/}

@Autowired
private Store<Integer> store;
{% endhighlight %}

En Spring 3 obtendríamos la excepción "No qualifying bean of type [Store] is defined, expected single matching bean but found 2: stringStore, intStore", mientras que en Spring 4 esto es perfectamente posible, lo que nos permite desarrollar siguiendo el patrón ["program to an interface, not an implementation"](http://stackoverflow.com/questions/383947/what-does-it-mean-to-program-to-an-interface).

### Inyección ordenada

En Spring 3 era posible inyectar una seria de beans que implementaran una interfaz en particular, pero no era posible garantizar el orden de la inyección:

{% highlight java %}
public interface GreetingService {
    String getGreeting();
}

public class MultiGreetingPrinter {

    @Autowired
    private List<GreetingService> greetingServices;

    public void printGreeting() {
    	for(GreetingService greetingService: greetingServices) {
    		System.out.println(greetingService.getGreeting());
    	}

    }

}
{% endhighlight %}

En el ejemplo, si tenemos varios objetos de diferentes clases que implementan la interfaz GreetingService en el contexto de Spring, todos ellos serán inyectadas en la lista, pero si queremos establecer prioridades en el orden en que dichas instancias serán invocadas (por ejemplo, para implementar el patrón [Chain of Responsibility](http://en.wikipedia.org/wiki/Chain-of-responsibility_pattern)) no había forma de hacerlo. Esto se ha solucionado en Spring 4 añadiendo la interfaz `Ordered` que solo expone un método, `getOrder`:

{% highlight java %}

public interface Ordered {
    int getOrder();
}

{% endhighlight %}

Nuestras implementaciones de GreetingService deberán implementar también la interfaz Ordered, y los objetos serán inyectados siguiendo el orden dictado por `getOrder`:

{% highlight java %}
public class PersonGreeting implements GreetingService, Ordered {
    /*..*/

    private int order;

    @Override
    public int getOrder() {
    	return order;
    }

    public void setOrder(int order) {
        this.order = order;
    }
}

{% endhighlight %}

De esta forma podemos crear los beans en el contexto estableciendo su orden:

{% highlight xml %}
<bean id="personGreeting" class="com.raulavila.spring.greeting.PersonGreeting">
	<property name="personName" value="Raul" />
	<property name="order" value="2" />
</bean>
{% endhighlight %}

Incluso podemos configurar ese orden por base de datos o un fichero de propiedades, siempre que getOrder retorne el valor correcto.

### Configuración del contexto

La configuración del contexto de Spring utilizando ficheros XML ha tenido sus voces críticas, aunque incido en que personalmente la prefiero al uso de anotaciones. No obstante, Spring ha ido introduciendo alternativas a la configuración XML durante los últimos años, y actualmente es posible generar un contexto mediante la configuración establecida en una clase Java anotada con `@Configuration` y `@ComponentScan`:

{% highlight java %}
@Configuration
@ComponentScan
public class MyApplicationContext {

	@Bean(name="intStore")
	@Description("Description of the bean")
	public Store<Integer> getIntStore() {
		return new IntStore();
	}

	@Bean(name="helloWorld")
	@Conditional(NoGreetingServiceDefined.class)
	public GreetingService createGreetingService(){
		return new GreetingService() {
			@Override
			public String getGreeting() {
				return "Hello world!!";
			}
		};
	}

}
{% endhighlight %}

En el ejemplo vemos como la clase contiene dos métodos "factory" que se encargan de instanciar los beans que residirán en el contexto y pueden ser inyectados. Esta clase no supone ninguna novedad en Spring 4, pero sí dos de las anotaciones que aparecen:

#### @Description

Bastante autoexplicativa, asocia una descripción al Bean. Es especialmente útil cuando los beans son expuestos externamente, utilizando [JMX](http://en.wikipedia.org/wiki/Java_Management_Extensions), por ejemplo, de esta forma podemos proporcionar información adicional a soporte de aplicaciones.

#### @Conditional

Opción bastante interesante, como su propio nombre indica condiciona la creación del bean al resultado (booleano, claro) devuelto por la clase indicada en la anotación, y que debe implementar la interfaz `Condition`:

{% highlight java %}
public class NoGreetingServiceDefined implements Condition{

	@Override
	public boolean matches(ConditionContext context,
			AnnotatedTypeMetadata metadata) {

		return context.getBeanFactory()
						.getBeansOfType(GreetingService.class)
						.isEmpty();
	}

}
{% endhighlight %}

En este ejemplo se revisa el contexto para comprobar si ya existe algún bean de la clase GreetingService, y en tal caso se evitaría la creación de uno nuevo. Bastante útil cuando tenemos la configuración repartida en diferentes ficheros y pueden existir duplicidades.

### Configuración del contexto mediante Groovy DSL

No soy un gran fan de [Groovy](http://groovy-lang.org/), y más desde la irrupción de Java 8, pero su auge es evidente. Spring 4 permite la configuraciónd el contexto mediante ficheros de definición Groovy:

{% highlight groovy %}
beans {
    framework String, 'Grails'
    foo String, 'hello'
    bar(Bar,s:'hello',i:123)
}
{% endhighlight %}

Cada línea dentro del scope beans define un bean mediante los parámetros "name tipo parámetros_del_constructor". La verdad es que Groovy aporta una gran flexbilidad, pero yo siempre he sido más de tipado estático, lo siento :). [Más información aquí](http://spring.io/blog/2014/03/03/groovy-bean-configuration-in-spring-framework-4).

Terminamos aquí el repaso a las mejoras del core. [En el próximo post](/2015/03/spring-4-novedades-2) hablaré de las mejoras introducidas a nivel de desarrollo Web.
