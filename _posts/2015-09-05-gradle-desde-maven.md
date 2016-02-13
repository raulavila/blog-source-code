---
layout: post
title: Gradle desde Maven
permalink: 2015/09/gradle-desde-maven/
tags:
- desarrollo
- herramientas
comments: true
---

Tras hablar de [Maven](/2015/08/caracteristicas-maven/) hace un par de posts, he pensado que sería interesante realizar una introducción a [Gradle](https://gradle.org/), una alternativa con sus orígenes en proyectos basados en el lenguaje [Groovy](http://www.groovy-lang.org/). Groovy, al ser un lenguaje ejecutable en la JVM, convive perfectamente con Java, y de esta compatibilidad surgen interesantes aplicaciones (entre las que, a mi parecer, destaca el framework de testing [Spock](https://github.com/spockframework/), del que hablaremos muy pronto).

Gradle, aunque pensado inicialmente para construir proyectos Groovy, se aprovecha de esta compatibilidad con Java, permitiendo gestionar proyectos en dicho lenguaje sin ningún problema. Aprender Gradle es sencillo, pero hay que realizar un pequeño esfuerzo para obviar el modelo Maven al tomar contacto con la herramienta. Al menos eso es lo que me ocurrió a mí durante las primeras semanas que tuve que utilizarlo. En este post explicaré los conceptos básicos, y los pondré en contraste con sus contrapuntos en Maven.

<!--break-->

###Fichero build

Tras instalar Gradle en nuestro equipo (os remito a la [documentación oficial](https://docs.gradle.org/current/userguide/installation.html)), vamos a construir el típico "Hello World". Todos conocemos el fichero POM en Maven, se trata de un XML en el que configuramos las características básicas del proceso de construcción, las dependencias, etc. Al ser XML un formato tan verboso, incluso el proyecto más básico requiere de una configuración mínima, o será imposible ejecutar tareas de Maven en el directorio en cuestión.

Esto cambia radicalmente en Gradle. El fichero de build ha de tener por defecto el nombre `build.gradle` (aunque se puede cambiar mientras se indique en las opciones del comando gradle...), y, ¡puede estar totalmente vacío! En efecto, veamos qué ocurre si, dentro de un directorio que contenga dicho fichero vacío invocamos el comando `gradle`:

![Gradle - empty](/public/pictures/gradle/gradle-1.jpg)

Vemos que, aunque ha ocurrido poca cosa, el build es exitoso. También vemos que nos ofrece algo de ayuda sobre las cosas que podemos hacer, entre las que destaca "ver la lista de tareas disponibles". Veamos qué ocurre si utilizamos esta opción (`gradle tasks`):

![Gradle - empty](/public/pictures/gradle/gradle-2.jpg)

Tenemos por tanto una lista de tareas que podríamos ejecutar, en la forma `gradle <task>`, por ejemplo `gradle dependencies`. Pero, ¿qué es una tarea?

###Tareas (tasks)

La tarea es la unidad básica de trabajo en Gradle. Mientras que en Maven podríamos decir que las unidades de trabajo son las fases y los plugin goals, que son conceptos de más alto nivel, en Gradle se produce una vuelta a la base, utilizando un concepto que guarda mucha relación con algo que los más viejos del lugar conocerán: los targets de [Ant](http://ant.apache.org/). Ant fue la primera herramienta seria para construir proyectos en Java, y que destacaba por su flexibilidad. Maven impuso un modelo más rígido, que tiene sus desventajas, y Gradle ha revertido algo las tornas volviendo a la flexibilidad (que no digo que sea necesariamente buena).

Volviendo a las tareas, el contenido de un fichero build en Gradle se especifica utilizando código Groovy, lo cual nos permite hacer cualquier cosa que se nos pase por la cabeza (¡podemos ejecutar el código fuente que queramos!).

Vamos a crear una tarea muy básica:

{% highlight groovy %}
task task1 << {
    println 'Hello World!'
}
{% endhighlight %}

Lo que estamos haciendo aquí es, ni más ni menos, que inyectar una nueva tarea, de nombre `task1` en el objecto `project` (más información abajo), tarea que podremos ver en la lista devuelta por `gradle tasks`, y que por supuesto podremos invocar:

![Gradle - empty](/public/pictures/gradle/gradle-3.jpg)

¡Ya hemos creado nuestro primer proyecto Gradle! Y tan sólo con un fichero, sin necesidad de tener un layout para contener fuentes, etc. Quizás estéis algo extrañado por la notación `<<`, este operador de Groovy se encarga de inyectar acciones a la tarea, que serán ejecutadas cada vez que la invoquemos. En realidad se trata de un alias del método `doLast`, como vemos a continuación:

{% highlight groovy %}
task task1 {
    println 'Configuring task'

    doLast {
        println 'Hello World!'
    }
}
{% endhighlight %}

¿Cuál será la salida de `gradle task1` ahora?

![Gradle - empty](/public/pictures/gradle/gradle-4.jpg)

Vemos que la ejecución de cada tarea viene precedida por `:task`, ¡pero el texto "Configuring task" se ha mostrado antes! En esta ocasión, no hemos utilizado el operador `<<`, por lo que el closure pasado a task1 no es un conjunto de acciones, sino lo que se conoce como "configuration closure", y que viene a ser el equivalente al constructor de la tarea. Dentro de este constructor, mediante el método `doLast` podemos ir añadiendo acciones, que serán las que realmente se ejecuten cuando se ejecute la tarea, y todo lo que esté fuera de este método serán comandos ejecutados en tiempo de construcción. Gradle construye las tareas en su fase `init` que se ejecuta antes de pasar a cualquier otra cosa (y que podemos ejecutar nosotros mismos para ver lo ocurre, probadlo con `gradle init`). Además de `doLast`, tenemos el contrapunto `doFirst`, aunque es utilizado menos.

###Proyecto (project)

Gradle genera una instancia de la clase Project, que contendrá la configuración de todo nuestro proyecto, a saber: tareas, propiedades, dependencias, etc. Por defecto, este objeto contiene un montón de propiedades configuradas (que podemos consultar mediante `gradle properties`). Nosotros podemos añadir propiedades al gusto mediante el closure `ext`, o utilizando un fichero de propiedades (gradle.properties). Veamos como sería en el primer caso:

{% highlight groovy %}
ext {
    myproperty = 'value1'
}
task task1 << {
    println 'myproperty: ' + project.properties['myproperty']
}
{% endhighlight %}

De acuerdo a esto, `gradle task1` mostraría el texto "myproperty: value1". La referencia explícita al objecto project está de más, porque Gradle buscará por defecto cualquier referencia ahí dentro :), así que el siguiente build sería equivalente:

{% highlight groovy %}
ext {
    myproperty = 'value1'
}
task task1 << {
    println 'myproperty: ' + myproperty
}
{% endhighlight %}

###Más sobre tareas

Sabiendo ya que las tareas no son más que un fragmento de código que se ejecuta cuando es requerido en el proceso de build, otras características a conocer de las mismas son:

* Las tareas pueden depender de otras tareas (¿recordáis Ant?)
* Las tareas pueden tener una descripción, que es la que se muestra cuando son listadas por `gradle tasks`
* Podemos invocar varias tareas en un comando gradle, y se ejecutarán en orden, por ejemplo `gradle task2 task3`. Sin embargo, cada tarea se ejecutará una sola vez en cada build
* Podemos configurar una tarea por defecto en el fichero build, que será la que se ejecute si invocamos tan solo `gradle`

En el siguiente ejemplo vemos todo esto plasmado en código:

{% highlight groovy %}
defaultTasks 'task1'

task task1(description: 'Description of the task') << {
    println 'task1'
}
task task2(dependsOn: task1) << {
    println 'task2'
}
task task3 << {
    println 'task3'
}
{% endhighlight %}

A partir de este fichero build, `gradle` ejecutaría task1 (que además tiene su descripción...), y `gradle task2` ejecutaría task1 primero.

Por último, existen en Gradle unos tipos de tareas base (algo así como tareas abstractas, que en la jerga de Gradle se llaman "enhanced tasks", y no son más que una clase Groovy), que podemos utilizar para realizar tareas más complejas añadiendo las acciones necesarias en el "configuration closure" (proceso basado en el [Template Method Pattern](https://en.wikipedia.org/wiki/Template_method_pattern)). Por ejemplo:

{% highlight groovy %}
task copyTask(type: Copy) {
      from "$projectDir/file1.txt"
      into "$buildDir"
}
{% endhighlight %}

Esta tarea copiaría el fichero file1.txt, que se encuentra en el directorio de proyecto (el mismo que contiene el fichero build.gradle) dentro del directorio de build (projectDir/build por defecto). Si no conocéis Groovy, la notación `$property` reemplaza en la cadena la referencia a la variable por su contenido (y es una de las características más molonas de Groovy, por otro lado). `projectDir` y `buildDir` no son más que propiedades del proyecto.

###Plugins

Vamos a ir poniendo las cosas interesantes. Un plugin en Gradle no es otra cosa que un conjunto de tareas, dependencias, propiedades, etc, que se inyectan al objecto Project. Tan simple como eso.

{% highlight groovy %}
apply plugin: '<plugin>'
{% endhighlight %}

En general, la configuración básica de Gradle permite hacer poco, y para proyectos de verdad necesitaremos, como poco, añadir el plugin `groovy` o `java`, que añaden las tareas que todos esperamos, como se puede ver en la siguiente pantalla (salida de un proyecto que contiene el plugin `java`):

![Gradle - empty](/public/pictures/gradle/gradle-5.jpg)

Al fin, ya tenemos algo más consistente, con tareas como `test`, `build`, `jar`, etc, que creo que no hace falta explicar demasiado.

Otra cosa que este tipo de plugins añaden al proyecto es un objecto `Convention`, que contiene la configuración de layout por defecto que debería tener nuestro proyecto, y que aunque puede ser sobreescrita, no se recomienda. En el plugin `java` es la siguiente (que coincide con la de Maven, por otra parte):

* src/main/java
* src/main/resources
* src/test/java
* src/test/resources

###Dependencias

Una vez aplicamos uno de los plugin comentados arriba, podemos añadir dependencias a nuestro proyecto. Además debemos indicar a Gradle donde buscarlas:

{% highlight groovy %}
apply plugin: 'groovy'
apply plugin: 'maven'

repositories {
    mavenLocal()
    mavenCentral()
    jcenter()
}

dependencies {
    compile "org.codehaus.groovy:groovy-all:2.4.4"
    testCompile "org.spockframework:spock-core:1.0-groovy-2.4"
}
{% endhighlight %}

En esta ocasión hemos añadido el plugin `groovy` en lugar de `java`, porque vamos a crear un pequeño proyecto utilizando el framework Spock para testing. Spock está desarrollado con Groovy, por lo que es más sencillo añadir el plugin `groovy`, ya que Groovy soporta código Java sin problemas.

Lo que vemos en esta configuración es:

* Adicionalmente hemos tenido que añadir el plugin `maven`. El principal objeto de este plugin es poder utilizar repositorios maven, destacando el repositorio local (`mavenLocal()`). Gradle busca la ubiación de mavenLocal a partir del directorio base de Maven (`M2_HOME`), que deberemos tener configurado convenientemente
* Además de mavenLocal, hemos añadido como repositorios de búsqueda, mavenCentral y jcenter
* El proyecto tiene dos dependencias, la librería groovy, y el framework Spock. Es fácil de deducir que el formato para referenciar las dependencias es 'groupId:artifactId:version'

En Maven las dependencias hay que asociarlas a "scopes". En Gradle tenemos un concepto similar, conocido como "configurations", y que son añadidas por los plugins. Podemos definir una configuración como un tag que permite agrupar dependencias (u otras entidades) juntas. Una o varias configuraciones específicas son asociadas a una tarea, de forma que todos los contenidos de esas configuraciones pasan a estar disponibles para la tarea. Por ejemplo, la configuración `testCompile` se asocia a la tarea `test`.

###Todo junto

En mi repositorio de GitHub podéis encontrar [un proyecto](https://github.com/raulavila/gradle-spock-intro) muy sencillo, donde se juntan varias de las piezas comentadas en el post. No tiene demasiado misterio, pero en posteriores posts le iré añadiendo más contenido, sobre todo cuando habemos de Spock. El proyecto contiene una clase, y un test, y las tareas básicas que podemos lanzar son:

* `gradle clean`: elimina el directorio build
* `gradle test`: ejecuta las clases de test
* `gradle jar`: empaqueta el proyecto

Investigar es sencillo, echad un vistazo a otras tareas disponibles mediante `gradle tasks`.

###Últimas consideraciones

Antes de terminar, destacar algunas cosas, en plan miscelánea:

* Por defecto tenemos disponible un objeto logger, con diferentes niveles de log (info, debug, warn, lifecycle). Para utilizarlo tan solo debemos añadirlo a nuestras tareas, por ejemplo: `logger.warn "warn message"`
* Quizás hayáis notado en los pantallazos que Gradle resalta la existencia de un "daemon" que aceleraría la ejecución de nuestros builds. Este daemon no es más que un proceso arrancado en la JVM que permanece activo entre builds, por lo que ahorramos considerablemente en tiempo de inicialización. Para utilizar este demonio hay que añadir la opción `--dameon`, por ejemplo `gradle test --daemon`, y para pararlo `gradle --stop` (o matar el proceso :)).
* Mediante el comando `gradle --gui` podremos lanzar una ventana para utilizar Gradle mediante una interfaz gráfica. Si utilizamos los tradicionales IDE's es raro que utilicemos esta opción, pero siempre está bien saberlo para entornos de test o producción, por ejemplo:

![Gradle - empty](/public/pictures/gradle/gradle-6.jpg)


####Entonces, ¿Gradle es mejor que Maven?

La respuesta es que ninguna de las dos herramientas es ideal, por lo que deberemos considerar las características de nuestro proyecto antes de decantarnos por una u otra. Gradle ofrece flexibilidad ilimitada, y esto puede ser contraproducente si no tenemos la disciplina adecuada, mientras que Maven en ocasiones es extremadamente rígido, pero gracias a esa rigidez es muy sencillo asimilar rápidamente la estructura de un proyecto.

Además, Gradle es una herramienta que aún sigue madurando y tiene algunos puntos flacos. En concreto hay uno, la resolución de dependencias, que no me convence en absoluto para proyectos corporativos, y que comentaré [en mi siguiente post](/2015/09/gradle-dependencias).
