---
layout: post
title: Spock vs JUnit
permalink: 2016/03/spock-vs-junit/
tags:
- desarrollo
- testing
- Spock
comments: true
---

[JUnit](http://junit.org/) es, de facto, el framework para testear código Java. No creo que nadie le retire su trono a corto plazo, [JUnit Lambda](http://junit.org/junit-lambda.html) está cerca de ser liberado, por lo que las nuevas funcionalidades de Java 8 serán perfectamente soportadas, cosa que en los dos últimos años ha sido un ligero hándicap.

Sin embargo, recientemente está irrumpiendo con relativa fuerza un framework, que inicialmente se ideó para testear código en lenguage [Groovy](http://www.groovy-lang.org/), pero dado que Groovy corre sobre la JVM se puede utilizar igualmente para testear código Java.

<!--break-->

##Spock

[Spock](http://spockframework.github.io/spock/docs/1.0/index.html) es el acrónimo de "Specification and Mock". Sabiendo esto es fácil deducir lo que nos permite hacer: crear especificaciones de nuestros sistemas, añadiendo capacidades para generar Mocks de dependencias. Digo especificaciones y no tests, cosa que puede resultar algo confusa de inicio, pero que se entenderá bien con los ejemplos. En líneas generales, una especificación no es más que una clase de test vitaminada.

Mi intención en este artículo es mostrar con un ejemplo claro en qué se diferencia Spock de JUnit. No quiero entrar en excesivos detalles de todo lo que Spock ofrece, para eso ya tenéis [la documentación oficial](http://spockframework.github.io/spock/docs/1.0/index.html).

##La clase a testear

Vamos a crear tests para la una clase `Calculator`, clase que ya utilizamos en el post sobre [Mutation Testing](/2015/05/mutation-testing). La versión "final", de la que queremos generar tests, es esta:

{% highlight java %}
public class Calculator {

    private Audit audit;

    public Calculator(Audit audit) {
        this.audit = audit;
    }

    public long add(int operand1, int operand2, Mode mode) {
        audit.register(String.format("%d + %d (%s)",
                        operand1,
                        operand2,
                        mode));

        if (mode == Mode.ABSOLUTE) {
            operand1 = Math.abs(operand1);
            operand2 = Math.abs(operand2);
        }

        return operand1 + operand2;
    }

    public static enum Mode {ABSOLUTE, STRAIGHT;}
}
{% endhighlight %}

Las funcionalidades de esta clase (que no es un derroche de buenas prácticas, por otro lado) son claras.

##Tests en JUnit

Esta sería la clase de Test en JUnit que cubriría convenientemente todas las funcionalidades de `Calculator`:

{% highlight java %}
@RunWith(Parameterized.class)
public class CalculatorTest {

    @Mock
    private Audit audit;
    @InjectMocks
    private Calculator calculator;

    @Before
    public void setUp() throws Exception {
        MockitoAnnotations.initMocks(this);
    }

    private int operand1;
    private int operand2;
    private long expectedResultStraight;
    private long expectedResultAbsolute;

    @Parameterized.Parameters
    public static Collection data() {
        Object[][] data = new Object[][] {
                { 2, 2, 4, 4 },
                { -2, 2, 0, 4 },
                { -3, -3, -6, 6 },
                { 0, 0, 0, 0 }
        };

        return Arrays.asList(data);
    }

    public CalculatorTest(int operand1,
                          int operand2,
                          long expectedResultStraight,
                          long expectedResultAbsolute) {

        this.operand1 = operand1;
        this.operand2 = operand2;
        this.expectedResultStraight = expectedResultStraight;
        this.expectedResultAbsolute = expectedResultAbsolute;
    }


    @Test
    public void testAddStraight() throws Exception {
        long sum = calculator.add(operand1, operand2, STRAIGHT);
        assertThat(sum).isEqualTo(expectedResultStraight);

        verify(audit).register(
                String.format("%d + %d (STRAIGHT)", operand1, operand2));
    }

    @Test
    public void testAddAbsolute() throws Exception {
        long sum = calculator.add(operand1, operand2, ABSOLUTE);
        assertThat(sum).isEqualTo(expectedResultAbsolute);

        verify(audit).register(
                String.format("%d + %d (ABSOLUTE)", operand1, operand2));
    }

}
{% endhighlight %}

Vemos varios problemas aquí:

* Estamos añadiendo dos librerías encima de JUnit para mejorar nuestros tests: [Fest assertions](https://github.com/alexruiz/fest-assert-2.x), para dar mayor expresividad a nuestros asserts ([Harmcrest](http://hamcrest.org/) habría sido una alternativa igualmente válida), y [Mockito](http://mockito.org/) para crear Mocks de dependencias
* Para utilizar Data Driven Testing de forma adecuada necesitamos que nuestros tests sean ejecutados por el runner `Parameterized` en lugar del runner JUnit por defecto
* La cantidad de código boilerplate que hay que escribir fuera de nuestros métodos de tests es grande, ya que necesitamos preparar la tabla de datos de una forma algo fea (en el método `data`), necesitamos un método `setUp` para inicializar dependencias, un constructor para inicializar los datos de cada test, etc

Todos estos contrapuntos no son malos en sí, pero ya sabemos que Java nunca se ha caracterizado por ser demasiado conciso. Esa verbosidad puede ser muy buena en determinadas ocasiones, ya que fuerza buenas prácticas y pensar de manera adecuada en nuestros diseños. Pero a la hora de escribir tests, ¿no sería mejor ahorrarse estos pequeños inconvenientes?

##Spock al rescate

Vamos a sumergirnos de lleno en Spock viendo cómo sería la implementación final de nuestra especificación para la clase `Calculator`:

{% highlight java %}
class CalculatorSpec extends Specification {

    Audit audit = Mock()

    @Subject
    Calculator calculator = new Calculator(audit)

    def "Calculator can add operands in straight mode"() {
        when: "We add two operands in straight mode"
        long sum = calculator.add(operand1, operand2, STRAIGHT)

        then: "The result of the sum matches the expected one"
        sum == expectedResult

        where:
        operand1 | operand2 || expectedResult
        2        | 2        || 4
        -2       | 2        || 0
        -3       | -3       || -6
    }

    def "Calculator can add operands in absolute mode"() {
        when: "We add two operands in absolute mode"
        long sum = calculator.add(operand1, operand2, ABSOLUTE)

        then: "The result of the sum matches the expected one"
        sum == expectedResult

        where:
        operand1 | operand2 || expectedResult
        2        | 2        || 4
        -2       | 2        || 4
        -3       | -3       || 6
    }

    def "Audit object intercepts all calls to the Calculator"() {
        when: "We add two operands in any mode"
        calculator.add(2, 2, ABSOLUTE)
        calculator.add(2, 2, STRAIGHT)

        then: "The Audit object registers the call"
        1 * audit.register("2 + 2 (ABSOLUTE)")
        1 * audit.register("2 + 2 (STRAIGHT)")
    }
}
{% endhighlight %}

Vemos que:

* El test está escrito en Groovy
* El nombre de la clase de test tiene el sufijo `Spec` y extiende a la clase `Specification`. El nombre especificación viene de que nuestra clase no solamente está testeando código, también está generando especificaciones legibles por un ser humano
* Podemos crear un Mock de una clase invocando al método `Mock`, que forma parte del framework
* La anotación `Subject` nos indica cuál es el sujeto que estamos especificando
* Nuestros métodos pueden ser nombrados con una cadena de caracteres libre, sin seguir ningún tipo de convención impuesta por lenguajes de programación. Esto hace mucho más fácil describir qué vamos a testear sin temor a crear nombres de métodos demasiado largos e ilegibles
* Los métodos de test están divididos en secciones claras, siguiendo el patrón [Arrange Act Assert](http://c2.com/cgi/wiki?ArrangeActAssert), o "Given When Then". En este ejemplo, al ser tan sencillo, no hay necesidad de preparar nada en la sección `given:`, por lo que la omitimos
* Cada sección puede tener su descripción, resumiendo qué estamos preparando, ejecutando o verificando
* La sección `then:` contiene assertions que será evaluadas utilizando [Groovy Truth](http://groovy-lang.org/semantics.html#Groovy-Truth). No hay necesidad por tanto de utilizar métodos assert, tan solo expresiones booleanas
* Las verificaciones de los mocks son muy claras: `1 * audit.register("2 + 2 (ABSOLUTE)")` verifica que estamos llamando una sola vez al método `register` de `audit` con esos parámetros, y fallará en caso contrario. Simular comportamientos en los Mocks es muy sencillo también, para profundizar nada mejor que la [documentación oficial](http://spockframework.github.io/spock/docs/1.0/interaction_based_testing.html)
* Las tablas de datos para utilizar Data Driven Testing son extremadamente claras, según podemos ver en las secciones `where:`. Nada de crear arrays de dos dimensiones, y campos en la clase para soportarlo

Reseñar por último que la forma en que el framework muestra errores en assertions es muy expresiva, indicando de forma gráfica donde está el error, los valores de todas la variables implicadas, etc. Aquí tenemos un ejemplo:

{% highlight text %}
Condition not satisfied:

sum == expectedResult
|   |  |
4   |  5
    false
{% endhighlight %}

## Spock vs JUnit

No pretendo convencer a nadie de si un framework es mejor que el otro, tan sólo comparar las diferencias entre ambos. Los beneficios de usar Spock son claros, pero también es cierto que obliga a añadir un lenguage nuevo a nuetro stack tecnológico, y eso no siempre es bueno, desde el punto de vista de la mantenibilidad, el aprendizaje por parte del equipo, etc.

Además, utilizar Spock en un proyecto Maven hace necesario configurar nuestro fichero POM de manera adecuada, saltándonos ligeramente las convenciones (más información [aquí](http://www.petrikainulainen.net/programming/testing/writing-unit-tests-with-spock-framework-creating-a-maven-project/)). Si utilizamos [Gradle](/2015/09/gradle-desde-maven) es bastante más fácil.

Por último, el hecho de que este año se vayan a publicar dos libros sobre la materia (aquí tenéis [uno](http://www.amazon.es/Spock-Running-Writing-Expressive-Groovy/dp/1491923296/ref=sr_1_1) y el [otro](https://www.manning.com/books/java-testing-with-spock)) da una idea de la difusión que Spock está teniendo dentro de la comunidad.

(El código, como siempre, en [GitHub](https://github.com/raulavila/gradle-spock-intro)).
