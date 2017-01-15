---
layout: post
title: Revisa tu código autogenerado
description: Cosas a tener en cuenta cuando utilizamos herramienas de generación de código en IDEs
permalink: 2016/02/codigo-autogenerado/
tags:
- desarrollo
- Java
- IDE
comments: true
---

En este post pretendo revisar una serie de detalles que, basándome en mi experiencia, es muy normal que pasemos por alto a la hora de desarrollar código en Java, y nos lleve a dar por buenas decisiones que pueden no serlo tanto. Todas ellas tienen su causa en la asistencia para generar código que nos ofrecen los IDEs, y que nos lleva a no revisar (o no reflexionar debidamente) sobre lo que acaba de ocurrir cuando seleccionamos esa "mágica" opción de nuestro menú.

No me malinterpretéis, estas herramientas son realmente magníficas, pero creo que es importante no tomar lo que hacen como dogma de fe.

<!--break-->

### Clases públicas

Para los IDEs por defecto todas las clases son públicas. De esta forma, si le decimos a IntelliJ que nos cree una clase, ocurrirá esto:

{% highlight java %}
public class MyClass {
}
{% endhighlight %}

Las clases por defecto deberían tener el scope package, es decir:

{% highlight java %}
class MyClass {
}
{% endhighlight %}

De forma que no serían visibles fuera del paquete a que pertenecen. Si en algún momento necesitamos ampliar su ámbito tendremos que pensar por qué lo necesitamos, y tomar las decisiones de diseño que sean convenientes. Si la clase es pública normalmente la utilizaremos y punto, haciendo que nuestro código tenga un acoplamiento más alto.

La mejor forma de solucionar esto es editando la plantilla por defecto para una nueva clase. En IntelliJ, por ejemplo, esta opción la podemos encontrar en Preferences > Editor > File and Code templates > Templates > Class.

Una de las mejores prácticas al utilizar el ámbito package por defecto, es crear una interfaz como pública, pero no exponer sus implementaciones. De esta forma los clientes de la interfaz no tendrán ni idea de la implementación que están utilizando porque ni siquiera tienen acceso a ella, favoreciendo de esta forma prácticas como la [inyección de dependencias](/2015/03/principios-dependencias/).

### Clases inmutables

Este es un detalle que no tiene una asociación directa con el código autogenerado, pero que en el 99% de ocasiones se pasa por alto. En general, cuando queremos diseñar una clase como inmutable el resultado se asemejará bastante a este:

{% highlight java %}
public class MyImmutableClass {

    private final int id;
    private final String name;

    public MyImmutableClass(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }
}

{% endhighlight %}

Campos privados y final, nada de setters e inicialización por constructor. Ok, está bastante bien, pero se nos ha escapado algo. Veamos que ocurre si hacemos esto:

{% highlight java %}
public class MyImmutableDerivedClass extends MyImmutableClass{

    public String address;

