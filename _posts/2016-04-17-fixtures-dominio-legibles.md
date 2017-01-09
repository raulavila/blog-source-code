---
layout: post
title: Creando fixtures de dominio legibles
permalink: 2016/04/fixtures-dominio-legibles/
tags:
- desarrollo
- testing
comments: true
---

En muchas ocasiones nos habremos encontrado en nuestros proyectos con la necesidad de testear capas aisladas que hacen uso de objetos de dominio, objetos que en tests de integración / producción serán creados y poblados por un framework ORM o similar.

Dichos objetos deberán contener una serie de valores que condicionarán el resultado del proceso a ejecutar sobre ellos, por lo que en nuestros tests unitarios deberemos crear instancias adecuadas, que formarán parte de lo que se conoce como [test fixture](https://en.wikipedia.org/wiki/Test_fixture).

<!--break-->

Siempre que me he encontrado con este escenario he tenido que comerme la cabeza junto con mi equipo sobre cuál es la mejor forma de hacer legibles los contenidos de estas instancias de dominio. Como base para el resto del artículo vamos a definir una clase `Person` con esta estructrura:

{% highlight java %}
public class Person {

    private String name;
    private String phoneNumber;
    private Person nextOfKin;
    private List<Person> children;
    private Map<String, String> additionalDetails;

    //getters, setters, toString, etc
}  
{% endhighlight %}

Nada complicado, en general es normal encontrarse con estructuras de datos más complejas.

Digamos que tenemos que testear una clase `PersonProcessor`, que expone en su API el siguiente método:

{% highlight java %}
public class PersonProcessor {

    public String process(Person person) {
        //...
    }    
}    
{% endhighlight %}

El proceso ejecutado no viene al caso, solo importa saber que según los contenidos de la instancia `Person`, la salida será una u otra. Según he comentado anteriormente, nuestro código de producción nunca generará instancias de persona directamente, que serán pobladas automáticamente por el framework de acceso a datos que utilicemos. Pero en un test unitario somos nosotros quienes deberemos hacer esto, es decir, buscamos algo así:

{% highlight java %}
@Test
public void processRaul() throws Exception {
    Person raul = //create person...

    String processResult = personProcessor.process(raul);

    assertThat(processResult).isEqualTo("I've processed Raul");
}
{% endhighlight %}

Veamos cuál sería la solución más directa, que utiliaría únicamente los métodos expuestos por `Person`:

{% highlight java %}
Person raul = new Person();

raul.setName("Raul");
raul.setPhoneNumber("123455334");

Person nextOfKin = new Person();
nextOfKin.setName("Vanesa");
nextOfKin.setPhoneNumber("1232433");

raul.setNextOfKin(nextOfKin);

Person child1 = new Person();
child1.setName("Pixie");
child1.setPhoneNumber("1232421");

Person child2 = new Person();
child2.setName("Dixie");
child2.setPhoneNumber("1232444");

raul.setChildren(Arrays.asList(child1, child2));

Map<String, String> additionalDetails = new HashMap<>();
additionalDetails.put("Favourite song", "Back in Black");
{% endhighlight %}

No sé a vosotros, pero a mí personalmente este tipo de código siempre me ha parecido muy poco directo y nada claro de leer rápidamente para hacerse una idea de los contenidos del objeto de dominio instanciado.

## Alternativas a la instanciación directa

Existen varias alternativas para crear estos objetos de forma más clara, pero la mayoría de ellas requieren añadir librerías adicionales a nuestros tests (cosa que, por otro lado, tampoco es el fin del mundo). Echemos un vistazo rápido a estas soluciones.

### Spock

Mi alternativa favorita es utilizar el framework [Spock](http://spockframework.github.io/spock/docs/1.0/index.html), del que [ya hablamos en el blog](/2016/03/spock-vs-junit/). Gracias a esto, y a poder utilizar las facilidades del lenguaje Groovy, instanciar objetos complejos es muy sencillo utilizando el "map constructor" que
expone Groovy para las clases de forma automática:

{% highlight groovy %}
Person raul = new Person(
    name: "Raul",
    phoneNumber: "123455334",
    nextOfKin: new Person(name: "Vanesa",
                          phoneNumber: "1232433"),
    children: [
        new Person(name: "Pixie",
                   phoneNumber: "1232421"),
        new Person(name: "Pixie",
                   phoneNumber: "1232444")
    ],
    additionalDetails: ["Favourite song": "Back in Black"]
)
{% endhighlight %}

Personalmente encuentro esta solución muy limpia y adecuada, pero no todo el mundo es partidario de utilizar Spock y Groovy en sus tests

### Serializar en ficheros de texto

Podemos almacenar el contenido del objeto en un fichero en formato [JSON](https://es.wikipedia.org/wiki/JSON) o [YAML](https://es.wikipedia.org/wiki/YAML), y deserializar estos ficheros en nuestros tests utilizando librerías como [GSon](https://es.wikipedia.org/wiki/Gson) o [YAML Beans](http://yamlbeans.sourceforge.net/). Echad un vistazo a los links para comprobar como sería el proceso (bastante sencillo por otro lado). El punto negativo de esta solución es que nos obliga a navegar entre ficheros de texto y nuestro código de test para entender qué está ocurriendo realmente.


### Crear factorías de test

Por último, y si queremos evitar añadir dependencias o frameworks adicionales para una tarea que debería ser bastante sencilla, podemos crear factorías de test que nos asistan en la creación de nuestros objetos de dominio.

La primera alternativa sería encapsular el código visto en el primer ejemplo de todos, de esta forma:

{% highlight java %}
public class PersonTestFactory {

    public static Person createRaul() {
        Person raul = new Person();
        //...
    }
}
{% endhighlight %}

Por lo que en nuestros test llamaríamos al método factoría directamente. De esta forma seguiríamos teniendo que navegar entre ficheros, pero es más directo que abrir ficheros de texto, etc. El problema es que seguiríamos teniendo la dificultad añadida de asimilar de manera rápida los contenidos de la instancia.

Como alternativa, una solución que comenzamos a utilizar recientemente en un proyecto es implementar una suerte de "factory helpers" que permiten la instanciación de manera anidada de forma bastante clara. Antes de ver el código de dicha factoría creo que es mejor comprobar de qué forma sería utilizado, aquí tenemos un ejemplo:

{% highlight java %}
Person raul =
    person(
        name("Raul"),
        phoneNumber("123455334"),
        nextOfKin(
            name("Vanesa"),
            phoneNumber("1232433")
        ),
        children(
            child(
                name("Pixie"),
                phoneNumber("1232421")
            ),
            child(
                name("Dixie"),
                phoneNumber("1232444")
            )
        ),
        additionalDetails(
            detail(
                detailName("Favourite song"),
                detailValue("Back in Black")
            )
        )
    );
{% endhighlight %}

La estructura de `Person` queda así expuesta de forma muy clara en los tests. El código de la clase factory puede llegar a resultar un pelín enrevesado, pero viene a compensar por su facilidad de uso, veamos la clase completa ([también tenéis el código en GitHub](https://github.com/raulavila/blog-examples/tree/master/src/test/java/com/raulavila/fixtures)):

{% highlight java %}
public class PersonTestFactory {

    public static Person person(
            String name,
            String phoneNumber,
            Person nextOfKin,
            List<Person> children,
            Map<String, String> additionalDetails) {

        Person person = new Person();
        person.setName(name);
        person.setPhoneNumber(phoneNumber);
        person.setNextOfKin(nextOfKin);
        person.setChildren(children);
        person.setAdditionalDetails(additionalDetails);
        return person;
    }

    public static Person person(
            String name,
            String phoneNumber) {

        Person person = new Person();
        person.setName(name);
        person.setPhoneNumber(phoneNumber);
        return person;
    }

    public static String name(String name) {
        return name;
    }

    public static String phoneNumber(String phoneNumber) {
        return phoneNumber;
    }

    public static Person nextOfKin(String name, String phoneNumber) {
        return person(name, phoneNumber);
    }

    public static List<Person> children(Person... child) {
        return Arrays.asList(child);
    }

    public static Person child(String name, String phoneNumber) {
        return person(name, phoneNumber);
    }

    public static Map<String, String> additionalDetails(
            Map.Entry<String, String>...entries) {

        Map<String, String> map = new HashMap<>();

        for (Map.Entry<String, String> entry : entries) {
            map.put(entry.getKey(), entry.getValue());
        }

        return map;
    }

    public static Map.Entry<String, String> detail(String key,
                                                   String value) {
        return Maps.immutableEntry(key, value);
    }

    public static String detailName(String name) {
        return name;
    }

    public static String detailValue(String value) {
        return value;

    }
}
{% endhighlight %}

Tenemos un conjunto de métodos factory, todos ellos con el nombre del campo que están creando (sin verbos tipo `create`, `build`, etc), y que utilizados de forma anidada nos llevan a crear instancias de manera muy legible. Quizás resulta algo chocante la existencia de métodos como `name` que no son más que funciones identidad, pero repito, la idea de esta clase es facilitar le legibilidad de los tests, y creo que se consigue de sobra.

El mayor problema con esta solución lo tendríamos en estructuras muy complejas, con varios niveles de anidamiento, etc. En ese caso, seguramente sean mejores alguna de las otras soluciones, pero en general, creo que conviene conocer todas las opciones a nuestro alcance antes de decidir la que mejor se ajuste a nuestras necesidades.

¡Si conocéis alguna otra alternativa no dudéis en dejar un comentario!
