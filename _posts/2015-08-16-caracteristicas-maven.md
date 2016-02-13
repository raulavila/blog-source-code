---
layout: post
title: Características de Maven que (quizás) no conocías
permalink: 2015/08/caracteristicas-maven/
tags:
- desarollo
- maven
comments: true
---

[Maven](https://maven.apache.org/) es, de facto, la herramienta Java para construir y gestionar proyectos de desarrollo. En su día ocupó el lugar de [Apache Ant](http://ant.apache.org/), debido sobre todo a su modelo "convention over configuration". La gran diferencia que aportaba sobre aquella es la facilidad para configurar un proyecto basándonos en estándares, que si no queríamos sobrescribir facilitaba enormemente el proceso de puesta en marcha inicial. Actualmente, [Gradle](https://gradle.org/) va ganando popularidad, y aunque personalmente le veo potencial, su flexibilidad (que remite a Ant) puede llegar a ser un hándicap a la hora de comprender una configuración determinada. Maven sigue siendo, por tanto, un must si desarrollas con Java.

En los últimos días he profundizado un poco en la documentación oficial y algún libro (estrenando mi flamante suscripción a [Safari Books Online](https://www.safaribooksonline.com)), que me ha hecho descubrir algunas características de Maven que desconocía y me parecen bastante útiles. Las voy a recoger en este post a modo de recopilación, aunque puede que no descubra nada extraordinario.

<!--break-->

###Modo offline

Esto me ha parecido toda una revelación. Bien es sabido por todos que Maven "se descarga todo Internet" la primera vez que lo ejecutamos en un proyecto. Yo pensaba que tener conexión internet era totalmente imprescindible para trabajar con Maven, y aunque en la práctica lo es, si configuramos nuestro fichero POM con todas sus dependencias es posible trabajar en modo offline, evitando que en cada build se chequeen posibles actualizaciones de dependencias, por ejemplo. Esto puede ser muy útil para trabajar fuera de casa, en un avión, por ejemplo. Las opciones que debemos conocer para utilizar este modo son dos:

1. Pasar a modo online: `mvn dependency:go-offline`. Descarga todas las dependencias del proyecto a nuestro repositorio local
2. Ejecutar goals o phases en modo offline: `mvn -o phase`, por ejemplo `mvn -o package`. Con esta opción utilizaremos exclusivamente las dependencias instaladas localmente

###Información del proyecto

El fichero `pom.xml`, centraliza toda la configuración de nuestro proyecto. A grandes rasgos, lo que todo el mundo ha de considerar es:

* Nombre del proyecto (groupId, artifactId, version)
* Modalidad de empaquetado (jar, war, ear, etc)
* Dependencias (junit, spring...)
* Plugins a utilizar en la construcción (Java compiler, para configurar la versión del JDK, por ejemplo)

La idea de los creadores de Maven es que fuera una herramienta de gestión integral de proyectos software, y para ello añadieron otro tipo de configuraciones al esquema del fichero POM que nos permiten añadir información importante, como los desarrolladores implicados con sus datos de contacto (muy práctico si se trabaja de forma descentralizada), colaboradores (importante en proyectos Open Source), links a herramientas de integración continua y gestores de incidencias, etc. De esta forma, descargándonos el proyecto tendremos todos los datos importantes relacionados con él en un solo sitio. Veamos un ejemplo:

{% highlight xml %}
<project
    xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.raulavila</groupId>
    <artifactId>maven-features</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>maven-features</name>
    <url>www.raulavila.com</url>

    <developers>
        <developer>
            <name>Raul Avila</name>
            <email>email@email.com</email>
            <roles>
                <role>Developer</role>
            </roles>
            <timezone>UTC+1</timezone>
        </developer>
    </developers>

    <contributors>
        <contributor>
            <name>Pepe Perez</name>
            <email>email@email.com</email>
            <roles>
                <role>Developer</role>
            </roles>
            <timezone>UTC+1</timezone>
        </contributor>
    </contributors>

    <issueManagement>
        <system>JIRA</system>
        <url>jira.raulavila.com</url>
    </issueManagement>

    <ciManagement>
        <system>Jenkins</system>
        <url>jenkins.raulavila.com</url>
    </ciManagement>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
    <!-- ---------
        rest of the file
    -->
</project>
{% endhighlight %}

###Maven sites

Hasta ahora no me había dado cuenta de que las páginas web de los proyectos Apache que tanto nos facilitan la vida estaban construidas con Maven. En efecto, si os fijáis en la parte inferior del menú de la izquierda en, por ejemplo, [la propia web de Maven](https://maven.apache.org/), se puede leer "Built by Maven".

Todos conocemos las fases más importantes del ciclo de vida de un proyecto Maven: clean, test, verify, package, install, etc. Pero, ¿sabiáis que existe una fase `site`? Probemos a invocar `mvn site`, y veremos que en la carpeta `target` se ha creado una subcarpeta `site`, con una web, que si abrimos tiene el siguiente aspecto:

![Maven site](/public/pictures/maven/maven1.jpg)

Este site ha sido generado a partir de un proyecto totalmente simple, añadiendo al fichero POM la información que detallé en el punto anterior. Sólo con esto se han añadido las correspondientes secciones al site, como podéis ver en el menú (Continous Integration, Issue Tracking, Project Team, etc). Veamos, por ejemplo, como queda la sección "Project Team":

![Project team](/public/pictures/maven/maven2.jpg)

Siguiendo con la filosofía "convention over configurarion" el texto introductorio de esta página viene pre-configurado, aunque es posible modificarlo, así como otros aspectos del site (encabezados, layout...). Todo esto está [perfectamente documentado](https://maven.apache.org/plugins/maven-site-plugin/), y no es mi intención aquí profundizar en ello.

Maven incluso permite levantar el site en un servidor de manera local mediante `mvn site:run` (y sería accesible desde http://localhost:8080).

####Añadiendo reportes a nuestro site

El fichero POM tiene un elemento, `<reporting>`, al que podemos añadir diferentes plugins que generarán páginas adicionales dentro de nuestro site. Las opciones son bastante amplias, por lo que vamos a ver un ejemplo combinando varias de ellas:

{% highlight xml %}
<reporting>
     <plugins>
         <plugin>
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-javadoc-plugin</artifactId>
             <version>2.10.3</version>
         </plugin>
         <plugin>
             <groupId>org.apache.maven.plugins</groupId>
             <artifactId>maven-checkstyle-plugin</artifactId>
             <version>2.6</version>
         </plugin>
         <plugin>
             <groupId>org.codehaus.mojo</groupId>
             <artifactId>cobertura-maven-plugin</artifactId>
             <version>2.7</version>
         </plugin>
         <plugin>
             <groupId>org.codehaus.mojo</groupId>
             <artifactId>dashboard-maven-plugin</artifactId>
             <version>1.0.0-beta-1</version>
         </plugin>
     </plugins>
 </reporting>
{% endhighlight %}

Hemos añadido cuatro plugins:

1. Javadoc, fácil de deducir, generará la página Javadoc con la documentación de toda nuestra aplicación
2. [Checkstyle](http://checkstyle.sourceforge.net/), herramienta de análisis estático de código, que revisa el estilo del código para ajustarlo a estándares
3. [Cobertura](http://cobertura.github.io/cobertura/), que genera informes de cobertura de tests
4. Dashboard: interesante plugin que genera una única página recogiendo un informe global de todas las herramientas de análisis estático, cobertura, etc.

Estas páginas serán generadas dentro del submenú "Project reports":

![Project reports](/public/pictures/maven/maven3.jpg)

Todos conocemos el aspecto de una página Javadoc, veamos cómo sería el informe generado por el plugin Cobertura:

![Cobertura](/public/pictures/maven/maven4.jpg)

Por último, un detalle del informe creado por el plugin Dashboard:

![Dashboard](/public/pictures/maven/maven5.jpg)

Aunque muchos de estos informes coincidirán con los de una herramienta como [SonarQube](http://www.sonarqube.org/), la gran ventaja es que no necesitaríamos instalar herramientas adicionales, por lo que Maven centralizaría todo el proceso, además de que un desarrollador podría echar un vistazo a estos informes de forma local antes de subir código al repositorio, etc.

(Como siempre, el código [está en GitHub](https://github.com/raulavila/maven-features), aunque en este caso no hay mucho en lo que profundizar :))