    public MyImmutableDerivedClass(int id, String name, String address) {
        super(id, name);
        this.address = address;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
{% endhighlight %}

Una instancia de esta segunda clase, que hereda de `MyImmutableClass` podría hacerse pasar perfectamente por una instancia de esta, es decir:

{% highlight java %}
MyImmutableClass myImmutableClass
                = new MyImmutableDerivedClass(1, "name", "address");
{% endhighlight %}

Lo cual no es ideal, porque esta clase derivada, ¡no es inmutable!

La solución es bien sencilla, hacer nuestra clase inmutable `final`:

{% highlight java %}
public final class MyImmutableClass {
    //...
}
{% endhighlight %}

Y ya no será posible crear instancias derivadas.

### Getters y Setters

Es bastante habitual crear una clase con sus campos, y añadir, en un frenesí de código autogenerado, un constructor, y getters y setters para todos los campos.

¿Realmente necesitamos hacer esto?

{% highlight java %}
public class MyClass {

    private Collaborator1 collaborator1;
    private Collaborator2 collaborator2;

    public MyClass(Collaborator1 collaborator1,
                   Collaborator2 collaborator2) {
        this.collaborator1 = collaborator1;
        this.collaborator2 = collaborator2;
    }

    public Collaborator1 getCollaborator1() {
        return collaborator1;
    }

    public void setCollaborator1(Collaborator1 collaborator1) {
        this.collaborator1 = collaborator1;
    }

    public Collaborator2 getCollaborator2() {
        return collaborator2;
    }

    public void setCollaborator2(Collaborator2 collaborator2) {
        this.collaborator2 = collaborator2;
    }
}
{% endhighlight %}

Una clase debería exponer el mínimo posible de detalles internos. Así que, de la misma forma que en la primera sección mencionaba que una clase debería tener scope package hasta que requiera ser pública, ahora digo que una clase no debería tener ni getters ni setters hasta que sea necesario. Y cuando lo sea, pensad si realmente es la mejor decisión o hay algo en vuestro diseño que no es adecuado.

### Método equals

En general, los IDEs exponen una opción para generar los métodos `equals` y `hashCode` conjutamente. Esto tiene sentido, ya que no tendría mucho sentido definir un método sin el otro (no entraré en detalles aquí, pero creo que todos entendemos el [contrato equals y hashCode](http://www.programcreek.com/2011/07/java-equals-and-hashcode-contract/)).

No tengo mucho que objetar a las implementaciones que suelen generarse para el método `hashCode`, suelen ser bastante óptimas, aunque seguramente se puedan optimizar algo más. El problema surge con la implementación de equals:

{% highlight java %}
public class MyClass {

    private final int id;
    private final String name;

    //Constructor, equals, etc

    @Override
    public boolean equals(Object o) {
        if (this == o)
            return true;

        if (o == null ||
            getClass() != o.getClass())
                return false;

        MyClass myClass = (MyClass) o;

        if (id != myClass.id)
            return false;

        if (name != null ?
            !name.equals(myClass.name) :
            myClass.name != null)
                return false;

        return true;
    }
}    
{% endhighlight %}

Parece todo correcto, pero aquí está ocurriendo algo muy grave, y es que esta clase rompe el [Liskov Susbstitution Principle](https://en.wikipedia.org/wiki/Liskov_substitution_principle). Este principio forma parte de los [principios SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)), y en pocas palabras viene a decir que "una clase derivada puede hacerse pasar por su clase base sin que los clientes lo noten". Se trata de una definición bastante burda, y es este un principio que requeriría de un artículo entero, pero creo que añadiendo un ejemplo que nos haga ver por qué esta implementación de `equals` lo rompe quedará bien claro.

Creemos una clase derivdada:

{% highlight java %}
public class MyDerivedClass extends MyClass {
    private final String address;

    public MyDerivedClass(int id, String name, String address) {
        super(id, name);
        this.address = address;
    }

    public String getAddress() {
        return address;
    }

    // equals and hashCode
}
{% endhighlight %}

Me ahorro la implementación de `equals` y `hashCode` en esta segunda clase, porque no lo necesitamos a efectos del ejemplo.

{% highlight java %}
public static void main(String[] args) {
    MyClass object1 = new MyClass(1, "name");
    MyDerivedClass object2 = new MyDerivedClass(1, "name", "address");

    System.out.println(areEquals(object1, object2));
}

public static boolean areEquals(MyClass object1, MyClass object2) {
    return object1.equals(object2);
}
{% endhighlight %}

El método `areEquals` recibe dos instancias de `MyClass`. Ambas instancias son idénticas en cuanto la información definida en dicha clase, por lo tanto sería de esperar que `areEquals` devuelva `true`. Pero esto no es así, porque el método `equals` autogenerado hace lo siguiente:

{% highlight java %}
if (o == null ||
    getClass() != o.getClass())
        return false;
{% endhighlight %}

`getClass` devuelve `MyClass` para el primer objeto y `MyDerivedClass` para el segundo, por lo que nuestro método `equals` devuelve `false`.

Se podría discutir que en realidad es adecuado que ocurra esto, ya que se trata de instancias de clases diferentes. ¿Pero tiene realmente esto sentido cuando si interrogamos cada uno de los campos de ambas clases los valores son idénticos? De hecho, una implementación alternative de `areEquals` podría ser:

{% highlight java %}
public static boolean areEquals(MyClass object1, MyClass object2) {
    return object1.getId() == object2.getId() &&
            object1.getName().equals(object2.getName());
}
{% endhighlight %}

Y en este caso la salida es `true`.

De esto trata el Liskov Substitution Principle. Otra de las implicaciones de este principio es que si sobrescribimos un método será para añadir comportamiento a lo que ya hace la clase base, no para eliminar nada. De hecho, una práctica que rompe el principio de forma clamorosa es sobrescribir un método para dejarlo vacío (que alguien tire la primera piedra aquí).

En resumidas cuentas, necesitamos buscar una mejor implementación para nuestro método `equals`. Y sería esta:

{% highlight java %}
@Override
public boolean equals(Object o) {
    if (this == o)
        return true;

    if (!(o instanceof MyClass))
        return false;

    MyClass myClass = (MyClass) o;

    if (id != myClass.id)
        return false;

    if (name != null ?
            !name.equals(myClass.name) :
            myClass.name != null)
                return false;

    return true;
}
{% endhighlight %}

En lugar de interrogar a los objetos por su clase utilizando `getClass` lo que hacemos es preguntar al objeto pasado por parámetro a `equals` si es una instancia de la clase donde vive nuestro `equals` mediante el operador `instanceof`. Este operador devuelve `true` tanto para instancias de la clase con la que estamos interrogando como con sus clases derivadas. Y ahora sí, conseguimos que nuestra primera versión de `areEquals` funcione correctamente. Todo esto (y mucho más) viene perfectamente explicado en el seminal libro [Effective Java](http://www.amazon.es/Effective-Java-2nd-Programming-Language/dp/0321356683/ref=sr_1_1?ie=UTF8&qid=1455984908&sr=8-1&keywords=Effective+Java), libro que todo desarrollador Java debería tener en su estantería.

En realidad IntelliJ permite autogenerar esta versión de `equals`, pero debemos ser cuidadosos en una de las opciones del asistente, que por defecto viene desmarcada, y debemos marcar nosotros. Se trata de "Accept subclasses as parameter to equals() method":

![IntelliJ equals message](/public/pictures/IntelliJ-message-equals.jpg)

El propio cuadro de diálogo nos menciona que la implementación por defecto rompe el contrato de `equals`. También parece que la implementación con `instanceof` puede no ser muy compatible con algunos frameworks. En tal caso deberíamos cambiar a la primera versión, pero siempre siendo conscientes de las consecuencias. Es decir, utilicemos en primera instancia la versión correcta, y solo pasemos al plan B si no queda más remedio :)
