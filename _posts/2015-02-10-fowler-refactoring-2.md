---
layout: post
title: Desgranando Fowler's Refactoring (parte 2)
description: Una visión crítica del libro Refactorin de Martin Fowler, segunda parte
permalink: 2015/02/fowler-refactoring-2/
tags:
- refactoring
- desarrollo
- libros
comments: true
---

Continúo la serie de artículos sobre "Refactoring". En esta ocasión la víctima de mi bisturí será lo que, a mi parecer, es un claro error de diseño cometido en la página 225, cuando se desgrana el refactoring "Replace type code with subclasses". El resultado final de este refactoring está representado en el siguiente diagrama UML:

![Jerarquía Employy](/public/pictures/employee_uml.png)

<!--break-->

La versión final de la clase Employee queda, más o menos, así:

{% highlight java %}

public abstract class Employee {

    protected Employee() {}

    public static Employee create(EmployeeType type) {
        switch (type) {
            case ENGINEER:
                return new Engineer();
            case SALESMAN:
                return new Salesman();
            case MANAGER:
                return new Manager();
            default:
                throw new AssertionError();
        }
    }

    public abstract EmployeeType getType();
}

{% endhighlight %}

Y digo más o menos porque en el libro, en lugar de utilizar una enumeración para las diferentes categorías de empleados se utilizan constantes de tipo int. En la época en que fue escrito el libro no existían las enumeraciones en Java, por lo que me he permitido actualizar esa propuesta final de la forma en que considero habría quedado con enumeraciones.

Veamos como quedaría una de las clases derivadas de Employee:

{% highlight java %}

class Manager extends Employee {
    @Override
    public EmployeeType getType() {
        return EmployeeType.MANAGER;
    }
}

{% endhighlight %}

Estas clases tienen un ámbito package (por defecto). Esto significa que los clientes de Employee no tienen acceso a las clases derivadas, y por tanto deben crear los diferentes tipos de empleado utilizando el factory method estático que expone la clase Employee:

{% highlight java %}

Employee manager = Employee.create(EmployeeType.MANAGER);

{% endhighlight %}

