---
layout: post
title: Tests de contrato
permalink: 2015/11/test-de-contrato
tags:
- desarrollo
- testing
- TDD
comments: true
---

Prosiguiendo con mi inmersión en el mundo del [TDD](/2015/08/primera-experiencia-tdd), hace poco he finalizado un curso online que me ha parecido excepcionalmente bueno. Impartido por [JB Rainsberger](https://en.wikipedia.org/wiki/J._B._Rainsberger), y con el modesto título de ["World's best intro to TDD"](http://www.jbrains.ca/permalink/the-worlds-best-intro-to-tdd-demo-video), durante más de 20 horas de vídeo, nos desgrana diferentes acercamientos (approaches que llaman) al desarrollo de software utilizando TDD.

Mi intención en este blog es ir hablando de estos acercamientos en el futuro, además de otras cosas que he aprendido en el curso. En cualquier caso, nada como realizarlo si estáis interesados en el tema (su precio, 97 dólares, me parece **barato** después de haberlo hecho, creedme). Comenzaré con una técnica que me ha gustado especialmente a la hora de trabajar con interfaces, conocida como "Tests de contrato" (o contract tests).

<!--break-->

##Qué es un contrato. Sintáxis y semántica.

Como todos sabemos, si programamos con interfaces crearemos mejores diseños, más encapsulados y cohesionados, y menos acoplados. Si no sabéis de lo que hablo, [ya tardáis](http://stackoverflow.com/questions/383947/what-does-it-mean-to-program-to-an-interface).

Un contrato no es más que una interfaz (y viceversa :)). El contrato contiene una **sintáxis** y una **semántica**. Veamos un ejemplo:

{% highlight java %}
public interface Calculator {
    long getSumFor(String expression);
}
{% endhighlight %}

En el ejemplo hemos creado una calculadora que evalúa expresiones en cadenas de caracteres. La sintáxis del contrato nos dice que tenemos un método `getSumFor` que recibe como entrada una expresión `String` y devuelve un número de tipo `long`.

No tenemos más información, por tanto, sin la documentación adecuada, ¿sabríamos utilizar esta interfaz sin ver la implementación? En una de las modalidades de TDD se alienta el uso de interfaces mediante el uso de Mocking frameworks (por ejemplo, [Mockito](http://mockito.org/)) en nuestros tests. Es decir, cuando creamos la especificación de una clase con colaboradores, deberemos añadir dichos colaboradores como interfaces al test, y "mockear" su comportamiento. Veamos un ejemplo, de una supuesta clase controladora que utilizaría nuestra calculadora para evaluar expresiones introducidas por el usuario:

{% highlight java %}
@Test
public void evaluateUserExpression() throws Exception {
    //Given
    Display display = mock(Display.class);
    Calculator calculator = mock(Calculator.class);
    Controller controller = new Controller(display, calculator);
    when(calculator.getSumFor("1,2")).thenReturn(3L);

    //When
    controller.onExpression("1,2");

    //Then
    verify(display).showResult(3);
}
{% endhighlight %}

En el ejemplo estamos testeando un [MVC](https://es.wikipedia.org/wiki/Modelo%E2%80%93vista%E2%80%93controlador), donde la vista es la clase `Display`, y el modelo es la clase `Calculator`. Creo que el código es bastante autoexplicativo, espero que quede claro el hecho de que tanto `Display` como `Calculator` son interfaces, mientras que `Controller` es el "subject under test".

Bien, la línea `when(calculator.getSumFor("1,2")).thenReturn(3L)` nos está desvelando parte de la **semántica** de la clase `Calculator`. Ya sabemos en parte que las expresiones son una lista de números separados por comas, y que el método `getSumFor` devolverá un número conteniendo la suma de todos esos números.

El problema es que en la clase `Controller` estamos testeando más bien las interaciones entre componentes, y no la semántica de `Display`, que seguramente contenga muchos casos límite y de error. Aquí es donde surge el concepto que da título a este post.

##Tests de Contrato

Un test de contrato es una conjunto de especificaciones de la semántica de una interfaz / contrato. Esta especificación se implementará en una clase abstracta de tests, que contendrá uno o más métodos abstractos de tipo factory, encargados de crear las implementaciones finales que deberán cumplir nuestro contrato. Esto se entenderá mucho mejor con el ejemplo:

{% highlight java %}
public abstract class CalculatorContract {

    private Calculator calculator;

    protected abstract Calculator getCalculatorImpl();

    @Before
    public void setup() {
        calculator = getCalculatorImpl();
    }

    @Test
    public void shouldReturnZero_GivenEmptyString() throws Exception {
        assertThat(calculator.getSumFor("")).isEqualTo(0);
    }

    @Test
    public void shouldReturnZero_GivenZeros() throws Exception {
        assertThat(calculator.getSumFor("0,0")).isEqualTo(0);
    }

    @Test
    public void shouldReturnTheNumber_GivenOneAddend() throws Exception {
        assertThat(calculator.getSumFor("2")).isEqualTo(2);
    }

    @Test
    public void shouldReturnTheCorrectSum_GivenTwoAddend() throws Exception {
        assertThat(calculator.getSumFor("1,2")).isEqualTo(3);
    }
}
{% endhighlight %}

Aquí tenemos el test de contrato para la clase `Calculator`. A fines didácticos asumo que la expresión no será nula, no contendrá caracteres diferentes a dígitos, etc. Para que quede bien claro, en la práctica esta clase contendría muchos más métodos especificando múltiples casos de error.

En el ejemplo vemos como hay un método factory `getCalculatorImpl` que se encarga de devolver el "subject under test", pero al ser abstracto, ¡hará imposible ejecutar nuestros tests! Por este motivo este no es un test al uso, sino una forma de dejar claro qué espera nuestro sistema de las clases que implementen `Calculator`.

Vamos, por tanto, a crear una implementación de nuestro contrato:

{% highlight java %}
public class CalculatorLegacyImpl implements Calculator {
    @Override
    public long getSumFor(String expression) {
        if(StringUtils.isEmpty(expression))
            return 0;

        String[] addends = expression.split(",");

        long result = 0;

        for (String addend : addends) {
            int number = Integer.parseInt(addend);
            result = result + number;
        }

        return result;
    }
}
{% endhighlight %}

¿Diríais que esta implementación es correcta, basada en las especificaciones? Para ello lo que tendremos que hacer es crear una clase de test que extienda `CalculatorContract` e implemente el método `getCalculatorImpl`:

{% highlight java %}
public class CalculatorLegacyImplTest extends CalculatorContract {

    @Override
    protected Calculator getCalculatorImpl() {
        return new CalculatorLegacyImpl();
    }
}
{% endhighlight %}

Esta clase ya no es abstracta, y por tanto podremos ejecutar los tests, que, efectivamente, son existosos.

##Añadiendo nuevas implementaciones del contrato

Java 8 nos facilita mucho la vida para trabajar con listas y colecciones, gracias a las [Lambda expressions](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) y [Streams](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html). Hagamos uso de ello en una nueva implementación:

{% highlight java %}
@Override
public long getSumFor(String expression) {
    if(StringUtils.isEmpty(expression))
        return 0;

    String[] addends = expression.split(",");

    return Arrays.stream(addends)
            .mapToInt(Integer::parseInt)
            .sum();
}
{% endhighlight %}

Espero que mentalmente ya hayáis anticipado el siguiente paso, que en efecto será crear una nueva implementación de `CalculatorContract`:

{% highlight java %}
public class CalculatorLambdaImplTest extends CalculatorContract{
    @Override
    protected Calculator getCalculatorImpl() {
        return new CalculatorLambdaImpl();
    }
}
{% endhighlight %}

Con este último ejemplo finalizamos el post. El ejemplo utilizado ha sido extremadamente sencillo, pero en la práctica nuestros métodos factory es posible que requieran parámetros para instanciar el "subject under test", o incluso que necesitemos varios métodos factory simulando condiciones diferentes (fixtures). Espero, al menos, que la idea haya quedado bien clara.

([Los ejemplos, como siempre, en GitHub](https://github.com/raulavila/blog-examples/tree/master/src/test/java/com/raulavila/tdd/contracttests)).
