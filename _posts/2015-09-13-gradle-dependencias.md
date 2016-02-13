---
layout: post
title: Resolución de dependencias en Gradle
permalink: 2015/09/gradle-dependencias/
tags:
- desarrollo
- herramientas
- integracion-continua
comments: true
---

En el [anterior post](/2015/09/gradle-desde-maven) introdujimos Gradle como herramienta de desarrollo y gestión de proyectos software. Comentaba al final del artículo que no me convencía demasiado la forma en que se llevaba a cabo la resolución de dependencias, así que intentaré explicar a qué me refiero exactamente.

<!--break-->

###Dependencias en Maven

Para comprender cuál es el problema en Gradle, primero tenemos que revisar cómo se resuelven las dependencias en Maven. Para ello, en primer lugar es necesario tener bien clara la diferencia entre una versión `SNAPSHOT` y una que no lo es.

Supongamos que estamos trabajando con un proyecto Maven que contiene las siguientes dependencias:

{% highlight xml %}
<dependency>
    <groupId>com.raulavila</groupId>
    <artifactId>artifact1</artifactId>
    <version>1.0</version>
</dependency>

<dependency>
    <groupId>com.raulavila</groupId>
    <artifactId>artifact2</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
{% endhighlight %}

El sufijo `SNAPSHOT` básicamente significa "versión en desarrollo", algo que Maven traduce como "utilizar la versión más reciente de la dependencia". Esto ocurre con `artifact2`, pero no con `artifact1`. Con este último, al tratarse de una versión liberada, Maven utilizará la copia de la dependencia que tenga más a mano.

