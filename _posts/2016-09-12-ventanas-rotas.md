---
layout: post
title: Ventanas rotas
permalink: 2016/09/ventanas-rotas/
tags:
- desarrollo
- 5-mantras
comments: true
---

En un [post reciente](/2016/07/mis-5-mantras/) repasaba cinco mantras que tengo muy presentes a diario cuando desarrollo software. Pasé un poco por encima de todos ellos, pero me quedó la sensación de que hay mucho que rascar y en lo que debería entrar en detalle.

Así que en este post y alguno que vendrá profundizaré en todos, o casi todos, estos mantras. Comenzamos con las ventanas rotas (`Don't leave broken windows`).

<!--break-->

## Niveles (o gravedad) de ventanas rotas

Si no habéis leído [el post a que me refiero más arriba](/2016/07/mis-5-mantras/), echadle un vistazo para saber que es una ventana rota...¿ya?

Bien, como todo en esta vida, no hay nada completamente blanco ni negro, sino diferentes tonos de grises. Lo mismo ocurre con estos descuidos que deterioran nuestro código, los hay muy flagrantes, mientras que otros podrían pasarse por alto (aunque no deberíamos). Como programadores, y para obtener el máximo valor posible de nuestro tiempo, debemos saber actuar consecuentemente según la gravedad de lo que veamos.

Siguiendo con el símil del edificio, sería algo así como diferenciar entre una ventana deteriorada en la primera planta o el bajo, que es una llamada clara a los vándalos para llevar a cabo sus fechorías, frente a una ventana rota en la buhardilla de un edificio de siete plantas. Ambos desperfectos deben repararse, pero el segundo quizás pueda esperar un poco más, según las prioridades que tengamos en ese momento.

Según esto, y para simplificar al máximo, digamos que habrá tres posibles niveles de ventanas rotas:

* Grave: se trata de problemas flagrantes que, si los dejamos estar, llevarán de manera irremediable y rápida a un deterioro de nuestro código y/o proyecto. Deben repararse cuanto antes, tan pronto como sean detectados
* Medio: son problemas que saltan a la vista, pero que pueden posponerse un tiempo razonable, por ejemplo, como tareas en nuestro siguiente sprint o iteración. En ningún caso deben posponerse eternamente ("ya lo haremos en el siguiente sprint..."), pues en tal caso daría a entender que como equipo no nos preocupa excesivamente la calidad de nuestro software
* Leve: son pequeños problemas o descuidos con los que podemos vivir, y que sin embargo, la mayoría de las veces son extremadamente fáciles de resolver. No tiene sentido añadir su resolución como tareas a nuestro backlog, ya que no debería tomar más de 10 o 20 minutos. Si nuestra estimación es mayor, deberíamos considerar el problema como de nivel medio

## Ejemplos de ventanas rotas

A continuación pasaré a listar una serie de problemas que he venido viendo en mis últimos años como programador, y que entran claramente en la categoría de ventanas rotas. Muchos de ellos los he detectado en mis primeros días en el proyecto, cuando trato de familiarizarme con el código, y normalmente intento arreglarlos en la medida de lo posible antes de ponerme manos a la obra con el día a día.

### Fichero README

