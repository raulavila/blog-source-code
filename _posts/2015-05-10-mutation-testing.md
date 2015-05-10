---
layout: post
title: Mutation testing&#58; tests de máxima calidad
permalink: 2015/05/mutation-testing
tags:
- desarrollo
- testing
comments: true
---

[Mutation testing](http://en.wikipedia.org/wiki/Mutation_testing) es una forma de evaluar la calidad de nuestros tests, que aunque según el artículo enlazado comenzó a plantearse en la década de los 70, no ha sido hasta hace relativamente poco tiempo que está adquiriendo relativa presencia en entornos empresariales. De hecho, diría que aún le falta mucho camino por recorrer para implantarse definitivamente. Espero, desde este modesto blog, aportar mi granito de arena para que su uso vaya aumentando como realmente merece.

<!--break-->

###Midiendo la calidad de los tests

La medida más extendida para medir la calidad de una suite de tests desarrollada para un sistema en concreto, es la [cobertura de código](http://es.wikipedia.org/wiki/Cobertura_de_c%C3%B3digo). Las herramientas utilizadas para generar esta medida analizan las líneas del código de producción que son ejecutadas por los tests. A más líneas ejecutadas, mayor cobertura de código.

En la siguiente captura de pantalla podemos ver el resultado de ejecutar los tests con análisis de cobertura en IntelliJ IDEA para el [proyecto de GitHub donde subo ejemplos de este blog](https://github.com/raulavila/blog-examples):

![Cobertura](/public/pictures/pitest/cobertura.jpg)

El resultado es bastante pobre, y sería intolerable para un proyecto real (en este caso me vale la excusa de que no es más que un proyecto de ejemplos :) ). En la parte izquierda se pueden ver los porcentajes de clases y líneas cubiertas por tests, y en la derecha he abierto una clase en concreto. Veréis que en el margen izquierdo se resaltan en verde o rojo las líneas de código según estén cubiertas o no.

Como norma general, en cualquier proyecto serio deberíamos perseguir el objetivo de lograr el 100% de cobertura. Aunque esto es algo extremadamente complicado, ya que determinados fragmentos de código son difíciles de cubrir por tests de forma no artificial. Uno de los ejemplos más claros son los constructores de las clases `Exception`, por ejemplo. Me atrevería a afirmar como deseable una cobertura de 90-95%.

Sin embargo, no siempre tener una gran cobertura de tests garantiza que los tests sean de calidad. Lo mejor es verlo con un ejemplo (utilizando un poco de reducción al absurdo):

{% highlight java %}
public class MaxCoverage {
    public int addOne(int number) {
        return 3;
    }
}
{% endhighlight %}

El método `addOne` debería devolver el resultado de sumar uno a su parámetro, pero devuelve 3 de forma constante. El siguiente test:

{% highlight java %}
public class MaxCoverageTest {

    private MaxCoverage maxCoverage = new MaxCoverage();

    @Test
    public void testAddOne() throws Exception {
        int result = maxCoverage.addOne(2);
        assertThat(result).isEqualTo(3);
    }
}
{% endhighlight %}

genera una cobertura del 100% en la clase MaxCoverage...¿diriáis que el test tiene una mínima calidad por ello? Por supuesto que no. De hecho a nada que añadiéramos más casos de test romperiamos la suite y deberíamos modificar la implementación del método `addOne` (en realidad esta sería la forma de desarrollar utiliando TDD, de la que hablaremos en el futuro).

###Mutation testing al rescate

¿En qué consiste Mutation testing? El concepto es bastante sencillo: consiste en realizar pequeñas modificaciones en el código de producción, conocidas como "mutaciones". Cada mutación "debería" romper algún test. Si no lo hace, la mutación "ha sobrevivido" (survived). Si rompe algún test, la mutación "ha sido asesinada" (killed). De esta forma estamos generando una nueva medida de calidad, mucho más fiable que la cobertura de código, conocida como "porcentaje de mutaciones asesinadas" (percentage of mutations killed). Este porcentaje debería ser del 100%.

Las mutaciones realizadas en el código son pequeñas modificaciones del estilo:

* Cambiar el sentido de una condición ( `if(a<0)` => `if(a>=0)`)
* Eliminar llamadas a métodos void
* Eliminar llamadas a métodos con retorno, y utilizar un valor por defecto en su lugar (0 para int, etc)
* Operaciones matemáticas: se mutan operaciones matemáticas reemplazando el operador +  por -, etc

En [este link](http://pitest.org/quickstart/mutators/) hay una lista completa de los mutadores utilizados por la herramienta que vamos a utilizar para mostrar un ejemplo práctico de desarrollo utilizando Mutation Testing: [PIT](http://pitest.org/).

###Mutation testing en la práctica

Configuremos nuestro proyecto para generar reportes de Mutation Testing con PIT. Optaremos por utilizarlo a través de su plugin de maven, para lo cual tenemos que añadir el siguiente fragmento de código al fichero POM:

{% highlight xml %}
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
    <version>1.1.5</version>
    <configuration>
        <targetClasses>
            <param>com.raulavila.mutationtesting*</param>
        </targetClasses>
        <targetTests>
            <param>com.raulavila.mutationtesting*</param>
        </targetTests>
    </configuration>
</plugin>
{% endhighlight %}

`targetClasses` indica las classes que serán mutadas (lo que muta realmente es su bytecode), y `targetTests` indica los tests que componen la suite y deben poner a prueba el código mutado. El plugin ofrece muchísimos parámetros de configuración, pero por simplicidad analizaremos un solo package utilizando los parámetros por defecto ([convention over configuration](http://en.wikipedia.org/wiki/Convention_over_configuration)).

Para generar los informes hay que lanzar el siguiente goal de maven: `mvn clean verify org.pitest:pitest-maven:mutationCoverage`, y los podremos encontrar dentro de la carpeta target/pit-reports (aunque en la salida por consola de maven también aparecerá un resumen).

En nuestro ejemplo crearemos e iremos evolucionando una calculadora. La primera versión es algo tan simple como:

{% highlight java %}
public class Calculator {

    public long add(int operand1, int operand2) {
        return operand1 + operand2;
    }
}
{% endhighlight %}

####Test defectuoso: falta de Assertions

Vamos a ir creando tests que en todo momento generarán una cobertura del 100%, pero que son claramente fallidos. PIT nos ayudará a detectar sus carencias y mejorarlos paulatinamente.

La primera clase de test es:

{% highlight java %}
public class CalculatorTest {

    private Calculator calculator = new Calculator();

    @Test
    public void testAdd() throws Exception {
        int operand1 = 2;
        int operand2 = 2;

        long sum = calculator.add(operand1, operand2);
    }
}
{% endhighlight %}

El fallo aquí es claro (además de que lo he mencionado en el título :) ): faltan assertions. Aunque aquí es bastante evidente, en sistemas grandes puede que se nos escape verificar algún valor dentro de un mar de assertions, cosa que PIT destapará sin problemas, como podemos ver en el informe:

![Informe pitest](/public/pictures/pitest/pitest1.jpg)

En esta vista general destaca cómo esta herramienta también realiza informes de cobertura. De hecho, si una mutación deja código sin cobertura nos lo hará saber. También podemos ver como todas las mutaciones han sobrevivido, cosa totalmente normal, puesto que no hay ningún assertion que compruebe el resultado de la llamada al método `add`. En el informe detallado de la clase vemos qué mutaciones se han puesto a prueba:

![Informe pitest](/public/pictures/pitest/pitest2.jpg)

Una de ellas ha modificdo el operador (+ por -), y la otra ha incrementado en 1 el retorno de la función.

Vamos a arrelgar nuestra clase de test:

{% highlight java %}
@Test
public void testAdd() throws Exception {
    int operand1 = 2;
    int operand2 = 2;

    long sum = calculator.add(operand1, operand2);

    assertThat(sum).isEqualTo(4);
}
{% endhighlight %}

Ahora sí hemos conseguido hacer feliz a PIT:

![Informe pitest](/public/pictures/pitest/pitest3.jpg)

La verdad es que este test no es perfecto. Desde mi punto de vista, cuando testeamos clases que trabajan con operandos, el enfoque de [Data Driven Testing](http://es.wikipedia.org/wiki/Data-driven_testing) es mucho más adecuado. Por ello lo utilizaremos en breve.

####Test defectuoso: casos de test insuficiente

Vamos a evolucionar nuestra calculadora. El método add recibirá un nuevo parámetro de tipo `Mode` (de tipo enumerado), de forma que si su valor es `ABSOLUTE`, se realizará una suma de los valores absolutos de los operandos:

{% highlight java %}
public class Calculator {

    public long add(int operand1, int operand2, Mode mode) {

        if (mode == ABSOLUTE) {
            operand1 = Math.abs(operand1);
            operand2 = Math.abs(operand2);
        }

        return operand1 + operand2;
    }

    public static enum Mode {ABSOLUTE, STRAIGHT;}
}
{% endhighlight %}

El test se mantiene, con la única modificación del parámetro:

{% highlight java %}
@Test
public void testAdd() throws Exception {
    int operand1 = 2;
    int operand2 = 2;

    long sum = calculator.add(operand1, operand2, ABSOLUTE);

    assertThat(sum).isEqualTo(4);
}
{% endhighlight %}

Este test es exitoso y da una cobertura del 100%, pero de nuevo, PIT nos desvela sus carencias:

![Informe pitest](/public/pictures/pitest/pitest4.jpg)

Lo que ha ocurrido aquí es que el test ha sobrevivido a la omisión de pasar los operandos a su valor absoluto. Esto es debido a que no hemos creado una suite de test cubriendo las diferentes combinaciones de datos que pueden dar lugar a escenarios diferentes (en este caso, números negativos). Vamos a solucionar esto utilizando tests parametrizados de JUnit ([más información aquí](https://github.com/junit-team/junit/wiki/Parameterized-tests)):

{% highlight java %}
@RunWith(Parameterized.class)
public class CalculatorTest {

    private Calculator calculator = new Calculator();

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
    }

    @Test
    public void testAddAbsolute() throws Exception {
        long sum = calculator.add(operand1, operand2, ABSOLUTE);
        assertThat(sum).isEqualTo(expectedResultAbsolute);
    }

}
{% endhighlight %}

En esta ocasión ponemos a prueba datos de diferente naturaleza y en ambos modos de ejecución. De nuevo, PIT vuelve a ser feliz, dando un 100% de mutaciones asesinadas.

####Test defectuoso: verificación de colaboraciones

Por último, añadamos a nuestra calculadora la funcionalidad de auditoría, de forma que todas las llamadas al método add sean registradas en algún sitio (que no importa demasiado a efectos de este post):

{% highlight java %}
public class Calculator {

    private Audit audit = new AuditImpl();

    public long add(int operand1, int operand2, Mode mode) {
        audit.register(String.format("%d + %d (%s)", operand1, operand2, mode));

        if (mode == ABSOLUTE) {
            operand1 = Math.abs(operand1);
            operand2 = Math.abs(operand2);
        }

        return operand1 + operand2;
    }

    public static enum Mode {ABSOLUTE, STRAIGHT;}
}
{% endhighlight %}

La interfaz Audit y su implementación son bastante triviales, podéis verlas [en el repositorio de GitHub](https://github.com/raulavila/blog-examples/tree/master/src/main/java/com/raulavila/mutationtesting).

Si mantenemos la última versión de la suite de tests, la cobertura seguirá siendo del 100%, pero PIT encontrará un defecto:

![Informe pitest](/public/pictures/pitest/pitest5.jpg)

Lo que ha ocurrido aquí es que la interacción con Audit se ha eliminado, ¡¡¡y los tests no se han quejado de nada!!! Es evidente que nos hemos olvidado de **verificar las interacciones con los colaboradores**. Esto se puede conseguir fácilmente con un framework como [Mockito](http://mockito.org/), pero hay un problema, y es que el diseño de la clase `Calculator` no facilita la creación de mocks en tests unitarios. Supongo que ya habréis caído en la cuenta de que es necesario modificar nuestra clase para que pueda configurarse mediante [inyección de dependencias](/2015/03/principios-dependencias/):

{% highlight java %}
public class Calculator {

    private Audit audit;

    public Calculator(Audit audit) {
        this.audit = audit;
    }
    //....
}
{% endhighlight %}

Por último, esta sería la versión final de nuestra clase de test, verificando las colaboraciones mediante Mockito:

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

Con lo que, de nuevo, hemos conseguido que PIT nos de un 100% de cobertura y de porcentaje de mutaciones asesinadas :). Podría seguir explorando aún más las posibilidades de esta utilidad, pero creo que este ejemplo es más que suficiente para descubrir todo su potencial.

El único punto flaco que le encuentro es que, debido al proceso que es necesario ejecutar para mutar el código, en sistemas con gran número de tests y miles de líneas de código, el tiempo necesario para generar un informe completo es elevado. Pero algo así no debería ser problema si tenemos programados builds nocturnos en nuestros sistema de integración continua, ¿verdad?