El problema que veo a esta aproximación es que la clase Employee no debería tener conocimiento alguno sobre sus clases derivadas. Estamos creando un acoplamiento innecesario que viola tanto el [Single Responsibility Principle (SRP)](http://en.wikipedia.org/wiki/Single_responsibility_principle) como el [Open Closed Principle (OCP)](http://en.wikipedia.org/wiki/Open/closed_principle). La clase Employee solo debería contener la lógica común de un empleado, y no procedimientos de creación de clases que la implementen.

## Mi propuesta

La creación de instancias debería ser claramente responsabilidad de una clase factoría, que contendría el método ubicado hasta ahora en Employee.

{% highlight java %}

public enum EmployeeFactory {
    INSTANCE;

    public Employee create(EmployeeType type) {
        switch (type) {
            case ENGINEER:
                return new Engineer();
            case SALESMAN:
                return new Salesman();
            case MANAGER:
                return new Manager();
            default:
                throw new AssertionError();
        }
    }
}

{% endhighlight %}

Nótese que he decidido crear la factoría como singleton (utilizando la [forma definitiva de implementar el patrón singleton en Java](http://stackoverflow.com/questions/70689/what-is-an-efficient-way-to-implement-a-singleton-pattern-in-java)), y no como clase con métodos estáticos. En general, considero que los métodos estáticos son algo a evitar, y será un tema que discutiré con detenimiento en el futuro. De momento quedémonos con que, gracias al uso de la factoría como singleton en lugar de clase estática será posible rizar el rizo de la forma en que lo haré al final del artículo.

De esta forma la clase Employee se ha quedado sin métodos implementados, y con un método abstracto, getType. Esto suena sospechosamente a interfaz:

{% highlight java %}

public interface Employee {
    EmployeeType getType();
}

{% endhighlight %}

Por lo que las diferentes clases de empleado pasarán a ser implementaciones de la interfaz:

{% highlight java %}

class Manager implements Employee {
    @Override
    public EmployeeType getType() {
        return EmployeeType.MANAGER;
    }
}

{% endhighlight %}


### Mejorando la propuesta. Fuera switch.

En general, las sentencias switch (o en su defecto, una serie de ifs encadenados), suelen ser un ["code smell"](http://en.wikipedia.org/wiki/Code_smell), ya que entre otras cosas, aumentan la [complejidad ciclomática](http://es.wikipedia.org/wiki/Complejidad_ciclom%C3%A1tica). Me atrevería a decir que en un 95% de los casos es posible encontrar mejores soluciones que el uso de una sentencia switch.

En este caso, tenemos que el método create genera instancias de clases derivadas de Employee, basándose en el valor de una enumeración...diría que esto puede representarse en un Map EmployeeType => Class, y generando las instancias mediante el método newInstance de la clase Class. Veamos el resultado:

{% highlight java %}

public enum EmployeeFactory {
    INSTANCE;

    private Map<EmployeeType, Class<? extends Employee>> typeClassMap =
                new ImmutableMap.Builder<EmployeeType, Class<? extends Employee>>()
                    .put(EmployeeType.ENGINEER, Engineer.class)
                    .put(EmployeeType.SALESMAN, Salesman.class)
                    .put(EmployeeType.MANAGER, Manager.class)
                    .build();

    public Employee create(EmployeeType type) {
        validateType(type);
        try {
            Class<? extends Employee> clazz = typeClassMap.get(type);
            return clazz.newInstance();
        } catch (InstantiationException | IllegalAccessException e) {
            throw new RuntimeException("Unexpected error instantiating employee!", e);
        }
    }

    private void validateType(EmployeeType type) {
        Validate.notNull(type, "Parameter type can't be null");

        if (!typeClassMap.containsKey(type)) {
            throw new RuntimeException("Type is not associated to a concrete class");
        }
    }

}

{% endhighlight %}

Varias cosas a puntualizar:

1. El Map está creado usando [guava](https://code.google.com/p/guava-libraries/), librería desarrollada por Google, y que añade API's muy útiles a la nativa de Java para trabajar con Collections o predicados, entre otras.
2. He añadido un método de validación del parámetro de entrada, como red de seguridad. Siempre es mejor lanzar errores concretos que esperar a un NullPointer en la llamada a newInstance, por ejemplo. La clase Validate es parte de la librería [Apache Commons Lang](http://commons.apache.org/proper/commons-lang/).
3. El método newInstance lanza varias Checked Exceptions... Para evitar propagar estas excepciones me limito a envolverlas en una RuntimeException (lo 100% correcto realmente sería crear nuestra propia Unchecked Exception).

Diría que el diseño de este método es mucho más robusto que el anterior. Ahora sí cumple sobradamente con los SRP y OCP, la única responsabilidad de esta clase es la creación de instancias de Employee, y puede extenderse con nuevos tipos de empleado, pero no habría que modificar el método create para nada...mmmm, ¿podemos hacerlo mejor?


### Rizando el rizo

Realmente la clase no cumple al 100% con el OCP, porque sí que habría que modificar la clase si se añaden al sistema nuevos tipos de empleado (añadiendo la nueva entrada en el Map). Pero, ¿es posible no tener que modificar esta clase si se da el caso? Bueno, realmente ¡sí que es posible!:

{% highlight java %}

public class EmployeeFactoryConfigurable {

    private final Map<EmployeeType, Class<? extends Employee>> typeClassMap;

    public EmployeeFactoryConfigurable(
        Map<EmployeeType, Class<? extends Employee>> typeClassMap) {

        this.typeClassMap = typeClassMap;
    }

    public Employee create(EmployeeType type) {
        validateType(type);
        try {
            Class<? extends Employee> clazz = typeClassMap.get(type);
            return clazz.newInstance();
        } catch (InstantiationException | IllegalAccessException e) {
            throw new RuntimeException("Unexpected error instantiating employee!", e);
        }
    }

    private void validateType(EmployeeType type) {
        Validate.notNull(type, "Parameter type can't be null");

        if (!typeClassMap.containsKey(type)) {
            throw new RuntimeException("Type is not associated to a concrete class");
        }
    }

}

{% endhighlight %}

Esta clase recibe en el constructor la configuración en forma de Map con la estrategia que regirá la creación de instancias. Aunque hemos perdido la naturaleza Singleton de la versión anterior, lo más seguro es que esta factoría será utilizada en el ámbito de un Framework de inyección de dependencias (como puedan ser [Spring](http://projects.spring.io/spring-framework/), [Google Guice](https://code.google.com/p/google-guice/) o [Apache Aries Blueprint](http://aries.apache.org/modules/blueprint.html)). En estos frameworks, el scope por defecto de las instancias generadas es singleton, por lo que no debería suponer un gran problema. Incluso tenemos la flexibilidad añadida de poder crear diferentes versiones de la factoría con diferentes estrategias (cosa altamente improbable en este ejemplo, la verdad).

Veamos como quedarían los tests:

{% highlight java %}

    private EmployeeFactoryConfigurable employeeFactory;

    private EmployeeFactoryConfigurable employeeFactory;

    @Before
    public void setUp() throws Exception {
        Map<EmployeeType, Class<? extends Employee>> typeClassMap =
                new ImmutableMap.Builder<EmployeeType, Class<? extends Employee>>()
                        .put(EmployeeType.ENGINEER, Engineer.class)
                        .put(EmployeeType.SALESMAN, Salesman.class)
                        .put(EmployeeType.MANAGER, Manager.class)
                        .build();

        employeeFactory = new EmployeeFactoryConfigurable(typeClassMap);
    }

    @Test
    public void testCreateEmployees() throws Exception {

        Employee manager = employeeFactory.create(EmployeeType.MANAGER);
        assertThat(manager.getType()).isEqualTo(EmployeeType.MANAGER);

        Employee engineer = employeeFactory.create(EmployeeType.ENGINEER);
        assertThat(engineer.getType()).isEqualTo(EmployeeType.ENGINEER);

        Employee salesman = employeeFactory.create(EmployeeType.SALESMAN);
        assertThat(salesman.getType()).isEqualTo(EmployeeType.SALESMAN);
    }


{% endhighlight %}

Como siempre, los ejemplos los dejo subidos [en mi repositorio de GitHub](https://github.com/raulavila/fowlers-refactoring-errors). Seguiré desgranando este libro [en mi siguiente post](/2015/02/fowler-refactoring-3).
