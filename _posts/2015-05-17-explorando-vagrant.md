---
layout: post
title: Explorando Vagrant
description: Primeros pasos con Vagrant como herramiena de desarrollo
permalink: 2015/05/explorando-vagrant/
tags:
- desarrollo
- devops
comments: true
---

La configuración de los entornos de desarrollo, y los problemas derivados de la incompatibilidad entre diferentes versiones, ha sido, desde siempre, uno de los grandes problemas que hay que afrontar durante el curso de un proyecto.

Es bien sabido que la primera tarea que debemos realizar al incorporarnos a un equipo es configurar nuestro equipo con las múltiples herramientas que son necesarias para poder compilar, desplegar, testear, depurar, etc, a saber:

* JDK
* Maven
* Base de datos (PostgreSQL, MySQL...)
* Tomcat
* etc

<!--break-->

En general la instalación de todo el entorno no es trivial, siendo necesaria la configuración de puertos, usuarios, directorios de log...sumemos a esto el enorme catálogo de versiones que cada herramienta proporciona. Todo esto deriva en los siguientes inconvenientes, que además tienen un coste asociado:

* Tiempo necesario por cada programador para configurar su entorno
* Bugs surgidos del uso de diferentes versiones, por ejemplo, versiones de plugins de Maven para desplegar ficheros WAR que no funcionan de la misma forma con diferentes versiones de Tomcat
* Disaster recovery: un error en el entorno puede llevarnos a desconocer el origen y tener que instalarlo todo de cero
* Portabilidad: no es inmediato llevarse el entorno a otro equipo (por ejemplo, si trabajamos con un fijo y tenemos que ir a una presentación)

En fin, creo que todos nos hemos peleado en algún momento con alguna de estas cosas. Por otra parte, a más programadores por proyecto mayor es el coste generado por esta complejidad adicional.

