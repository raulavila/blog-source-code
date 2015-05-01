---
layout: post
title: Complejidad esencial vs Complejidad accidental
permalink: 2015/05/complejidad-esencial-accidental
tags:
- buenas-practicas
comments: true
---

"¿Complejidad qué? ¿De qué demonios hablas?", pensarán muchos lectores (o al menos un porcentaje de los que me leen :)). Trataré en este post de aclarar estos conceptos, que, siendo algo omnipresente en nuestro día a día como desarrolladores de software, seguramente el hacerlos patentes nos haga reflexionar más sobre nuestas decisiones en el futuro.

<!--break-->

##Complejidad esencial

Nuestra profesión es resolver problemas. Así de simple. No es analizar requisitos, diseñar sistemas, implementar código, resolver incidencias, etc. Bueno, por supuesto que es todo eso y más, pero si hubiera que resumirlo en dos palabras, diría que "resolver problemas" es la mejor forma de hacerlo.

Cuando el cliente nos pide que le entreguemos un sistema contable antes del inicio del próximo año, nos está planteando un problema que habremos de saber resolver (por la cuenta que nos trae). Cuanto nosotros, como expertos tecnológicos, tenemos que poner límites a los requisitos que nos quieren exigir, también estamos resolviendo un problema. Cuando tenemos que implementar un front-end basándonos en unas bonitas pantallas creadas con Photoshop por un diseñador, idem. Cuando hay que liarse la manta a la cabeza para poner en marcha un ciclo de integración continua lo mismo...

Podría seguir hasta el infinito. Bien, la complejidad esencial es aquella que va inherente al problema que tenemos que resolver, ni más ni menos. Por tanto, básandonos en el párrafo anterior, la complejidad esencial de cada problema residiría en:

* Facilitar la comunicación con el cliente para la toma de requisitos
* Estimar el tiempo de desarrollo lo más acertadamente posible para poder negociar de forma conveniente las historias de usuario a filtrar
* Pelearnos con nuestras queridas hojas de estilo CSS para montar el prototipo del front-end
* Configurar las herramientas de integración continua que vayamos a utilizar, los build plans, los plugins de maven, etc

##Complejidad accidental

Al grano: complejidad accidental es aquella que añadimos nosotros mismos sobre la marcha al trabajo que estamos realizando para resolver el problema esencial, complicando su ejecución.

Podemos pensar, si queremos, que nadie va a tirarse piedras sobre su propio tejado. Esto es así "en teoría", y como diría [Homer Simpson](http://es.wikipedia.org/wiki/Homer_Simpson), "en teoría funciona hasta el comunismo". Coñas aparte, la experiencia nos ha dictado que es extremadamente sencillo que las cosas se compliquen solas debido a todos los factores que influyen tanto en la toma de decisiones como en el trabajo que plasmamos a diario.

Siguiendo con el ejemplo, listemos varias formas de añadir complejidad accidental a nuestros problemas de inicio:

* Dar lugar a reuniones improductivas, de forma que, llegado el momento crítico, no se hayan definido correctamente los requisitos y se desperdicie trabajo desarrollando funcionalidades de escasa importancia, por ejemplo. Ya comenté lo que opino sobre las reuniones [en otro post](http://raulavila.com/2015/04/reuniones/)
* Estimaciones incorrectas (generalmente por debajo). En inglés se utiliza bastante el concepto de [wishful thinking](http://en.wikipedia.org/wiki/Wishful_thinking) como anti-patrón. No por pensar que algo va a salir bien saldrá bien. He vivido situaciones en las que casi todo el mundo sabía que no se llegaría a tiempo a una entrega, pero nadie tenía el valor de plantearlo de manera formal. El desastre a largo plazo está garantizado
* Intentar implementar los estilos CSS desde cero en lugar de utilizar algún framework. A otro nivel, malgastar horas-hombre de un programador senior peleándose con CSS cuando sería mucho más eficiente subcontratar a un experto que nos lo implementara en un par de semanas. Puede parecer un mayor desembolso inicial, pero diría que a medio plazo saldremos ganando
* Crear un build plan demasiado sencillo para no "perder tiempo en algo improductivo", sin darnos cuenta de que tomando esa decisión estamos acumulando una [deuda técnia](http://es.wikipedia.org/wiki/Deuda_t%C3%A9cnica) que pagaremos seguramente en fases avanzadas del desarrollo cuando haya varios programadores apresurados subiendo código constantemente al repositorio de control de versiones

En fin, podríamos seguir hasta el infinito. Como asumo que casi todos aquí somos programadores, voy a poner un último ejemplo. El problema consiste en implementar un método que sume todos los números de un array de enteros, devolviendo el resultado. Fácil, ¿verdad? Muchos harían algo así:

{% highlight java %}
public long sum(int[] numbers) {
    long result = 0;

    for (int number : numbers) {
        result = result + number;
    }

    return result;
}
{% endhighlight %}

Pero otra forma de hacerlo sería esta:

{% highlight java %}
public long sum(int[] numbers) {
    return sum(numbers, numbers.length);
}

private long sum(int[] numbers, int length) {
    if (length == 0) {
        return 0;
    } else {
        return numbers[length - 1] + sum(numbers, length - 1);
    }
}
{% endhighlight %}

Creo que no es necesario mencionar qué solución ha añadido mayor complejidad accidental, ¿verdad? Diréis que nadie implementaría este algoritmo así...y mi respuesta es que durante años desarrollando software me he encontrado con compañeros que entregaban código absolutamente enrevesado para solucionar problemas muy sencillos. En su libro [The Clean Coder](http://www.amazon.co.uk/The-Clean-Coder-Professional-Programmers/dp/0137081073) (creo recordar), Uncle Bob decía que nuestra profesión está llena de personas dispuestas a demostrar cada día que tienen un cerebro del tamaño de un planeta, y yo me he encontrado con varios casos así, por no hablar de vérmelas con código de este estilo:

{% highlight java %}
public long sum(int[] a) {
    long i = 0;

    for (int b : a) i = i + b;

    return i;
}
{% endhighlight %}

Que, efectivamente, hace exactamente lo mismo que el primer método, pero es mucho más difícil de mantener a nada que el código vaya creciendo.

##Minimizando la complejidad accidental

Igual que al principio del post afirmaba que "resolver problemas" es la mejor forma de definir nuestro trabajo, creo que también puedo plasmar en dos palabras la mejor forma de reducir la complejidad accidental: "ser valiente".

En efecto, hay que perder el miedo a plantear nuestras dudas a jefes o clientes, a protestar cuando se toman decisiones que sabemos son incorrectas, a desafiar la forma de hacer las cosas en nuestra empresa, a plantarnos en una estimación, justificando nuestros motivos, cuando se nos intente forzar a reducirla...A otros niveles, cuando un miembro del equipo demuestra de forma palpable su voluntad de ir por libre, no deberíamos dudar ni un ápice en hablar con él para dejar constancia de lo incorrecto de su actitud, y de las consecuencias negativas que tiene para el equipo, el proyecto, y para él mismo.

Seguramente estéis pensando que "esto mejor dicho que hecho". Y bueno, es posible que no sea tan sencillo llevarlo a la práctica, pero creedme, si no lo intentamos, entonces sí que no será posible. En muchas más ocasiones de las que creemos, los que están al otro lado son más receptivos de lo que creemos.
