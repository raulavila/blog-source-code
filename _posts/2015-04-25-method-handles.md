---
layout: post
title: MethodHandles como alternativa a Reflection
description: MethodHandles en Java
permalink: 2015/04/method-handles/
tags:
- desarrollo
- Java
comments: true
---

En la versión 7 de Java se incluyó una nueva API llamanda [MethodHandle](http://docs.oracle.com/javase/7/docs/api/java/lang/invoke/MethodHandle.html), y que fue presentada como una alternativa a [Reflection](https://docs.oracle.com/javase/tutorial/reflect/) para poder acceder de forma dinámica a los miembros de una clase en tiempo de ejecución. Además, al parecer, el rendimiento es superior al ofrecido por Reflection, acercándose al obtenido por invocaciones directas a los métodos.

He estado jugando un poco con esta API, y comprobando si las afirmaciones sobre rendimiento son ciertas. Crearé un par de ejemplos sencillos, accediendo a métodos mediante Reflection y Method Handles, y comparando el tiempo de ejecución.

<!--break-->

### El código a testear

Tendremos un par de beans sencillos, `Item` y  `Store`:

{% highlight java %}
public class Item {

    private final String name;
    private int quantity;

    public Item(String name, int quantity) {
        this.name = name;
        this.quantity = quantity;
    }

    //getters, setters, equals, etc
}
{% endhighlight %}

{% highlight java %}
public class Store {

    private List<Item> items = Lists.newArrayList();

    public void addItem(Item item) {
        items.add(item);
    }

    public void removeItem(Item item) {
        items.remove(item);
    }

    public int size() {
        return items.size();
    }

    public List<Item> getItems() {
        return Collections.unmodifiableList(items);
    }

    @Override
    public String toString() {
        return "Store{" +
                "items=" + items +
                '}';
    }
}
{% endhighlight %}

### Acceso a miembros mediante Reflection

Creemos un par de tests en los que accederemos, por un lado, al método `getName` de `Item`, y por otro, al método `addItem` de Store.

{% highlight java %}
@Test
public void testItemReflection() throws Throwable {
    long start = System.nanoTime();

    Method method = Item.class.getDeclaredMethod("getName");
    Object output = method.invoke(table);

    long end = System.nanoTime();

    assertThat(output.toString()).isEqualTo("table");

    printExecutionTime("Reflection (item), total time", start, end);
}

@Test
public void testStoreReflection() throws Throwable {
    assertThat(store.getItems()).doesNotContain(chair);

    long start = System.nanoTime();

    Method method = Store.class.getDeclaredMethod("addItem", Item.class);
    method.invoke(store, chair);

    long end = System.nanoTime();

    assertThat(store.getItems()).contains(chair);

    printExecutionTime("Reflection (store), total time", start, end);

}
{% endhighlight %}

Supongo que en general todos estamos familiarizados con Reflection, a partir de un objeto class específico que representa una clase podemos acceder a los métodos, constructores, campos...mediante su nombre como String. Una ventaja de Reflection es que es posible incluso el acceso a atributos privados (lo cual se puede conseguir mediante el método `setAccesible`), pero para nuestro ejemplo no lo utilizaremos.

Lo bueno de Reflection es que facilita la implementación de frameworks en los que se exponen métodos de configuración dinámicos a través de ficheros de propiedades, XML's, etc, que serán resueltos en tiempo de despliegue o ejecución. La desventaja es, por supuesto, que determinados errores no serán desvelados en tiempo de compilación, por lo que su uso sólo está recomendado para casos muy específicos, o para la implementación de frameworks. Otra desventaja es la cantidad de checked exceptions que lanzan los métodos de la API, aunque en el ejemplo está oculto, ya que hemos propagado las excepciones a la declaración del método. **Esta aproximación para las excepciones no es nada recomendable en código de producción**, que quede claro :)

### Acceso a miembros mediante MethodHandle

El mismo ejemplo, utilizando Method Handles, sería:

{% highlight java %}

private MethodHandles.Lookup lookup = MethodHandles.lookup();

@Test
public void testItemMethodHandles() throws Throwable {
    long start = System.nanoTime();

    MethodHandle methodHandle =
            lookup.findVirtual(Item.class,
                                "getName",
                                methodType(String.class));

    Object output = methodHandle.invoke(table);

    long end = System.nanoTime();

    assertThat(output.toString()).isEqualTo("table");

    printExecutionTime("MethodHandles (item), total time", start, end);

}

@Test
public void testStoreMethodHandles() throws Throwable {
    assertThat(store.getItems()).doesNotContain(chair);

    long start = System.nanoTime();

    MethodHandle methodHandle =
            lookup.findVirtual(Store.class,
                                "addItem",
                                methodType(void.class, Item.class));

    methodHandle.invoke(store, chair);

    long end = System.nanoTime();

    assertThat(store.getItems()).contains(chair);

    printExecutionTime("MethodHandles (store), total time", start, end);
}
{% endhighlight %}

Las diferencias, respecto al uso de Reflection, son:

* Es necesario el uso de una clase lookup, que podemos obtener mediante el factory method `MethodHandles.lookup()`
* Esta clase lookUp [expone varios métodos](https://docs.oracle.com/javase/8/docs/api/java/lang/invoke/MethodHandles.Lookup.html), el más importante es `findVirtual` debido a su flexibilidad
* Este método `findVirtual` recibe como parámetros la clase a inspeccionar, el nombre del método como String, y una instancia de [MethodType](https://docs.oracle.com/javase/7/docs/api/java/lang/invoke/MethodType.html), que es creada mediante el método estático `MethodType.methodType`
* MethodType contiene, por orden, el tipo de retorno del método, y los tipos de los parámetros. El tipo de retorno supone una diferencia respecto a la configuración utilizada en Reflection, donde se omite
* `findVirtual` devuelve una instancia de `MethodHandle`, que será quien invocará al método del objeto junto con sus parámetros

Personalmente prefiero el diseño de esta API al de Reflection, ya que separa mejor las responsabilidades y la veo algo más clara. De hecho, siempre que he tenido que utilizar Reflection en el pasado, lo he hecho mediante una librería que la encapsula (tipo [ReflectionUtils](http://docs.spring.io/spring-framework/docs/4.0.4.RELEASE/javadoc-api/org/springframework/util/ReflectionUtils.html) de Spring).

### Los tiempos de ejecución

Comenzamos el artículo diciendo que, una de las supuestas ventajas de utilizar Method Handles es su mejor rendimiento sobre Reflection. Al menos en mis pruebas, esto no es así, veamos los resultados de los tests:

{% highlight text %}
Reflection (item), total time: 0.077939 ms
MethodHandles (item), total time: 3.331216 ms
Reflection (store), total time: 0.053087 ms
MethodHandles (store), total time: 18.233666 ms
{% endhighlight %}

Parece que los tiempos de ejecución con MethodHandles son bastante mayores a los requeridos por Reflection. Supongo que bajo determinadas circunstancias estos resultados se igualarán en cierta medida (por ejemplo, con métodos que contengan varios parámetros), pero al menos por ahora, diría que el rendimiento de la aplicación no debería ser un factor que nos influya para optar por el uso de Method Handles.

([El código completo, en GitHub](https://github.com/raulavila/blog-examples/tree/master/src/test/java/com/raulavila/methodhandles))