El uso de máquinas virtuales vino en parte a paliar parte de estos dolores de cabeza. Nos permitían empaquetar en un archivo todo un sistema operativo con las aplicaciones que necesitáramos, y levantar la máquina con proveedores como [VMWare](http://www.vmware.com/) o [VirtualBox](https://www.virtualbox.org/). El gran hándicap de estas soluciones es que los ficheros que empaquetaban estas máquinas virtuales tienen un tamaño considerable, y la instancia de referencia debe estar alojada en algún lugar accesible a todos los miembros del equipo. Además, esta instancia debía ser configurda desde cero, y sin posibilidad de volver atrás a su configuración básica (en plan rollback).

## Vagrant

[Vagrant](https://www.vagrantup.com/) es una herramienta que mitiga prácticamente todos los problemas que acabo de describir. Es una utilidad para generar, levantar, provisionar y compartir entornos de desarrollo de manera extremadamente sencilla. A diferencia de las máquinas virtuales al uso, lo único que una desarrollador necesita para levantar una instancia local es tener instalado Vagrant, un proveedor (como [VirtualBox](https://www.virtualbox.org/)), y un fichero de nombre Vagrantfile donde queda descrita la máquina virtual que debe iniciarse. Este fichero no es más que texto plano, y por tanto puede subirse junto con el código del proyecto a un repositorio de control de versiones. Veamos el contenido de uno de estos ficheros:

{% highlight text %}
Vagrant.configure(2) do |config|
  config.vm.box = "troii/java7"
  config.vm.provision :shell, path: "bootstrap.sh"
  config.vm.network :forwarded_port, host: 4567, guest: 8080
end
{% endhighlight %}

¿Sencillo, verdad? Este fichero será el que utilice en un ejemplo que desarrollaré a continuación. Por tanto nosotros como desarrolladores, lo único que deberemos hacer el día que nos incorporemos a un proyecto o deseemos replicar el entorno es:

1. Instalar Vagrant y un proveedor (VirtualBox)
2. Instalar Git (u otro sistema de control de versiones)
3. Descargar el proyecto del repositorio
4. Lanzar el comando `vagrant up` desde el directorio donde se encuentra el proyecto (y el fichero Vagrantfile)

Para comprender mejor como funciona todo, crearemos un "Hello World" desde cero.

## Hello World!

Lo primero que tendremos que hacer es seguir las instrucciones de la [documentación oficial de Vagrant](https://docs.vagrantup.com/v2/installation/index.html), para preparar nuestro equipo. Es muy sencillo, yo al menos no tuve ningún problema

En segundo lugar tenemos que seleccionar una box de referencia dentro del catálogo que existe en [esta web](https://atlas.hashicorp.com/boxes/search). Aquí podremos localizar desde sistemas operativos con lo más básico (por ejemplo, el Ubuntu que se utiliza en la documentación, "hashicorp/precise32"), hasta otros con todo tipo de utilidades instaladas. De hecho nosotros mismos podemos empaquetar una box y subirla al catálogo para uso público o privado.

En nuestro ejemplo vamos a utilizar como referencia la máquina "troii/java7", que contiene instalados por defecto Java 7, Tomcat 7, y Maven 3.

Una vez localizada la box de partida, debemos inicializarla de forma local (creando un clon), lo cual se hace con el siguiente comando:

``
vagrant init troii/java7
``

Que nos devuelve este mensaje

>A `Vagrantfile` has been placed in this directory. You are now ready to `vagrant up` your first virtual environment! Please read the comments in the Vagrantfile as well as documentation on `vagrantup.com` for more information on using Vagrant.

En efecto, lo único que ha hecho Vagrant ha sido crear un fichero `Vagrantfile` a modo de plantilla para configurar una nueva box. Lo bueno de esto es que este fichero, ¡ya es totalmente válido! Si quisiéramos podríamos arrancar la máquina virtual mediante este comando:

``
vagrant up
``

Y tendríamos un Ubuntu "up and running". No lo vamos a hacer de momento, porque queremos tunear un poco la configuración de esta máquina para alojar y desplegar nuestra aplicación "Hello World".

### El fichero Vagrantfile

El fichero Vagrantfile creado por defecto esta lleno de comentarios sobre cómo configurarlo, y os animo a echarle un ojo. Pero, en realidad, el único contenido de utilidad es el siguiente:

{% highlight text %}
Vagrant.configure(2) do |config|
    config.vm.box = "troii/java7"
end
{% endhighlight %}

En efecto, el fichero indica a Vagrant qué box utilizar como base para arrancar la instancia virtual y poco más. Como lo que queremos es compilar y desplegar una aplicación web en un Tomcat, vamos a añadir configuración adicional al fichero para dejar todo preparado una vez iniciemos la instancia.

En primer lugar, vamos a redireccionar un puerto de nuestra máquina local para que redirija el tráfico al puerto 8080 de Tomcat. De esta forma conseguimos poder abrir desde el explorador de nuestro equipo las aplicaciones web desplegadas en el Tomcat de la máquina virtual:

{% highlight text %}
config.vm.network :forwarded_port, host: 4567, guest: 8080
{% endhighlight %}

De esta forma conseguiremos que accediendo a la dirección `http://localhost:4567` estemos accediendo al Tomcat en ejecución dentro de la máquina virtual de Vagrant.

### Provisionamiento

O provisioning, es el proceso mediante el cual preparamos mediante configuración adicional la instancia básica, y en ocasiones le añadimos software que no viene instalado por defecto. [Vagrant permite trabajar con herramientas](https://docs.vagrantup.com/v2/provisioning/index.html) como [Chef](https://learn.chef.io/) o [Puppet](https://puppetlabs.com/), pero para tareas básicas nos basta con utilizar shell scripts, como haremos en el ejemplo.

En primer lugar añadiremos la siguiente línea al fichero Vagrantfile:

{% highlight text %}
config.vm.provision :shell, path: "bootstrap.sh"
{% endhighlight %}

En el mismo nivel que Vagrantfile crearemos un script de nombre bootstrap.sh, con el siguiente contenido:

{% highlight sh %}
#!/usr/bin/env bash

cp /vagrant/bootstrap-conf/tomcat-users.xml /usr/local/apache-tomcat-7.0.54/conf
/usr/local/apache-tomcat-7.0.54/bin/startup.sh
{% endhighlight %}

Vemos que contiene dos instrucciones, la primera copia un fichero `tomcat-users.xml` en el directorio de configuración de Tomcat, sobrescribiendo el que exista. Esto lo hacemos porque, por defecto, Tomcat no expone usuarios de administración, que serán necesarios para desplegar automáticamente desde maven. El contenido del fichero lo podéis ver en mi repositorio de GitHub, pero a grandes rasgos crea un usuario *admin* sin password.

En segundo lugar arranca la instancia de Tomcat en el momento de iniciar la máquina virtual.

Seguramente os estéis preguntando dónde se encuentra la carpeta /vagrant/bootstrap-conf, la respuesta es...

### Carpetas sincronizadas

Una de las principales utilidades de Vagrant es el uso de [carpetas sincronizadas](https://docs.vagrantup.com/v2/getting-started/synced_folders.html). Por defecto, la carpeta donde se encuentra el fichero Vagrantfile se sincroniza de forma automática con la carpeta `/vagrant/` de la instancia virtual, y cualquier cambio en esos ficheros se refleja en ambos sentidos. Esto es muy útil porque, en general, trabajaremos con un IDE de nuestro ordenador, mientras que desplegaremos en la máquina virtual. En breve crearemos un proyecto Maven para verlo todo en funcionamiento.

En el punto anterior copiamos mediante un shell script el fichero tomcat-users, que realmente se encuentra en una subcarpeta /bootstrap-conf del proyecto Vagrant, y pasará a estar disponible para la máquina virtual tan pronto como ésta arranque (de hecho lo utilizamos para provisionarla).

### Arrancando la instancia

Vamos a arrancar la instancia, ejecutando desde el directorio donde se encuentra el fichero Vagrantfile:

``
vagrant up
``

El proceso puede llevar unos minutos, sobre todo la primera vez que iniciamos la máquina:

![Vagrant up](/public/pictures/vagrant/vagrant1.png)

Podemos ver en el log generado que también se ha iniciado la instancia de Tomcat de acuerdo a las instrucciones del fichero `bootstrap.sh`, perfecto. Si nos conectamos mediante nuestro explorador a la dirección `http://localhost:4567/` podremos confirmar que está operativo.

Vamos a conectarnos por SSH a la máquina virtual, algo tan sencillo como ejecutar la siguiente instrucción:

``
vagrant ssh
``

Si vamos a la carpeta `/vagrant`, veremos los ficheros sincronizados con nuestra máquina:

![Vagrant sync](/public/pictures/vagrant/vagrant2.png)

### Creando la aplicación Hello World

Dentro de la instancia virtual, vamos a crear una aplicación web "Hello World" utilizando arquetipos de Maven. Para ello invocamos el goal `mvn archetype:generate`. De todo el catálogo disponible yo seleccioné para el ejemplo el arquetipo `co.ntier:spring-mvc-archetype`.

Si todo funciona correctamente, se habrá creado una nueva carpeta con el proyecto dentro de la carpeta vagrant sincronizada. Tan solo nos quedaría añadir el plugin de Tomcat para realizar despliegues automáticamente, al fichero `pom.xml`:

{% highlight xml %}
<plugin>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <version>2.2</version>
    <configuration>
        <url>http://localhost:8080/manager/text</url>
    </configuration>
</plugin>
{% endhighlight %}

La configuración por defecto de este plugin asume la existencia de un usuario `admin` y sin contraseña (algo acorde a la configuración de nuestro Tomcat).

Ya lo tenemos todo. La ejecución de la siguiente instrucción Maven

``
mvn clean install tomcat7:deploy
``

compilará, empaquetará y desplegará el proyecto dentro del Tomcat en ejecución dentro de la máquina virtual. Pero lo mejor de todo es que si vamos en el explorador de nuestro equipo a la dirección `http://localhost:4567/spring-mvc/`, ¡¡se abrirá la apliación desplegada!!

Tenemos por tanto un proyecto que podremos subir a un repositorio de control de versiones de forma que, cualquier persona que se sume al proyecto solo tendrá que ejecutar los pasos descritos al principio de este artículo para tener un entorno totalmente operativo (cosa que no debería llevar más de 10 minutos). Los repetiré de nuevo :)

1. Instalar Vagrant y un proveedor (VirtualBox)
2. Instalar Git (u otro sistema de control de versiones)
3. Descargar el proyecto del repositorio
4. Lanzar el comando `vagrant up` desde el directorio donde se encuentra el proyecto (y el fichero Vagrantfile)

### Compartiendo entornos Vagrant de forma pública

Si os parece que todo esto es genial, aún queda alguna sorpresa. ¿Alguna vez habéis pasado una tarde entera pasando la última versión de una aplicación a un servidor público de pruebas para mostrarla en una presentación? Suele ser una tarea bastante desagradecida en ocasiones, y en general, el único cometido es discutir nuevos aspectos de la funcionalidad con el cliente.

Con Vagrant, es posible compartir el entorno en internet mediante una URL pública. Así, el contenido de nuestra máquina virtual pasa a estar disponible para todo el mundo mediante una URL proporcionada por la herramienta. Veamos los pasos a dar:

1. Registrarse (de forma gratuita) en [https://atlas.hashicorp.com/](https://atlas.hashicorp.com/)
2. Ejecutar `vagrant login`, introducir username y password
3. Ejecutar `vagrant share`, donde se nos confirmará la URL pública

![Vagrant share](/public/pictures/vagrant/vagrant3.png)

En el ejemplo vemos como se está compartiendo la máquina en la URL [http://greedy-impala-2134.vagrantshare.com](http://greedy-impala-2134.vagrantshare.com), de forma que para acceder a nuestro Hello World deberíamos ir a [http://greedy-impala-2134.vagrantshare.com/spring-mvc/](http://greedy-impala-2134.vagrantshare.com/spring-mvc/), y ¡ya está!

No sé vosotros, pero yo habría ahorrado muchísimo tiempo en el pasado de haber tenido a mi disposición una herramienta como ésta.

### Otros comandos útiles

Por supuesto, la máquina virtual puede suspenderse en cualquier momento, o incluso resetearla a su estado inicial, de forma que al levantarla de nuevo esté como nueva. Todo esto viene explicado muy bien en [la documentación](https://docs.vagrantup.com/v2/getting-started/teardown.html), y no es el cometido de este post profudizar demasiado en ello.

Por último, es posible empaquetar una instancia de referencia a partir del estado actual de nuestra máquina virtual. Es decir, que si arrancamos una instancia, la provisionamos, la instalamos distintas aplicaciones manualmente, etc, y queremos guardar la box en ese estado, bastaría con ejecutar este comando

``
vagrant package
``

para tener listo un clon que podrán utilizar otros usuarios. Las opciones de package están explicadas [aquí](http://docs.vagrantup.com/v2/cli/package.html).

### Conclusiones

Las posibilidades de Vagrant son muy amplias, pero espero haber dado una buena visión general. [En mi repositorio de GitHub](https://github.com/raulavila/vagrant-hw) está el proyecto completo, por si queréis tomarlo como punto de partida. Ya sabéis: `vagrant up`, ¡y a correr!
