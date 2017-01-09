---
layout: post
title: Dependency Inversion vs Dependency Injection
permalink: 2015/03/principios-dependencias/
tags:
- desarrollo
- principios
comments: true
---

Mi intención con este post es dar un poco de luz a toda la confusión que parece que existe en torno a dos conceptos/principios:

* Principio de Inversión de Dependencias (Dependency Inversion principle - DIP)
* Inyección de Dependencias (Dependency Injection - DI)

Como bonus comentaré además el Principio de Inversión de Control (Inversion of Control - IoC), que muchas veces se confunde con la inyección de dependencias.

<!--break-->

El motivo de tanta confusión es que estos conceptos están fuertemente relacionados, de hecho la experiencia puede llevar a muchos desarrolladores a hacer uso de ellos sin saber que realmente los están utilizando, sobre todo si se trabaja con Spring o algún framework similar. No veo mucho problema en que pase algo así, pero sí veo problema en mencionar que en cierto proyecto se está utilizando "inversión de dependencias" cuando realmente se está utilizando "inyección de dependencias".

Otro motivo de confusión, a mi parecer, es que en foros, blogs, etc, se suele tratar el tema en el orden equivocado: IoC => DI => DIP. Yo haré justo lo contrario, y comenzaré explicando el Principio de Inversión de dependencias.

## Principio de Inversión de Dependencias

