---
layout: post
title: Uso avanzado de enumerados
permalink: 2015/02/enums-java/
tags:
- Java
- enums
- desarrollo
comments: true
---

[Los tipos enumerados de Java](http://docs.oracle.com/javase/7/docs/api/java/lang/Enum.html) (`enum`) fueron añadidos en la versión 5 del lenguaje. Su uso más inmediato fue el de "contenedores de constantes", y me temo que gran parte de los desarrolladores desconocen la flexibilidad que permiten más allá de este típico uso:

 {% highlight java %}
public enum EmployeeType {
    ENGINEER,
    MANAGER,
    SALESMAN;
}
 {% endhighlight %}

<!--break-->

También es bastante normal que se utilicen de una forma algo más avanzada, asignando parámetros a cada valor enumerado:

{% highlight java %}
public enum EmployeeType {
    ENGINEER("Engineer", 30000),
    MANAGER("Manager", 40000),
    SALESMAN("Salesman", 50000);

    private final String value;
    private final int salary;

    EmployeeType(String value, int salary) {
        this.value = value;
        this.salary = salary;
    }

    public String getValue() {
        return value;
    }

    public int getSalary() {
        return salary;
    }
}
{% endhighlight %}

En el ejemplo se asigna un nombre "para mostrar" (`toString()` por defecto sacaría el valor del enumerado tal cual, en letras mayúsculas) y un salario a cada tipo de empleado.

Mi intención con este post es dar a conocer usos más avanzados de los enumerados, que yo personalmente utilizo bastante a menudo y de formas quizás menos ortodoxas.

([Como siempre el código está en GitHub](https://github.com/raulavila/enums-examples))

### Enums como estados con comportamiento asociado

Los enumerados me parecen una de las formas más claras y elegantes de implementar [el patrón State](http://en.wikipedia.org/wiki/State_pattern). En el siguiente ejemplo creamos una clase *Calculator* de una sola operación, y que expone dos métodos, `setOperation`, para cambiar la operación (el estado) de la calculadora, y `performOperation`, que recibe dos números y devuelve el resultado de aplicar la operación:

  {% highlight java %}
public class Calculator {

    private Operation operation;

    public Calculator(Operation operation) {
        this.operation = operation;
    }

    public int performOperation(int operand1,
                                int operand2) {

        return operation.perform(operand1, operand2);
    }

    public void setOperation(Operation operation) {
        this.operation = operation;
    }
}
  {% endhighlight %}

  Como vemos, la clase no tiene ni idea de como se implementan las operaciones, delegando su ejecución a la instancia de Operation:

   {% highlight java %}
public enum Operation {
    ADD("+") {
        @Override
        public int perform(int operand1, int operand2) {
            return operand1 + operand2;
        }
    },
    SUBSTRACT("-"){
        @Override
        public int perform(int operand1, int operand2) {
            return operand1 - operand2;
        }
    },
    MULTIPLY("*"){
        @Override
        public int perform(int operand1, int operand2) {
            return operand1 * operand2;
        }
    },
    DIVIDE("/"){
        @Override
        public int perform(int operand1, int operand2) {
            return operand1 / operand2;
        }
    };

    private final String operator;

    Operation(String operator) {
        this.operator = operator;
    }

    public String getOperator() {
        return operator;
    }

    public abstract int perform(int operand1, int operand2);
}
   {% endhighlight %}

Aquí es donde se ve de forma clara cómo encapsulamos cada operación en las distintas instancias de Operation dentro del método `perform`, que a nivel de clase está declarado como método abstracto (ya que se implementa en cada uno de los enumerados). También exponemos un atributo, `operator`, que no es más que el símbolo que identifica a cada operación, aunque en el ejemplo no se utiliza.

El método `perform` podría haber contenido una implementación por defecto, de forma que si luego no la sobrescribimos en las instancias sería la ejecutada por el compilador:

 {% highlight java %}
public enum Operation {
   //....
   VERY_COMPLEX_OPERATION;
   //....
   public int perform(int operand1, int operand2){
      throw new UnsupportedOperationException();
   }
}
 {% endhighlight %}

 En este ejemplo podemos declarar una operación y no implementarla de momento, mientras que con la primera versión no sería posible.

Como es de esperar la calculadora funciona según lo esperado:

  {% highlight java %}
Calculator calculator = new Calculator(Operation.ADD);

assertThat(calculator.performOperation(8, 2)).isEqualTo(10);

calculator.setOperation(Operation.SUBSTRACT);

assertThat(calculator.performOperation(8, 2)).isEqualTo(6);
  {% endhighlight %}


###Estados asociados

Siguiendo con el patrón State, es posible asociar estados de la misma enumeración entre sí, con la única condición de que no es posible referenciar un valor del enumerado que aún no ha sido declarado:

 {% highlight java %}
enum OvenState {

    END("Oven finished cooking", null),
    COOLING("Oven is cooling", END),
    COOKING("Oven is cooking", COOLING),
    HEATING("Oven is heating", COOKING),
    BEGIN("Oven is starting", HEATING);

    private final String value;
    private final OvenState next;

    OvenState(String value, OvenState next) {
        this.value = value;
        this.next = next;
    }

    public String getValue() {
        return value;
    }

    public OvenState getNext() {
        return next;
    }

    public boolean hasNext() {
        return next != null;
    }
}
 {% endhighlight %}

 En el ejemplo defino estados de un horno imaginario, de forma que cada estado contiene una referencia de su estado siguiente. Debido a la restricción comentada más arriba, no es posible declarar los estados como sería más natural, de BEGIN a END, porque en tal caso el compilador se quejaría.

La implementación del horno con su estado quedaría:

{% highlight java %}
public class Oven {

    private OvenState state;

    public Oven() {
        reset();
    }

    public void reset() {
        state = OvenState.BEGIN;
    }

    public void click() {
        if(hasFinished())
            throw new IllegalStateException(
                    "Oven has finished cooking");

        state = state.getNext();
    }

    public String getState() {
        return state.getValue();
    }

    public boolean hasFinished() {
        return !state.hasNext();
    }
}
{% endhighlight %}

De nuevo el horno no tiene ni idea de todos los estados por los que pasa, solo sabe que hay un estado inicial, y que existe un estado final según el valor devuelto por `hasFinished`.

{% highlight java %}
Oven oven = new Oven();
assertThat(oven.getState()).isEqualTo("Oven is starting");

oven.click();
assertThat(oven.getState()).isEqualTo("Oven is heating");

oven.click();
assertThat(oven.getState()).isEqualTo("Oven is cooking");

oven.click();
assertThat(oven.getState()).isEqualTo("Oven is cooling");

oven.click();
assertThat(oven.getState()).isEqualTo("Oven finished cooking");

assertThat(oven.hasFinished()).isTrue();

try {
    oven.click();
} catch (IllegalStateException e) {
    assertThat(e.getMessage()).isEqualTo(
            "Oven has finished cooking");
}

oven.reset();
assertThat(oven.getState()).isEqualTo("Oven is starting");
{% endhighlight %}

Siguiendo con mi manía de desacoplar al máximo, ¡el horno ni siquiera tendría por qué saber el valor del estado inicial! Para eso habría que declarar un método estátido en OvenState:

{% highlight java %}
public static OvenState getInitialState() {
        return BEGIN;
}
{% endhighlight %}

Y en Oven:

{% highlight java %}
public void reset() {
        state = OvenState.getInitialState();
}
{% endhighlight %}

Esto ya queda al gusto de cada uno, pero yo prefiero esta última versión.


###Enums como parametrizaciones

Imaginemos un sistema que contiene una serie de parámetros de configuración a persistir en una base de datos o similar. Dichos parámetros pueden contener diversos tipos de datos, pero en la tabla el tipo del valor será un String para dar flexibilidad, es decir, tendremos algo como:

<table>
    <thead>
        <tr>
            <th>
               PARAM_NAME : STRING
            </th>
            <th>
               PARAM_VALUE : STRING
            </th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
               systemName
            </td>
            <td>
               Inventory
            </td>
        </tr>
        <tr>
            <td>
              concurrentUsers
            </td>
            <td>
               100
            </td>
        </tr>
    </tbody>
</table>

Aunque el tipo de PARAM_VALUE sea un String, es evidente que cada parámetro tiene una naturaleza muy concreta (podemos tener fechas, enteros, cadenas, etc). Nos gustaría que nuestra clase DAO que persiste esta información chequeara el tipo de los datos antes de almacenarlos. Además queremos que los nombres de los parámetros estuvieran acotados a una serie muy concreta.

Diría que no hay mejor forma de hacer esto que utilizando un enumerado:

{% highlight java %}
public enum ConfigParam {

    SYSTEM_NAME("systemName", String.class),
    CONCURRENT_USERS("concurrentUsers", Integer.class);

    private final String paramName;
    private final Class<?> type;

    ConfigParam(String paramName, Class<?> type) {
        this.paramName = paramName;
        this.type = type;
    }

    public boolean hasValidTypeFor(Object value) {
        if (value == null) {
            return false;
        }

        return value.getClass().isAssignableFrom(type);
    }

    public String getParamName() {
        return paramName;
    }

    public String getTypeName() {
        return type.getSimpleName();
    }
}
{% endhighlight %}

Como vemos, cada instancia contiene el nombre del parámetro a persistir, y la clase del valor. El propio enumerado expone un método que chequea si un objeto en concreto contiene un valor válido para el parámetro.

La clase DAO, por tanto, delegaría toda la lógica al valor del enumerado que quiera almacenar el cliente:

{% highlight java %}
public interface ConfigDAO {
    void save(ConfigParam param, Object value);
}

public abstract class AbstractConfigDAO implements ConfigDAO {

    @Override
    public void save(ConfigParam param, Object value) {
        validate(param, value);

        doSave(param.getParamName(), value);
    }

    private void validate(ConfigParam param, Object value) {
        Validate.notNull(param, "Param can't be null");
        Validate.notNull(value, "Value can't be null");

        if (!param.hasValidTypeFor(value)) {
            throw new IllegalArgumentException(
                    param.getParamName()+
                            " value must be an instance of " +
                            param.getTypeName());
        }

    }

    protected abstract void doSave(String paramName, Object value);
}
{% endhighlight %}

La clase `AbstractConfigDAO` es una implementación del [Template method pattern](http://en.wikipedia.org/wiki/Template_method_pattern), y sería necesario completar el "algoritmo" con una implementación concreta del método `doSave`. En el código para este ejemplo me olvido por completo de almacenar nada (el cometido de este post no es hablar de persistencia de datos en Java :) ),por lo que he creado una implementación Dummy:

 {% highlight java %}
public class DummyConfigDAO extends AbstractConfigDAO {

    @Override
    protected void doSave(String paramName, Object value) {
        System.out.println("Saving param " + paramName +
                                " with value " + value.toString());
    }
}
 {% endhighlight %}

 En los tests se ve muy bien lo claro que quedan los clientes utilizando esta aproximación:

 {% highlight java %}
@Test
public void testCorrectSystemName() throws Exception {
    configDAO.save(ConfigParam.SYSTEM_NAME, "mySystem");
}

@Test
public void testIncorrectSystemName() throws Exception {
    expectedException.expect(IllegalArgumentException.class);
    expectedException.expectMessage(
            "systemName value must be an instance of String");

    configDAO.save(ConfigParam.SYSTEM_NAME, Integer.valueOf(1));
}
 {% endhighlight %}


###Conclusión

 Espero que con este post haya quedado demostradada la flexibilidad de los enumerados en Java, y la claridad que dan al código si se usan convenientemente. Me he dejado algún uso en el tintero, como puede ser la implementación del patrón Singleton, [pero este tema ya está cubierto de sobra.](http://javarevisited.blogspot.co.uk/2012/07/why-enum-singleton-are-better-in-java.html)