Tener un fichero README en nuestros repositorios de código es **fundamental**. A veces parece que la gente se agarra como un clavo ardiendo al [Agile Manifesto](http://agilemanifesto.org/) cuando dice que hay que priorizar "Working Software over comprehensive documentation", y lo traducen como "No es necesario documentar".

Esto es una atrocidad. Por favor, desde el día cero, escribid y mantened un fichero README que contenga información básica como:

* Resumen de nuestra aplicación (en dos o tres líneas, en qué consiste y qué problema resuelve)
* Cómo construir el proyecto
* Cómo y dónde desplegar el proyecto
* Cómo testear el código (automática, y en menor medida, manualmente)
* Qué dependencias existen (base de datos, message broker, etc), y cómo conectarse a ellas
* Estrategia de versionado
* Convenciones a utilizar (por ejemplo, formato de los mensajes de commit, link a guía de estilo, etc)

Seguramente me deje algunas cosas. La idea es, que si alguien nuevo en nuestro proyecto no es capaz de poner a funcionar el sistema sin un soporte constante de los empleados más veteranos (¡ojo! un mínimo soporte es necesario), entonces hay algo que falta en nuestro fichero README.

### Demasida documentación (y desactualizada)

Es el contrapunto de la sección anterior. En las primeras fases de un proyecto escribimos toneladas de documentación, conteniendo requisitos, diseño, etc, en forma de Wiki o en documentos de Word, y según aumenta la carga de trabajo no mantenemos estos documentos, convirtiéndose en una fuente de confusión importante.

En general, considero que es fundamental tener documentada la arquitectura a alto nivel de nuestro proyecto, pero los detalles de implementación deben estar en un solo lugar, el código. Por otro lado, cada vez que os veáis iniciando la escritura de un documento que sabéis llevará tiempo, pensad en el valor que estáis añadiendo, y en la "deuda de tiempo" que estáis contrayendo al escribirlo. Una deuda de tiempo no es más que una acción concreta que nos demandará la dedicación de un tiempo adicional en el futuro. [Aquí](http://jamesclear.com/time-assets) podéis leer más sobre esto.

### Código muerto (dead code)

Se trata de código que permanece en nuestro proyecto pero no es invocado por nadie, y por tanto, inútil. Mucha gente decide dejar ese código "por si lo necesito en el futuro". Yo hice esto mucho en el pasado, y la verdad es que nunca me vi recuperando dicho código. Además, existe una cosa llamada control de versiones :) que nos facilita enormemente la labor de recuperar código implementado tiempo atrás.

Este problema pertenece al grupo nivel medio. Con las herramientas actuales es muy sencillo detectar código muerto, y su eliminación es muy sencilla.

### Código comentado

Siendo un subconjunto de código muerto, esto me parece aún más aberrante. Por favor, no dejéis nunca código comentado en vuestro software.

### Tests ignorados

Se trata de tests que un día fallaron, y para continuar con lo que estábamos haciendo sin rommper nuestro entorno de integración continua, les añadimos un `@Ignore`, sin especificar claramente el motivo. Más adelante, cuando alguien ve el `@Ignore`, lo quita, y ve que el test falla, lo vuelve a poner, así que se queda ahí, para siempre.

Sólo veo un caso en el que hacer esto está justificado, cuando queremos subir código "en progreso", mientras trabajamos en una nueva funcionalidad. Dicho código no está expuesto al usuario final, y la nueva funcionalidad no estará terminada hasta que dicho tests pase. Cualquier otro caso es una ventana rota, y de gravedad.

### Utilizar varias librerías para hacer la misma cosa

Un desarrollador decide añadir una librería de utilidad para una tarea concreta, por ejemplo trabajar con JSON, y elige [Jackson](https://github.com/FasterXML/jackson).

Más tarde, otro desarrollador diferente necesita trabajar con JSON y decide añadir [GSON](https://github.com/google/gson) al proyecto. ¡Por lo que tenemos dos librerías para hacer la misma cosa!

Para evitar este tipo de problemas, es muy importante llevar a cabo [revisiones de código](/2015/03/code-reviews/), ya que en última instancia esto no es más que un problema de comunicación. Si ya es demasiado tarde, y ambas librerías llevan conviviendo un tiempo, más pronto que tarde deberíamos unificar en una por consistencia.

### Fórmato del código

¿Quién no se ha encontrado alguna vez con una clase como esta?:

{% highlight java %}
public class DishImpl implements Dish {

    private final String name;
    private boolean mixed = false;
    private boolean cooked = false;
    private final List<Ingredient> ingredientList = Lists.newArrayList();

    public DishImpl(String name)
    {
        this.name = name;
    }

    public void add_ingredient(Ingredient ingredient)
    {
      Validate.notNull(ingredient);

      System.out.printf("%s - Adding ingredient %s%n",
                        name,
                        ingredient.getName());
      ingredientList.add(ingredient);
    }

    public void mix()
    {
        if(ingredientList.isEmpty())
        {
            throw new IllegalStateException(
                "There are no ingredients to mix");
        }

        System.out.printf("%s - Mixing ingredients: %s%n",
                            name,
                            ingredientList.toString());

        mixed = true;
    }

    public void cook() {
        if(!mixed)
            throw new IllegalStateException(
                "Ingredients are not mixed, please call mix first");

        System.out.printf("%s - Cooking...%n",
                            name);

        cooked = true;
    }

    public void serve()
    {
      if(!cooked)
        throw new IllegalStateException(
            "Dish is not cooked");

      System.out.printf("%s - Serving...%n", name);

    }
}
{% endhighlight %}

Vemos ciertas incosistencias en el código, como dos formas diferentes de colocar las llaves de inicio y fin de bloque (en algunos casos está en una línea separada y en otros no), dos formas de indentar el código, dos formas de formatear sentencias `if` con una sola instrucción en su bloque (con llaves o sin ellas), diferentes convenciones para nombrar los métodos (camel case y guiones bajos), etc.

Es muy importante que todos los programadores se pongan de acuerdo en una guía de estilo a seguir, y la respeten. La máxima aquí debería ser que nadie pueda decir quién ha escrito un fragmento de código basándose en el estilo.

En Internet es muy fácil encontrar buenos ejemplos de estas guías como referencia, por ejemplo [la de Google](https://google.github.io/styleguide/javaguide.html).

### Insuficiente cobertura de tests

Si seguís la metdología TDD esto no debería suponer un gran problema, pero en caso contrario puede darse que nuestros tests solo cubran un 60-70% de nuestro código de producción (¡o aún menos!).

En los criterios de aceptación de nuestras historias de usuario debería estar siempre bien claro que una historia no se puede entregar si disiminuye la cobertura de tests, que en ningún caso debería bajar de un 90-95%. Y si baja, habría que analizar los motivos.

En cualquier caso, la cobertura de tests jamás debería ser un fin en sí mismo, sino una medida de calidad, así que cuidado con esto.

### Acciones manuales en los despliegues

El despliegue de una nueva versión de nuestro proyecto debería estar totalmente automatizado. Idealmente tendremos una tarea en nuestro entorno de integración continua que lleve a cabo todas las acciones necesarias, y que de nosotros solo exiga **pulsar un botón**. Cualquier acción adicional es un desperdicio de tiempo que debe ser analizada para su automatización.

### Entorno de integración continua inestable

Cuando nuestro entorno de integración continua está más tiempo en rojo que en verde, es que algo estamos haciendo mal. Con esto quiero decir, que el 95% del tiempo nuestras tareas (en Jenkins, Concourse, Bamboo, o lo que sea) deben ejecutarse de manera exitosa. Si esto no ocurre hay que parar cualquier trabajo que tengamos entre manos para estabilizar este problema.

### Imports no utilizados

Esto es muy típico y muy fácil de evitar. Añadimos una sentencia `import` a nuestro fichero de clase porque lo necesitamos en un momento determinado. Pero más adelante eliminamos el código (porque estamos refactorizando y nos lo llevamos a otro lado, por ejemplo), dejando allí la sentencia `import`, que es completamente inútil.

Eliminar "unused imports" es muy sencillo en los IDEs actuales. Existen atajos de teclado que lo hacen de forma instantánea, e incluso la opción de activar [opciones](http://stackoverflow.com/questions/19026756/how-configure-optimize-imports-on-the-fly-without-rearrange-import-statements) que escanean permanentemente nuestro código y eliminan los que no necesitemos.

### Dependencias no utilizadas

Mientras que el punto anterior entra dentro de los problemas leves, tener dependencias en nuestro código que han dejado de utilizarse abre la puerta a potenciales problemas que además pueden ser muy difíciles de detectar, debido a conflictos entre dependencias transitivas, por ejemplo.

Tan pronto como dejéis de utilizar una dependencia, eliminadla de vuestro proyecto (del fichero `pom` de Maven, por ejemplo).

### Estrategia de Git sin determinar

Sin haber establecido una directrices claras sobre como trabajar con Git, cada desarrollador toma sus propias decisiones, y nos encontramos con un repositorio Git plagado de ramas (muchas muertas tras ser fusionadas con master), montones de merges, etc.

No voy a proponer aquí una solución ideal para trabajar con Git, [hay varias estrategias](http://www.creativebloq.com/web-design/choose-right-git-branching-strategy-121518344) (aka workflows), pero en ningún caso debéis dejar al libre albedrío de cada persona la estrategia a utilizar.

### Código sucio

No voy a entrar en muchos detalles aquí. Casi todos sabemos lo que es código sucio, y como profesionales nunca deberíamos aceptar entregar funcionalidades desarrolladas de forma que su mantenimiento sea más complicado debido a la poca legibilidad del código. No es la primera vez que lo recomiendo (ni será la última), pero si no lo habéis hecho, leed [Clean Code](https://www.amazon.es/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882/ref=sr_1_1?ie=UTF8&qid=1473708316&sr=8-1&keywords=Clean+Code) o [Code Complete](https://www.amazon.es/Code-Complete-Practical-Costruction-Professional/dp/0735619670/ref=sr_1_1?ie=UTF8&qid=1473708342&sr=8-1&keywords=Code+Complete).

Cómo última nota en este apartado, por favor no hagáis algo de lo que yo mismo me declaro culpable de haber incurrido muchas veces en el pasado: ¡mezclar inglés y español! Idealmente todo nuestro código debería estar en inglés, pero si utilizáis español para nombrar variables o métodos nunca mezcléis, dando lugar a engendros como:

{% highlight java %}
if(setup)
    iniciar();
{% endhighlight %}

Me duelen los ojos de verlo :)

## Conclusión

Creo que he cubierto varios apartados importantes del desarrollo que no debemos descuidar. ¿Echáis alguno de menos?