¿Qué significa esto de que tenga más a mano? Maven busca las dependencias en todos los repositorios que tenga configurados, comenzando por el repositorio local de la máquina que esté construyendo el proyecto, siguiendo por el repositorio Maven Central ([http://repo1.maven.org/maven/](http://repo1.maven.org/maven/)), y yendo por último a todos los repositorios que nosotros hayamos añadido en la configuración del proyecto (fichero `pom.xml`) o el fichero `settings.xml` de Maven. El siguiente diagrama representa a nivel general el proceso:

![Dependencias Maven 1](/public/pictures/dependencias/dep-maven-1.jpg)

Si la dependencia no está en el repositorio local, una vez localizada en alguno de los remotos será copiada allí (= cacheada), por lo que en siguientes builds nos ahorraremos ancho de banda utilizando esta copia de la caché. Para versiones liberadas (sin el sufijo `SNAPSHOT`) esto es ideal (asumo que nadie va a liberar la misma versión dos veces, ya que aparte de romper la lógica de versionado habría que hacer bastantes triquiñuelas para conseguirlo con Maven). El "problema" surge cuando utilizamos versiones `SNAPSHOT`:

![Dependencias Maven 2](/public/pictures/dependencias/dep-maven-2.jpg)

Supongo que habréis notado que uno de nuestros repositorios tiene el nombre "CI internal repository (shared remote repo)". En general, para proyectos corporativos, y si utilizamos prácticas como integración continua, lo normal es que tengamos configurado un repositorio remoto público a nivel interno ([Artifactory](http://www.jfrog.com/artifactory/) o [Nexus](http://www.sonatype.com/nexus/product-overview) son dos de las plataformas utilizadas). El cometido de estos repositorios es tanto servir como proxies para repositorios externos, como alojar los módulos desarrollados en el proyecto por los equipos de desarrollo. En el último diagrama, tenemos dos módulos/dependencias diferentes, `Artifact3` y `Artifact2`. Ambos módulos pueden ser desarrollados perfectamente por equipos diferentes, de forma que el equipo desarrollando `Artifact3` dependerá del trabajo realizado en `Artifact2`, pero no al revés. Además, necesitaríamos que la versión de `Artifact2` utilizada sea siempre la más reciente que exista en en el sistema de control de versiones.

La solución "chapuzas" sería que el equipo trabajando en `Artifact3` tenga también una copia local del código de `Artifact2` (aunque no trabaje en ella), y antes de construir su proyecto actualice el código del otro, instalando el módulo final en el repositorio local de Maven. Pero nosotros no somos chapuzas, y para abstraernos totalmente del trabajo realizado por el equpo `Artifact2`, configuramos el repositorio "CI internal repository" en cada copia local de Maven (y del entorno de integración continúa, claro), para que Maven busque allí la copia más reciente de la versión `1.0-SNAPSHOT`. La comprobación de la copia más reciente se hace en base a la fecha y el checksum de los ficheros. ¡Ojo!, que esta copia puede estar en el repositorio local de Maven, y no en uno de los remotos.

Para que este proceso funcione sin fisuras es necesario que nuestra herramienta de integración continua (véase [Jenkins](https://jenkins-ci.org/)) publique la versión `SNAPSHOT` en `CI repo` tan pronto como detecte una nueva versión en el repositorio CVS (véase [Git](https://git-scm.com/)). Es por esto que, en el primer diagrama, las flechas entre "Local Maven repository" y "CI internal repository" son bidireccionales, porque la versión de Maven que utilice Jenkins tendrá una tarea encargada de publicar el módulo en dicho repositorio.

###Dependencias en Gradle

Tras comprender (o eso espero) la resolución de dependencias en Maven, pasemos a ver cómo funciona Gradle. Primero, y más importante, es que en los proyectos Gradle lo normal ¡será configurar repositorios Maven para buscar las dependencias! Esto es debido es que no existe algo así como "repositorios Gradle". La otra alternativa es utilizar repositorios [Ivy](http://ant.apache.org/ivy/history/latest-milestone/tutorial/build-repository.html), aunque está menos extendida.

Total, que nuestro fichero `build.gradle`, utilizando las pautas dictadas [en su documentación](https://docs.gradle.org/current/userguide/artifact_dependencies_tutorial.html) tendrá un aspecto tal que así:

{% highlight groovy %}
//...
apply plugin: 'maven'

//...
repositories {
    mavenLocal()  //Maven installation required in the same machine
    maven {
        url 'http://ci-repo'
    }
    mavenCentral()
}

//...
dependencies {
    //...
    compile "com.raulavila:artifact2:1.0-SNAPSHOT"
}

{% endhighlight %}

El closure `repositories` define todos los repositorios donde buscar dependencias. En Gradle no hay repositorios configurados por defecto, por lo que si no especificamos `mavenLocal()` no se buscará allí. Los repositorios se escanean en el orden que aparecen, y gana el primero que contenga la dependencia.

Otra gran novedad es que, de la misma forma que el repositorio local de Maven servía de caché para dependencias externas, en Gradle existe una caché también, propia de la herramienta (y cuya ubicación se puede especificar mediante la variable de entorno [GRADLE_USER_HOME](http://blog.james-carr.org/2011/05/04/setting-gradle-cache-to-a-common-location/)). Esta caché, sin embargo, no actúa de fachada para todos los repositorios, sino **solo para los externos**. Esto, traducido, significa que Gradle añadirá las dependencias al classpath del proceso de build que existan en la máquina local, por lo que si una dependencia está en el repositorio local de Maven se añadirá desde allí, y si, en cambio, está en un repositorio remoto, se copiará a la caché de Gradle antes de añadirla al classpath.

![Dependencias Gradle 1](/public/pictures/dependencias/dep-gradle-1.jpg)

Sigamos con más diferencias. En Gradle existen dos tipos de dependencias, dinámicas y estáticas. Una dependencia es marcada como dinámica si lo hacemos nosotros de forma explícita en la configuración de la dependencia:

{% highlight groovy %}
dependencies {
    compile group: "group", name: "projectA", version: "1.1-SNAPSHOT", changing: true
}
{% endhighlight %}

Pero Gradle considera por defecto a todas las dependencias `SNAPSHOT` como versiones dinámicas. Cuando una dependencia es dinámica vive en la caché de Gradle durante 24 horas por defecto, y durante su tiempo de vida Gradle no chequeará la existencia de nuevas versiones en repositorios remotos. Este comportamiento se puede cambiar, y lo veremos a continuación. Las dependencias estáticas se consideran "fijas" y no deberían dar problemas una vez cacheadas.

Con estas bases, veamos diferentes casos de uso en los que Gradle deberá resolver la dependencia `Artifact2`:

1. La dependencia está en `mavenLocal`. Sencillo, Gradle añade la dependencia al classpath del build desde el repositorio local de Maven, sin copiarla en su propia caché
2. La dependencia no está en `mavenLocal`, pero si en el `CI repo` (primera vez que ejecutamos el build). En este caso Gradle chequea `mavenLocal` primero, y al no encontrar allí la dependencia va a `CI repo`. Encuentra allí la dependencia, así que la descarga y copia en la caché, añadiéndola al classpath desde allí
3. Mismo caso que 2 (dependencia no en `mavenLocal`, sí en `CI repo`), pero en esta ocasión es la segunda vez que ejecutamos el build. Con la configuración por defecto de Gradle, si este segundo build es ejecutado en las siguientes 24 horas después del primero, la dependencia de la caché de Gradle se utilizará en lugar de la que haya en `CI repo` (¡aunque esta última sea más reciente!). Si este segundo build se ejecuta tras 24 horas, entonces se descargaría una nueva versión de `CI repo`
4. Nos descargamos el código de `Artifact2` y lo instalamos en `mavenLocal` (mediante el comando `gradle clean install`). En el siguiente build se dará el caso 1 por tanto

El proceso aparece descrito en el siguiente diagrama:

![Dependencias Gradle 1](/public/pictures/dependencias/dep-gradle-2.jpg)

###Problemas

Este proceso presenta varios problemas:

* (basado en el punto 3) Supongamos que se publica un cambio en `Artifact2` que rompe `Artifact3`. Así que un programador de nuestro equipo actualiza `Artifact3` para adaptarse al cambio. Tras esta actualización otro programador se baja los cambios de `Artifact3`, y construye el proyecto, pero tiene una versión "viva" de `Artifact2` en la caché de Gradle. El build fallará. La solución a este problema es modificar la configuración de la caché de Gradle, de forma que el tiempo de vida de los módulos sea 0:

{% highlight groovy %}
configurations.all {
    resolutionStrategy {
        cacheDynamicVersionsFor 0, "seconds"
        cacheChangingModulesFor 0, "seconds"
    }
}
{% endhighlight %}

Con esta nueva configuración, cada build chequeará los repositorios externos, y descargará una versión más reciente en caso de que la haya (si no la hay utilizará la de la caché).

Otra opción es utilizar el parámetro `--refresh-dependencies` en cada build.

* (basado en el punto 4) Pongamos que me descargo el código de `Artifact2` y lo instalo en el repositorio local de Maven. Esto es realmente peliagudo, porque Gradle va a utilizar esa versión SIEMPRE, por lo que si hay nuevos cambios en el código, y no actualizamos el proyecto `Artifact2` instalándolo de nuevo en `mavenLocal`, el build de `Artifact3` fallará.

Este último problema tiene difícil solución. He probado varias configuraciones, y ninguna lo llega a solucionar. Una de ellas, poner `mavenLocal` como última opción en el closure `repositories` es especialmente mala, ya que se produce el efecto contrario, que si queremos utilizar cambios locales sin subir, sea imposible, ya que siempre se utilizará la versión remota por delante.

Así que no nos queda otra que pelearnos de vez en cuando con este tema. Al menos el primer problema tiene solución, por lo que si no utilizamos dependencias desde la caché de Maven todo funcionará bien, aunque es improbable que no tengamos que hacerlo en algún momento u otro. Raro es el caso en que no necesitamos descargarnos el código de un módulo del que dependemos para arreglar algo, ¿verdad?

###Conclusiones

A pesar de los problemas que he descrito en el post, sigo considerando Gradle como una gran opción para gestionar y construir proyectos, pero siempre es importante conocer los puntos flacos de cualquier cosa que utilicemos. Y en entornos tan distribuidos como los que existen hoy día, estos conflictos pueden dar lugar a situaciones algo enrevesadas. Esperemos que antes o después se modifique esta estrategia para dotar de mayor solidez a la herramienta.
