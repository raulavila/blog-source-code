---
layout: post
title: Eliminando dependencias estáticas en legacy code
permalink: 2015/03/refactoring-legacy-static
tags:
- desarrollo
- refactoring
- legacy
comments: true
---

Modificar un sistema legacy es, quizás, una de las labores más desagradables de nuestra profesión, sobre todo cuando nos es completamente ajeno. El motivo es, sin discusión, **la ausencia de tests**.

En efecto, hasta hace unos años, la norma de la industria venía a ser, "desarrollar lo más rápido posible para salir a producción". Dentro de esa filosofía, la implementación de tests (unitarios, de integración, de sistema, etc) parecía entenderse como una pérdida de tiempo total. El concepto de [deuda técnica](http://es.wikipedia.org/wiki/Deuda_t%C3%A9cnica) era desconocido (o parecía serlo), y nadie pensaba en las consecuencias posteriores, tan sólo en el "sálvese quién pueda".

<!--break-->

No me voy a extender aquí en los peligros de semejante barbaridad. Los sistemas van creciendo sin control, el [Spaghetti code](http://stackoverflow.com/questions/2756527/what-is-spaghetti-code) hace acto de presencia, los diferentes módulos del sistema están cada vez más acoplados, y cualquier mínimo cambio tiene consecuencias desastrosas en partes del sistema que no podríamos ni imaginar.

Precisamente el no desarrollar tests unitarios es una de las principales causas de llegar a ese resultado. Los tests, si son realmente unitarios, nos ayudan a aislar dependencias entre clases/módulos, de forma que nos llevan a mejorar dramáticamente la calidad de nuestro diseño.

Uno de los grandes vicios de sistemas antiguos es el uso excesivo de métodos estáticos, así como de clases estáticas (clase con todos sus miembros estáticos). El problema es que un método estático no deja de ser un método global, y todos aprendimos que las variables globales son malas cuando aprendimos a programar, ¿verdad? :)

El mayor inconveniente de los métodos estáticos es lo enormemente que dificultan el desarrollo de tests unitarios de las clases que los utilizan, porque es bastante complicado crear [Mocks](http://java-white-box.blogspot.co.uk/2012/12/java-player-que-es-un-mock-para-que.html) de estos métodos. Existen librerías como [PowerMock](https://code.google.com/p/powermock/) para puentear esta restricción, pero a mi parecer ensucian bastante los tests, y además los hacen mucho más difíciles de depurar.

Si descartamos el uso de PowerMock, la opción es no crear un Mock de la clase estática, pero en tal caso ya no tendríamos tests unitarios, o intentar eliminar la naturaleza estática a los métodos/clases que nos están molestando.

Esta labor no es sencilla, cuando hay que abarcarla en un proyecto con miles de líneas de código, que además no contiene ningún test. Describiré a continuación una técnica paso a paso para conseguirlo de la forma más segura posible, técnica que aprendí en uno de los hangouts del [VirtualJUG](http://virtualjug.com/) impartido por [Sandro Mancuso](https://twitter.com/sandromancuso).

###El sistema de partida

Nuestro caso de uso es un sistema de registro y cobro de multas. El modelo de datos está simplificado al máximo, y contiene solo dos clases:

{% highlight java %}
public class Person {

    private int id;
    private String name;

    //...contructor, getters, setters
}

public class Fine {

    private int id;
    private Person person;
    private BigDecimal amount;
    private boolean paid;

    //...
}
{% endhighlight %}

El importe de la multa es de tipo BigDecimal, que es el tipo recomendado en Java para almacenar cantidades monetarias.

Las clase DAO de Person está simplificada como un contenedor de constantes:

{% highlight java %}
public class PersonDAO {
    public static final Person JOHN = new Person(1, "John");
    public static final Person RAUL = new Person(2, "Raul");
}
{% endhighlight %}

La clase DAO de Fine sería como sigue:

{% highlight java %}
public class FineDAO {

    private static List<Fine> fines = Lists.newArrayList();

    static {
        fines.add(new Fine(1, JOHN, valueOf(100), true));
        fines.add(new Fine(2, JOHN, valueOf(110), true));
        fines.add(new Fine(3, RAUL, valueOf(120), true));
        fines.add(new Fine(4, RAUL, valueOf(130), false));
        fines.add(new Fine(5, RAUL, valueOf(140), false));
    }


    public static List<Fine> getFines(Person person) {
        List<Fine> output = Lists.newArrayList();

        for (Fine fine : fines) {
            if (fine.getPerson().equals(person)) {
                output.add(fine);
            }
        }

        return output;
    }
}
{% endhighlight %}

Como se puede ver, almacena una lista de multas en memoria, lo que vendría a ser una representación in memory de una tabla Fine. Suficiente para nuestro ejemplo, aunque nada correcto, por supuesto. Solo contiene un método de acceso a datos, getFines, que recibe una instancia de Person y devuelve una lista de todas las multas registradas a nombre de esa persona.

Tenemos por último una clase FineService, con lógica asociada a la gestión de multas:

{% highlight java %}
public class FineService {

    public List<Fine> getUnpaidFines(Person person) {
        List<Fine> output = Lists.newArrayList();

        List<Fine> fines = FineDAO.getFines(person);

        for (Fine fine : fines) {
            if (!fine.isPaid()) {
                output.add(fine);
            }
        }

        return output;
    }
}
{% endhighlight %}

Solo contiene un método, `getUnpaidFines`, que devuelve una lista de las multas impagadas por la persona proporcionada en los parámetros.

Esto es todo. Un sistema bastante tonto, pero con una carencia muy importante (dejando aparte simplificaciones de las clases DAO para este post): la clase `FineDAO` ha sido creada como una clase estática, por lo que está fuertemente acoplada con la clase que la utiliza, `FineService`.

Imaginemos que el sistema es monstruoso y existen dependencias de la clase DAO por todas partes. ¿Cómo eliminar la naturaleza estática de forma segura? Veámoslo.

###El proceso, paso a paso

Lo primero que tenemos que hacer es crear tests para la clase `FineService`:

{% highlight java %}
public class FineServiceTest {

    private Person JOHN;
    private Person RAUL;

    private FineService fineService = new FineService();

    @Before
    public void setUp() throws Exception {
        JOHN = PersonDAO.JOHN;
        RAUL = PersonDAO.RAUL;
    }

    @Test
    public void testGetUnpaidFines() throws Exception {
        List<Fine> unpaidFines = fineService.getUnpaidFines(RAUL);
        Assertions.assertThat(unpaidFines).hasSize(2);
    }

    @Test
    public void testNoUnpaidFines() throws Exception {
        List<Fine> unpaidFines = fineService.getUnpaidFines(JOHN);
        Assertions.assertThat(unpaidFines).isEmpty();
    }
}
{% endhighlight %}

Varias puntualizaciones aquí:

* Teniendo en cuenta que la clase `PersonDAO` es una chapucilla creada para el ejemplo, en un sistema legacy real lo más seguro es que también fuera una clase estática, con sus métodos `getPersonByXXX`, que también habría que eliminar
* La clase service no tiene control de errores, etc, y los tests deberían ser más completos, comprobando no solo el número de multas devueltas. De nuevo, he optado por la simplicidad para centrarme en un aspecto muy determinado

####El problema: la clase DAO accede a la capa de persistencia

Efectivamente, ¿qué clase de test unitario accede a los datos reales? Como la clase `FineDAO` es estática, tenemos que pensar en alguna forma de "mockear" la llamada al método `getFines`...Para ello lo primero que vamos a hacer es aislar la invocación a `FineDAO.getFines` en un método propio dentro de `FineService`:

{% highlight java %}
public class FineService {

    public List<Fine> getUnpaidFines(Person person) {
        List<Fine> output = Lists.newArrayList();

        List<Fine> fines = getFines(person);

        for (Fine fine : fines) {
            if (!fine.isPaid()) {
                output.add(fine);
            }
        }

        return output;
    }

    protected List<Fine> getFines(Person person) {
        return FineDAO.getFines(person);
    }
}
{% endhighlight %}

¿Por qué protected? Porque la idea al hacer esto es crear una clase derivada que sobrescriba ese método:

{% highlight java %}
public class FineServiceTestable extends FineService {

    @Override
    protected List<Fine> getFines(Person person) {
        return super.getFines(person);
    }
}
{% endhighlight %}

Por supuesto, en el test ahora deberemos instanciar esta nueva clase:

{% highlight java %}
private FineService fineService = new FineServiceTestable();
{% endhighlight %}

Paro un momento para hacer hincapié en que, la idea de este proceso es implementarlo en incrementos pequeños, **ejecutando los tests a cada paso**. El uso de una herramienta como [infinitest](https://infinitest.github.io/) facilita bastante esta labor, porque lo hace automáticamente.

####Aislando dependencias en los tests

Ya estamos en posición de eliminar la dependencia de la clase estática en el test de `FineService`. Esto se hace modificando el método `getFines` de FineServiceTestable:

{% highlight java %}
public class FineServiceTestable extends FineService {

    private List<Fine> fines = Lists.newArrayList();

    public FineServiceTestable() {
        fines.add(new Fine(1, JOHN, valueOf(100), true));
        fines.add(new Fine(2, JOHN, valueOf(110), true));
        fines.add(new Fine(3, RAUL, valueOf(120), true));
        fines.add(new Fine(4, RAUL, valueOf(130), false));
        fines.add(new Fine(5, RAUL, valueOf(140), false));
    }

    @Override
    protected List<Fine> getFines(Person person) {
        List<Fine> output = Lists.newArrayList();

        for (Fine fine : fines) {
            if (fine.getPerson().equals(person)) {
                output.add(fine);
            }
        }

        return output;
    }
}
{% endhighlight %}

¡Maldita sea!, diréis. ¡Esta clase hace exactamente lo mismo que FineDAO! Bien, recordemos que FineDAO es un ejemplo muy simplificado, que en la realidad accedería a base de datos a traves de JDBC, MyBatis, Hibernate...En tal caso la clase `FineServiceTestable` sí pasaría a ser totalmente diferente. Además...nos hemos librado de la dependencia estática, que es lo que queríamos, ¿verdad?

####Refactorizando FineDAO

Vamos a eliminar paso a paso la naturaleza estática de FineDAO. Creemos en primer lugar los tests de la clase:

{% highlight java %}
public class FineDAOTest {

    private Person JOHN;
    private Person RAUL;

    @Before
    public void setUp() throws Exception {
        JOHN = PersonDAO.JOHN;
        RAUL = PersonDAO.RAUL;
    }

    @Test
    public void testGetFines() throws Exception {
        List<Fine> finesJohn = FineDAO.getFines(JOHN);
        Assertions.assertThat(finesJohn).hasSize(2);

        List<Fine> finesRaul = FineDAO.getFines(RAUL);
        Assertions.assertThat(finesRaul).hasSize(3);
    }
}
{% endhighlight %}

Añadamos ahora un método de instancia a la clase FineDAO que delega su comportamiento en el método estático:

{% highlight java %}
public class FineDAO {
 //...
 public List<Fine> getFinesByPerson(Person person) {
        return FineDAO.getFines(person);
    }
}
{% endhighlight %}

Y añadamos un test para este método:

{% highlight java %}
public class FineDAOTest {
    //...
    @Test
    public void testGetFinesByPerson() throws Exception {
        FineDAO fineDAO = new FineDAO();

        List<Fine> finesJohn = fineDAO.getFinesByPerson(JOHN);
        Assertions.assertThat(finesJohn).hasSize(2);

        List<Fine> finesRaul = fineDAO.getFinesByPerson(RAUL);
        Assertions.assertThat(finesRaul).hasSize(3);
    }
}
{% endhighlight %}

De nuevo, vamos paso a paso, con todos nuestros tests en verde, y sin romper otras partes del sistema.

Lo siguiente sería inyectar la dependencia `FineDAO` en `FineService`, y eliminar la dependencia estática:

{% highlight java %}
public class FineService {

    private FineDAO fineDAO;

    public List<Fine> getUnpaidFines(Person person) {
        List<Fine> output = Lists.newArrayList();

        List<Fine> fines = getFines(person);

        for (Fine fine : fines) {
            if (!fine.isPaid()) {
                output.add(fine);
            }
        }

        return output;
    }

    protected List<Fine> getFines(Person person) {
        return fineDAO.getFinesByPerson(person);
    }

    public void setFineDAO(FineDAO fineDAO) {
        this.fineDAO = fineDAO;
    }
}
{% endhighlight %}

La inyección de dependencias puede realizarse de múltiple formas (setters, constructores, anotaciones...). Intentando simplificar, en el ejemplo se hará mediente un setter.


####Añadiendo Mocks auténticos

Puesto que ya no tenemos dependencias estáticas en la clase `FineService`, es posible "mockear" sus dependencias de la forma tradicional, utilizando un framework como [Mockito](https://code.google.com/p/mockito/). Dejamos de utilizar, por tanto, la clase `FineServiceTestable`:

{% highlight java %}
public class FineServiceTest {
//...
    @Mock
    private FineDAO fineDAO;
    @InjectMocks
    private FineService fineService = new FineService();

    @Before
    public void setUp() throws Exception {
        MockitoAnnotations.initMocks(this);
//...
}
{% endhighlight %}

En este fragmento reflejo solo los cambios sobre la versión anterior. Hemos añadido Mockito al proyecto, inicializando los mocks en el método `setUp`, configurando la clase DAO para que se cree como mock, y la clase `FineService` para que se inyecten todas las dependencias.

En esta ocasión los tests no pasan, porque fineDAO es un mock al que aún no hemos asociado ningún comportamiento. Solventemos esto en cada uno de los tests:

{% highlight java %}
@Test
public void testGetUnpaidFines() throws Exception {
    when(fineDAO.getFinesByPerson(RAUL)).thenReturn(
            Lists.newArrayList(
                    new Fine(3, RAUL, valueOf(120), true),
                    new Fine(4, RAUL, valueOf(130), false),
                    new Fine(5, RAUL, valueOf(140), false)));

    List<Fine> unpaidFines = fineService.getUnpaidFines(RAUL);
    Assertions.assertThat(unpaidFines).hasSize(2);
}

@Test
public void testNoUnpaidFines() throws Exception {
    when(fineDAO.getFinesByPerson(JOHN)).thenReturn(
            Lists.newArrayList(
                    new Fine(1, RAUL, valueOf(100), true),
                    new Fine(2, RAUL, valueOf(110), true)));

    List<Fine> unpaidFines = fineService.getUnpaidFines(JOHN);
    Assertions.assertThat(unpaidFines).isEmpty();
}
{% endhighlight %}

Ya tenemos todos los tests en verde, y hemos conseguido lo que buscábamos, librarnos de la dependencia estática :D

####Repetir el proceso para dependencias pendientes

Si existen más dependencias en el sistema del método estático `FineDAO.getFines`, deberíamos repetir el proceso  para cada una de ellas. En caso contrario, habrá llegado el momento de eliminar definitivamente el método (moviendo su cuerpo dentro del método de instancia, eliminando de esta forma la delegación), y si queremos, renombrar el método instancia a getFines (el IDE se encargará de actualizar todas sus referencias sin problema). También tendremos que eliminar los tests del método estático, puesto que dejarán de compilar. La cosa, por tanto, queda así:

{% highlight java %}
public class FineDAO {

    private static List<Fine> fines = Lists.newArrayList();

    static {
        fines.add(new Fine(1, JOHN, valueOf(100), true));
        fines.add(new Fine(2, JOHN, valueOf(110), true));
        fines.add(new Fine(3, RAUL, valueOf(120), true));
        fines.add(new Fine(4, RAUL, valueOf(130), false));
        fines.add(new Fine(5, RAUL, valueOf(140), false));
    }

    public List<Fine> getFines(Person person) {
        List<Fine> output = Lists.newArrayList();

        for (Fine fine : fines) {
            if (fine.getPerson().equals(person)) {
                output.add(fine);
            }
        }

        return output;

    }
}
{% endhighlight %}


##Conclusiones

Eliminar esta dependencia estática no significa que ya esté todo el trabajo hecho. En el ejemplo utilizado en este post habría que seguir eliminando más dependencias estáticas (`PersonDAO`), y en un sistema legacy real seguramente estemos solo comenzando.

Sin embargo hemos conseguido varias mejoras, entre otras:

* Desacoplar dependencias
* Disminuir la rigidez del sistema
* Mejorar la cobertura de tests

(El código como siempre, en [GitHub](https://github.com/raulavila/refactoring-examples)).