Este principio, que no sé al 100% si acuñado, pero sí popularizado por [Uncle Bob](http://en.wikipedia.org/wiki/Robert_Cecil_Martin), tiene la siguiente [definición formal oficial](http://en.wikipedia.org/wiki/Dependency_inversion_principle):

>A. High-level modules should not depend on low-level modules. Both should depend on abstractions.

>B. Abstractions should not depend on details. Details should depend on abstractions.

Diría que esta definición por sí misma no sirve de mucho, y para entenderlo en condiciones necesitamos un ejemplo plasmado en código y acompañado de algún diagrama UML.

#### Dependencias en tiempo de compilación / de ejecución

Para entender este principio primero hay que aprender a distinguir la diferencia entre dependencias en tiempo de compilación (compile time dependencies) y dependencias en tiempo de ejecución (runtime dependencies).

Utilizaremos un ejemplo simplicado para un sistema de inventario. Tenemos tres clases principales:

{% highlight java %}
public class Item {

    private final String name;
    private final BigDecimal price;
    private final int quantity;

    public Item(String name,
                BigDecimal price,
                int quantity) {
        this.name = name;
        this.price = price;
        this.quantity = quantity;
    }
    //.. getters, setters, etc
}

public class ItemDAO {
    public final List<Item> items;

    public ItemDAO() {
        items = Lists.newArrayList(
                new Item("item1", BigDecimal.valueOf(10.00), 2),
                new Item("item1", BigDecimal.valueOf(20.00), 1),
                new Item("item1", BigDecimal.valueOf(30.00), 3));
    }

    public List<Item> getItems() {
        return items;
    }
}

public class InventoryService {

    private ItemDAO itemDAO;

    public InventoryService() {
        itemDAO = new ItemDAO();
    }

    public BigDecimal getTotalAmount() {
        List<Item> items = itemDAO.getItems();

        BigDecimal totalAmount = BigDecimal.ZERO;

        for (Item item : items) {
            totalAmount = addItemAmountToTotal(item, totalAmount);
        }

        return totalAmount;
    }

    private BigDecimal addItemAmountToTotal(Item item, BigDecimal totalAmount) {
        BigDecimal price = item.getPrice();
        int quantity = item.getQuantity();

        BigDecimal amountItem =
                price.multiply(BigDecimal.valueOf(quantity));

        totalAmount = totalAmount.add(amountItem);
        return totalAmount;
    }

}
{% endhighlight %}

En el código vemos claramente como `InventoryService` instancia `ItemDAO`, y posteriormente invoca su método `getItems` para obtener todos los productos del inventario y calcular el importe total del almacén. Esto significa que el flujo de control en tiempo de ejecución lleva la dirección que va desde `InventoryService` a `ItemDAO`. Esto no es otra cosa que una **dependencia en tiempo de ejecución** (runtime dependency).

Dejando de lado `Item`, que es una clase "de modelo" (existen mil formas de denominar estas clases: de dominio, business objects...), las clases que implementan el comportamiento del sistema son: `InventoryService` e `ItemDAO`. El diagrama UML que representa la relación entre ambas clases sería:

![UML sin DIP](/public/pictures/inventory_nodip.jpg)

En este diagrama vemos cómo hemos modelado el sistema y cuáles son las **dependencias en tiempo de compilación**. ¿Qué significa esto? Significa que para compilar `InventoryService` necesitamos haber compilado previamente `ItemDAO`. Si ambas clases estuvieran desarrolladas en diferentes módulos/librerías, y por tanto formaran parte de ficheros jar diferentes (pongamos por ejemplo, un fichero *services.jar* y otro *daos.jar*), para poder compilar el módulo que contiene `InventoryService` previamente debería estar compilado y disponible en el classpath el módulo que contiene `ItemDAO`.

En este ejemplo, por tanto, las dependencias en tiempo de compilación tienen el mismo sentido que las dependencias en tiempo de ejecución. Esto tiene varias implicaciones:

* Estamos acoplando fuertemente módulos con diferentes responsabilidades (acceso a datos -DAO- vs. cálculos de negocio -Service-)
* El proceso de compilación y despliegue del sistema es más complejo (puesto que es necesaria la presencia de todas las dependencias)
* Los tests son mucho más complicados de desarrollar (es difícil aislar las clases a testear mediante mocks)

Imaginad los problemas que conlleva este acoplamiento en sistemas realmente complejos, de, digamos, cientos de módulos.

#### Invirtiendo las dependencias

¿Qué significa invertir las dependencias? Pues ni más ni menos que diseñar el sistema de forma que **las dependencias en tiempo de ejecución tengan el sentido contrario que las dependencias en tiempo de compilación**. Esto se consigue estableciendo un contrato entre los módulos de alto nivel y los módulos de bajo nivel. En Java, un contrato no es ni más ni menos que una interfaz.

En el sistema anterior, introducir una interfaz nos llevaría al siguiente diagrama UML:

![UML con DIP](/public/pictures/inventory_dip.jpg)

Aquí vemos que `InventoryService` depende en tiempo de compilación de la interfaz `ItemDAO` (nótese que en la primera versión `ItemDAO` era una clase, no una interfaz), y dicha interfaz es implementada por la clase `ItemDAOImpl`, pero `InventoryService` no dependerá más de dicha clase `ItemDAOImpl`.

Por tanto hemos desacoplado ambas clases, que se comunican entre ellas mediante un contrato (establecido en la interfaz `ItemDAO`). En tiempo de ejecución, sin embargo, la clase `InventoryService` seguirá invocando a la implemantación `ItemDAOImpl`, por lo que, efectivamente, está ocurriendo que "la dirección del flujo de ejecución es opuesta a la dirección de las dependencias en tiempo de compilación". En esto consiste el Principio de Inversión de Dependencias.


#### ¡Pero si la dependencia en tiempo de compilación sigue ahí!

En el punto anterior he omitido el código resultante del nuevo diseño a propósito. Lo primero que puede venir a nuestra mente tras ver el diagrama UML es lo siguiente:

{% highlight java %}
public class InventoryService {

    private ItemDAO itemDAO;

    public InventoryService() {
        itemDAO = new ItemDAOImpl();
    }
    //...
}
{% endhighlight %}

En este código siguen existiendo dependencias en tiempo de compilación, porque la clase Service sigue siendo responsable de instanciar la implementación concreta del contrato. Y aquí es donde entra en juego la inyección de dependencias.

## Inyección de Dependencias

El código que implementa el último diagrama UML ajustándose realmente al DIP sería este:

{% highlight java %}
public interface ItemDAO {
    List<Item> getItems();
}

public class ItemDAOImpl implements ItemDAO{
    public final List<Item> items;

    public ItemDAOImpl() {
        items = Lists.newArrayList(
                new Item("item1", BigDecimal.valueOf(10.00), 2),
                new Item("item1", BigDecimal.valueOf(20.00), 1),
                new Item("item1", BigDecimal.valueOf(30.00), 3));
    }

    @Override
    public List<Item> getItems() {
        return items;
    }
}

public class InventoryService {

    private final ItemDAO itemDAO;

    public InventoryService(ItemDAO itemDAO) {
        this.itemDAO = itemDAO;
    }

    public BigDecimal getTotalAmount() {
        List<Item> items = itemDAO.getItems();

        BigDecimal totalAmount = BigDecimal.ZERO;

        for (Item item : items) {
            totalAmount = addItemAmountToTotal(item, totalAmount);
        }

        return totalAmount;
    }

    private BigDecimal addItemAmountToTotal(Item item, BigDecimal totalAmount) {
        BigDecimal price = item.getPrice();
        int quantity = item.getQuantity();

        BigDecimal amountItem =
                price.multiply(BigDecimal.valueOf(quantity));

        totalAmount = totalAmount.add(amountItem);
        return totalAmount;
    }
}
{% endhighlight %}

Como podemos ver, ahora sí, la clase `InventoryService` es completamente ignorante de la existencia de una implementación concreta del contrato establecido por la interfaz `ItemDAO`, y lo único que hace es recibir la implementación que sea en su constructor y guardar la referencia. Estamos, por tanto, **inyectando** la dependencia DAO en la clase Service.

Esto es fantástico, porque hemos borrado de un plumazo todos los problemas detallados en la lista de la primera versión del sistema (acoplamiento, tests, compilación y despliegue), pero ahora un nuevo quebradero de cabeza entra en juego (de hecho, en este punto del razonamiento aún no estamos utilizando inyección de dependencias como tal).

Con la primera versión, si queríamos invocar la operación `getTotalAmount` del servicio, bastaba con instanciar la clase y ejecutar la operación. Por simplicidad, hagámoslo en un método main estático dentro de la propia clase, es decir:

{% highlight java %}
public class InventoryService {

    //....
    public static void main(String[] args) {
        InventoryService inventoryService = new InventoryService();
        BigDecimal totalAmount = inventoryService.getTotalAmount();
        System.out.println(totalAmount);
    }
}
{% endhighlight %}

En la nueva versión, sin embargo, si queremos que llevar a cabo la misma operación, "alguien" deberá encargarse de instanciar la implementación concreta de `ItemDAO`, para posteriormente instanciar `InventoryService` pasándole la instancia del DAO. Creemos nosotros mismos una clase con esa responsabilidad:

{% highlight java %}
public class Assembler {

    public static void main(String[] args) {
        ItemDAO itemDAO = new ItemDAOImpl();
        InventoryService inventoryService = new InventoryService(itemDAO);
        BigDecimal totalAmount = inventoryService.getTotalAmount();
        System.out.println(totalAmount);
    }

}
{% endhighlight %}

Esta clase tiene la responsabilidad de instanciar y ensamblar las diferentes piezas del sistema (de ahí su nombre). Es la presencia de este componente el que, ahora sí, nos permite afirmar que nuestro sistema funciona mediante **Inyección de Dependencias**.

El concepto fue acuñado por [Martin Fowler](http://es.wikipedia.org/wiki/Martin_Fowler) en [este artículo](http://es.wikipedia.org/wiki/Martin_Fowler), y en resumen viene a decir que los sistemas diseñador mediante Dependency Injection están controlados por un componente externo que se encarga de instanciar y ensamblar todas sus piezas convenientemente. En general, nosotros jamás desarrollaremos una clase como la `Assembler` de mi ejemplo, y dejaremos esta responsabiliad a frameworks como [Spring](http://projects.spring.io/spring-framework/).

#### Añadiendo Spring al sistema

Veamos como quedaría nuestro sistema utilizando Spring. El core de Spring lo forma el conocido como "Inversion of Control Container" ([IoC container](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html#beans-basics)), o más comúnmente denominado **Application Context**. No es mi intención profundizar aquí en cómo funciona Spring, pero, para posibles lectores que nunca hayan trabajado con el framework, el contexto de Spring puede verse a rasgos generales como un contenedor que almacena instancias de clases Java denominadas beans. Dichos beans están etiquetados con un identificador, su clase y la interfaz que implementan, de forma que cuando una clase necesita hacer uso de una instancia que implemente un contrato determinado, Spring realiza una búsqueda en su contexto de un bean que cumpla las características requeridas e inyecta el bean en la clase que lo solicite.

Repito, esto es una simplifación bastante grande, pero creo que será suficiente para entender el ejemplo. En Spring existen múltiples formas de configurar el contexto de la aplicación, pero aquí optaré por la más primitiva (y para mí la mejor :)), que es hacerlo mediante un fichero XML:

{% highlight xml %}
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	    http://www.springframework.org/schema/beans/spring-beans-4.0.xsd">

    <bean id="itemDAO" class="com.raulavila.dependencies.dip.withdip.ItemDAOImpl" />

    <bean id="inventoryService"
            class="com.raulavila.dependencies.dip.withdip.InventoryService">
        <constructor-arg ref="itemDAO"/>
    </bean>
</beans>
{% endhighlight %}

Aquí vemos como le estamos diciendo al IoC container que instancie los beans "itemDAO" e "inventoryService", y que inyecte itemDAO en inventoryService.

La clase principal de nuestra aplicación ahora sería bastante diferente:

{% highlight java %}
public class MainApp {

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext(
                "spring-beans.xml");

        InventoryService inventoryService =
            context.getBean("inventoryService", InventoryService.class);
        BigDecimal totalAmount = inventoryService.getTotalAmount();
        System.out.println(totalAmount);
    }
}
{% endhighlight %}

Como se puede ver, lo único que instancia es el contexto de Spring, y a partir de ahí solicita los servicios que necesita a dicho contexto.

## Inversión de Control (Inversion of Control)

Este concepto es confundido habitualmente con la Inyección de Dependencias (la propia documentación de Spring lo hace), pero en realidad es bastante más amplio.

Inversión de control viene a decir que el control principal de la aplicación no está gestionado por la aplicación misma, sino por un agente externo, consiguiendo así un código más desacoplado. Es una definición bastante genérica, y se puede aplicar a muchos aspectos del desarrollo software, veamos dos ejemplos:

* Una aplicación que solicita datos al usuario puede hacerlo a la vieja usanza (y hablo de MS-DOS), solicitando un dato cada vez (es decir, solicitar el nombre, esperar la acción del usuario, solicitar apellidos, esperar la acción del usuario, etc), o puede utilizar un framework GUI que mediante eventos guarde los valores de cada dato en el objeto que corresponda cuando estos datos se modifican en los campos correspondientes. Hemos invertido el control de la aplicación, siendo ahora el GUI quien gestiona el flujo del proceso (programación mediante eventos)
* Una aplicación está compuesta de diferentes piezas que deben ser ensambladas convenientemente antes de poder ser utilizadas. Esta labor de ensamblaje puede ser realizada por la clase Main de la aplicación antes de iniciar el hilo principal, o puede ser externalizada a un contenedor externo, invirtiendo por tanto el control de este proceso

Efectivamente, el segundo ejemplo no es otra cosa que la inyección de dependencias, que podríamos considerar como un subconjunto de la inversión de control, pero **nunca la misma cosa**.

En general, diría que el uso del concepto "Inversión de Control" es bastante difuso, y es preferible olvidarnos de ello en nuestro día a día para evitar confusiones, mencionando y haciendo uso de la inyección de dependencias con verdadera propiedad.


## DI sin DIP

Es posible que un sistema esté diseñado para funcionar mediante inyección de depedencias, ¡y que no siga el principio de inversión de depedencias! Aplicándolo a nuestro ejemplo, si modificáramos la clase `InventoryService` como sigue:

{% highlight java %}
public class InventoryService {

    private final ItemDAOImpl itemDAO;

    public InventoryService(ItemDAOImpl itemDAO) {
        this.itemDAO = itemDAO;
    }
    //....
}
{% endhighlight %}

Estamos creando una depedencia en tiempo de compilación desde `InventoryService` hacia `ItemDAOImpl`. El hecho de que este sistema no respete el DIP no implica que el diseño sea horrible, porque la inyección de dependencias sigue ahí, y por tanto aún podremos crear tests unitarios en condiciones, que era uno de los principales hándicaps de la primera versión comentada más arriba.

Tratar de seguir a rajatabla el DIP puede llevar a sistemas excesivamente dimensionados (sobreingenería lo llaman) en ocasiones, y es labor nuestra, y de nuestro sentido común, dar con soluciones equilibradas. No siempre es necesario crear contratos/interfaces entre todas las clases de diferentes niveles de nuestro sistema, aunque existen determinados patrones que siempre es recomendable seguir (un buen ejemplo serían las clases DAO, para cuyas implementaciones encuentro difícil de justificar la no creación de una interfaz).

Como contraejemplo de esto que estoy hablando, pongamos que creamos una calculadora para abstraer las operaciones con BigDecimal's (un ejemplo un poco tonto). En tal caso la clase `InventoryService` podría quedar así:

{% highlight java %}
public class InventoryService {

    private final ItemDAO itemDAO;
    private final BigDecimalCalculator bigDecimalCalculator;

    public InventoryService(ItemDAO itemDAO,
                            BigDecimalCalculator bigDecimalCalculator) {
        this.itemDAO = itemDAO;
        this.bigDecimalCalculator = bigDecimalCalculator;
    }

    public BigDecimal getTotalAmount() {
        List<Item> items = itemDAO.getItems();

        BigDecimal totalAmount = BigDecimal.ZERO;

        for (Item item : items) {
            totalAmount = addItemAmountToTotal(item, totalAmount);
        }

        return totalAmount;
    }

    private BigDecimal addItemAmountToTotal(Item item, BigDecimal totalAmount) {
        BigDecimal price = item.getPrice();
        BigDecimal quantity = BigDecimal.valueOf(item.getQuantity());

        BigDecimal amountItem = bigDecimalCalculator.multiply(price, quantity);
        totalAmount = bigDecimalCalculator.add(totalAmount, amountItem);

        return totalAmount;
    }
}
{% endhighlight %}

Omito la implementación de la nueva clase por obvia. En este caso no le veo mucho sentido a crear un contrato para BigDecimalCalculator, y sí a crear una única implementación a inyectar en las clases que lo necesiten. Otra cosa sería que creáramos una clase genérica `Calculator<T>`, pero esa es otra historia :).

Tras añadir esta última clase al sistema, el contexto de spring quedaría configurado así:

{% highlight xml %}
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
	    http://www.springframework.org/schema/beans/spring-beans-4.0.xsd">

    <bean id="itemDAO" class="com.raulavila.dependencies.dip.withdip.ItemDAOImpl" />
    <bean id="bigDecimalCalculator"
        class="com.raulavila.dependencies.dip.withdip.BigDecimalCalculator"/>

    <bean id="inventoryService"
        class="com.raulavila.dependencies.dip.withdip.InventoryService">
        <constructor-arg ref="itemDAO"/>
        <constructor-arg ref="bigDecimalCalculator"/>
    </bean>
</beans>
{% endhighlight %}

#### Inyección mediante setters

La inyección no tiene por qué hacerse en el constructor. Es posible exponer los atributos de la clase mediante setters, y en ese caso la configuración en el fichero XML es incluso más clara:

{% highlight java %}
<bean id="inventoryService"
    class="com.raulavila.dependencies.dip.withdip.InventoryService">
    <property name="itemDAO" ref="itemDAO"/>
    <property name="bigDecimalCalculator" ref="bigDecimalCalculator"/>
</bean>
{% endhighlight %}

Omito la nueva versión de la clase `InventoryService`, pero sería necesario crear los correspondientes métodos set para los atributos, así como eliminar su naturaleza `final` y crear un constructor por defecto.

Esto nos lleva a un debate sin fin sobre si es mejor una modalidad o la otra. Personalmente esta última es bastante más clara para leer los ficheros de configuración (porque en la etiqueta property hay que mencionar también el nombre del atributo que almacenará la referencia, bastante más claro en clases con muchos atributos), pero elimina la naturaleza inmutable de las clases, y puede llevarnos a crear objetos "a medias". En [este artículo](http://spring.io/blog/2007/07/11/setter-injection-versus-constructor-injection-and-the-use-of-required/) de la web de Spring se discute sobre el tema por si queréis profundizar.

## Conclusiones

Puede que me haya extendido un poco más de la cuenta, pero yo creo que este es un tema con bastante miga. He dejado de lado aspectos más avanzados de la inyección de dependencias, como puede ser la [inyección mediante anotaciones](http://blogs.sourceallies.com/2011/08/spring-injection-with-resource-and-autowired/), y también me gustaría mencionar otros frameworks que sirven a la misma causa (no solo de Spring vive el hombre), como [Apache Aries Blueprint](http://aries.apache.org/modules/blueprint.html), [Google Guice](https://github.com/google/guice) o [PicoContainer](http://picocontainer.codehaus.org/).

(El código fuente, como siempre, en [GitHub](https://github.com/raulavila/blog-examples)).
