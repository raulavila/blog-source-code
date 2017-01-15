---
layout: post
title: Extract till you drop
description: La importancia de extraer y extraer cuando desarrollamos código
permalink: 2016/09/extract-till-you-drop/
tags:
- desarrollo
- 5-mantras
comments: true
---

"Extract till you drop" es otro de [mis cinco mantras](/2016/07/mis-5-mantras/), y quizás el que más presente tengo cuando estoy escribiendo código, hasta el punto de que más de un compañero ha hecho coñas con mi obsesión por extraer y extraer.

Hace unos meses tuve una discusión amigable, a la vez que acalorada, con un desarrollador del cliente. Él no era excesivamente partidario de extraer tanto como lo hago yo, y su argumento era que es mejor tener todo en el mismo sitio en lugar de tener que estar navegando entre funciones y clases para encontrar algo. Creo que no conseguí convencerle del todo de los beneficios de la extracción, pero bien es verdad que le hice recapacitar un poco al menos, y cuando terminó nuestro proyecto me prometió enviarme una botella de [mi vino favorito](https://es.wikipedia.org/wiki/Albari%C3%B1o) si con el tiempo acababa descubriendo que mi punto de vista era mejor.

Cuento esta historia a modo de curiosidad, y para que veáis que predico con el ejemplo :). Empecemos con una descripción de lo que significa extraer, a qué niveles podemos hacerlo, y cómo.

<!--break-->

En pocas palabras, extraer no es más que crear una nueva unidad de código independiente a partir de un bloque de código más monolítico. Se habla mucho hoy en día de "monolitos vs. microservicios", y personalmente creo que esta discusión se lleva demasiadas veces al nivel de arquitectura y pocas veces al nivel de codificación. Personalmente, creo que un monolítico de código es tan peligroso, o incluso más, que una aplicación monolítica, ya que daña extraordinariamente la mantenibilidad de nuestras aplicaciones.

Prácticamente todos los niveles de extracción pueden llevarse a cabo mediante las herramientas de desarrollo que utilizamos a diario (aka [IDEs](/2015/01/entornos-integrados-desarrollo/)). Por lo que, si estas funcionalidades no os son conocidas, echadle un vistazo tras terminar de leer este post.

### Extraer variables

Es el nivel más básico de extracción, y no es más que crear variables locales conteniendo el resultado de una expresión concreta en lugar de embeber dicha expresión entre los parámetros de la llamada a un método, por ejemplo:

{% highlight java %}
List<Person> people =
             Lists.newArrayList(
                     new Person("Paco", "Madrid"),
                     new Person("Pepe", "Madrid"),
                     new Person("Luis", "Barcelona"));

//...

 printNames(people.stream()
         .filter(p -> "Madrid".equals(p.getCity()))
         .map(p -> p.getName())
         .collect(toList()));
{% endhighlight %}

En el ejemplo tenemos una lista de personas creada en algún punto de nuestro código, y en algún otro punto invocamos a un método `printNames` que suponemos imprimirá en algún sitio los nombres de las personas de la lista que recibe por parámetro (detalle que, sinceramente, en este momento carece de relevancia). Por algún motivo queremos filtrar nuestra lista e imprimir únicamente los nombres de aquellas que viven en Madrid, así que utilizando una expresión lambda hacemos ese filtro. Se trata de una expresión relativamente sencilla, pero que a mi parecer complica bastante la lectura del código. Mi propuesta es hacer esto, mediante la función "extraer variable":

{% highlight java %}
List<String> namesOfPeopleInMadrid = people.stream()
                .filter(p -> "Madrid".equals(p.getCity()))
                .map(p -> p.getName())
                .collect(toList());

printNames(namesOfPeopleInMadrid);
{% endhighlight %}

Podréis argumentar que estamos creando una variable que solo se utiliza una vez, y por tanto la primera versión sería válida. Bien, mi respuesta a esto es que:

* Cuando leemos la línea con la invocación a `printNames` no necesitamos tener que leer también la forma en que hemos generado dicha lista de nombres, cosa que es necesario hacer en la primera versión
* Si la expresión lambda continúa creciendo (por ejemplo, si queremos poner el nombre en mayúsculas, filtrar por otra cosa, generalizar el filtro...), cada vez será más y más complicado leer un bloque de código que debería ser bastante sencillo, al no ser más que ¡una invocación a un método de un sólo paramétro!
* Si hacemos algo así dejamos la puerta abierta a escribir código siguiendo este estilo en otras expresiones algo más complejas, por lo que elevamos el riesgo de terminar encontrando en nuestro código expresiones como la que sigue:

{% highlight java %}
printNames(
    people.stream()
        .filter(p -> city.equals(p.getCity()))
        .map(p -> p.getName())
        .map(n -> n.toUpperCase())
        .collect(toList()),
    formats.stream()
        .filter(f -> city.equals(f.getCity()))
        .findAny()
        .orElse("Standard"));
{% endhighlight %}

En este caso, una vez haya pasado un tiempo razonable desde que escribiéramos este código, si tenemos que volver a él necesitaremos utilizar mucha mayor capacidad cognitiva para comprender lo que ocurre que si vemos algo así:

{% highlight java %}
printNames(namesOfPeopleUppercased, formatForCity);
{% endhighlight %}

Ni siquiera hace faltar retroceder al punto en el que declaramos las variables si no interesa, la expresión habla por si misma.

He mencionado el termino "capacidad cognitiva". Quiero aprovechar este punto para haceros una pregunta. ¿Cuántas veces durante una conversación se os ha pasado algo por la cabeza que queréis decir, pero por no interrumpir os habéis esperado, y cuando habéis visto el momento de hablar os habéis olvidado de ello? A mí esto me ha pasado muchas veces durante mi vida. La capacidad del cerebro para manejar diferentes piezas de información al mismo tiempo es limitada, y cuando creamos una expresión compleja como la que acabo de plasmar aquí, forzamos a utilizar más capacidad cognitiva de la que es estrictamente necesaria.

En todos los ejemplos de extraer a variables he utilizado el ejemplo de una expresión lambda, ya que se trata de un tipo de expresión que puede resultar bastante compleja. Pero, en general, yo aplico esta regla a expresiones más sencillas, como puedan ser invocaciones a métodos, y así:

{% highlight java %}
printNames(
    namesDAO.fetchNamesForCity(city),
    formatsDAO.getFormatForCity(city));
{% endhighlight %}

suelo reemplazarlo por esto:

{% highlight java %}
List<String> namesForCity = namesDAO.fetchNamesForCity(city);
Format format = formatsDAO.getFormatForCity(city);

printNames(namesForCity, format);
{% endhighlight %}

Quizás no siempre se obtiene un gran beneficio, ya que la primera expresión es bastante sencilla de leer. De nuevo reitero que el problema de no extraer por norma, es que damos lugar a seguir el patrón de embeber las expresiones, por lo que encontrar cosas como:

{% highlight java %}
printNames(
    namesDAO.fetchNamesForCity(city)
            .stream().map(n -> n.toUpperCase()).collect(toList()),
    formatsDAO.getFormatForCity(city),
    Locale.US);
{% endhighlight %}

será mucho más habitual. Por otro lado, tenemos un beneficio añadido, y es que al extraer en variables, la depuración es mucho más sencilla. En efecto, bastará con ubicar un punto de ruptura en la línea que contiene la invocación a `printNames` para poder conocer de inmediato los valores de todos sus parámetros.

### Extraer funciones

Es el nivel más importante de extracción a mi parecer. Repito aquí las dos "reglas de las funciones" según Uncle Bob:

>1. Functions should be small
>2. They should be smaller than that

Las funciones deberían [hacer una sola cosa](/2016/09/haz-una-sola-cosa/), y la manera más sencilla de conseguirlo es extraer, una y otra vez, hasta que no sea posible extraer más.

De nuevo, tenemos una herramienta perfecta en nuestros IDEs para hacer esto, llamada "extraer función", que realiza este cometido a la perfección.

El ejemplo de código que viene a continuación fue desarrollado por mí mismo hace unos tres años para un proceso de selección. Se trataba de implementar el juego del ahorcado en una aplicación de Spring MVC, y en su día me pareció que la solución era bastante correcta y legible. Pongo aquí un extracto del método de la clase `Controller` que procesaba una apuesta determinada (es decir, intentar acertar una letra):

{% highlight java %}
@RequestMapping(value="/bet", method=RequestMethod.PUT)
public @ResponseBody Object playGame(
    @CookieValue(value="hangmanGameId", required=false) String gameId,
    @RequestBody GameUserView gameUserView,
    HttpServletResponse response) {

    if(gameId == null)
        gameId = gameUserView.getGameId();

    Game game = gameRepository.getGame(gameId);

    ErrorInfo errorInfo = null;

    if(game == null) {
        response.setStatus(HttpStatus.INTERNAL_SERVER_ERROR);
        errorInfo = new ErrorInfo(
            Constants.UNEXPECTED_SERVER_ERROR);
    }
    else {
        synchronized (game) {
            if(game.getState() != gameUserView.getState()) {
                response.setStatus(HttpStatus.CONFLICT);
                errorInfo = new ErrorInfo(
                    Constants.CONCURRENT_GAMES_ERROR);
            }
            else {
                if(!game.play(gameUserView.getCharacter())) {
                    response.setStatus(HttpStatus.BAD_REQUEST);
                    errorInfo = new ErrorInfo(
                        Constants.INCORRECT_PLAY_ERROR);
                }

                if(game.isGameOver() || game.isFinished()) {
                    gameRepository.removeGame(game);
                    CookieUtils.clearCookie(
                        response, Constants.GAME_ID_COOKIE_NAME);
                }
            }
        }
    }

    if(errorInfo != null)
        return errorInfo;
    else
        return game.asUserView();
}
{% endhighlight %}

Vemos como un solo método hace un montón de cosas, y comprender la lógica en su totalidad puede llevar un tiempo. De esta forma, si alguien diferente a nosotros tiene que modificar el código, para, por ejemplo, añadir una acción adicional al finalizar el juego, antes deberá encontrar el punto en el que tal proceso se está realizando, y ser cuidadoso para no alterar otro tipo de comportamientos.

Me he liado la manta a la cabeza, y aplicando mi mantra "extract till you drop", he generado una nueva versión del código, que hace **exactamente lo mismo**:

{% highlight java %}
@RequestMapping(value="/bet", method=RequestMethod.PUT)
public @ResponseBody GameUserView playGame(
        @CookieValue(value="hangmanGameId", required=false) String gameId,
        @RequestBody GameUserView gameUserView,
        HttpServletResponse response) {

    Game game = fetchGame(gameId, gameUserView);

    synchronized (game) {
        verifyGameConsistency(game, gameUserView);
        playBet(game, gameUserView);
        verifyEndState(game, response);
    }

    return game.asUserView();
}

private Game fetchGame(
    String gameId,
    GameUserView gameUserView) {

    if(gameId == null)
        gameId = gameUserView.getGameId();

    Game game = gameRepository.getGame(gameId);

    if (game == null) {
        throw new GameNotFoundException();
    }

    return game;
}

private void verifyGameConsistency(
    Game game,
    GameUserView gameUserView) {

    if(gameIsBeingPlayedConcurrently(game, gameUserView)) {
        throw new GameIsPlayedConcurrentlyException();
    }
}

private boolean gameIsBeingPlayedConcurrently(
    Game game,
    GameUserView gameUserView) {

    return game.getState() != gameUserView.getState();
}

private void playBet(
    Game game,
    GameUserView gameUserView) {

    char characterPlayed = gameUserView.getCharacter();

    boolean valid = !game.play(characterPlayed);

    if(!valid)
        throw new InvalidBetPlayedException();
}

private void verifyEndState(
    Game game,
    HttpServletResponse response) {

    if(cantContinuePlaying(game)) {
        cleanUpGame(response, game);
    }
}

private boolean cantContinuePlaying(Game game) {
    return game.isGameOver() || game.isFinished();
}

private void cleanUpGame(
    HttpServletResponse response,
    Game game) {

    gameRepository.removeGame(game);
    CookieUtils.clearCookie(response, Constants.GAME_ID_COOKIE_NAME);
}

@ExceptionHandler(GameNotFoundException.class)
@ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
public @ResponseBody ErrorInfo gameNotFound() {
    return new ErrorInfo(Constants.UNEXPECTED_SERVER_ERROR);
}

@ExceptionHandler(GameIsPlayedConcurrentlyException.class)
@ResponseStatus(HttpStatus.CONFLICT)
public @ResponseBody ErrorInfo gamePlayedConcurrently() {
    return new ErrorInfo(Constants.CONCURRENT_GAMES_ERROR);
}

@ExceptionHandler(InvalidBetPlayedException.class)
@ResponseStatus(HttpStatus.BAD_REQUEST)
public @ResponseBody ErrorInfo invalidBetPlayed() {
    return new ErrorInfo(Constants.INCORRECT_PLAY_ERROR);
}
{% endhighlight %}

De acuerdo, ahora tenemos un número mayor de líneas de código. ¿Y qué? Las responsabilidades están perfectamente encapsuladas en pequeñas funciones que realizan una sola cosa, de forma que:

* El "happy path", donde no se da ningún error, es muy claro en el método principal
* Vemos que el proceso de apuesta se divide en cuatro pequeños pasos: extraer la instancia de `Game` del repositorio, verificar su estado / consistencia, hacer la apuesta, y verificar que el juego no ha terminado. Por último se devuelve en el cuerpo de la respuesta la vista actual del juego al usuario
* Las expresiones booleanas compuestas están contenidas en métodos que describen de forma precisa el significado de cada expresión, por lo que no es necesario intentar entenderlas cada vez
* El nivel de indentación se reduce drásticamente, y las expresiones if / else han desaparecido. En general, cuando encontramos que indentamos más de un nivel, deberemos plantearnos la necesidad de extraer comportamientos
* Los casos de error están gestionados en funciones individuales anotadas con `@ErrorHandler`. Para conseguir esto hemos tenido que añadir tres nuevas clases de tipo `Exception` a nuestro código, pero este cambio no hace otra cosa que resaltar los puntos donde se están produciendo estas situaciones "excepcionales"

Por tanto, y siguiendo con la propuesta de cambio mencionada más arriba, si queremos añadir una acción adicional al finalizar, ¿qué haríamos? En efecto, navegar en nuestras funciones: `playGame` > `verifyEndState` > `cleanUpGame`. Si utilizamos buenos nombres el proceso debería ser bastante directo, y no tendremos necesidad de comprender más lógica que la que es necesario modificar.

Los nombres son muy importantes en este punto para evitar el efecto de "el código que busco está siempre en otro lugar". Si utilizamos buenos nombres el código estará en el lugar que le corresponde, ni más ni menos. Una buena metáfora podría ser el típico cajón donde guardamos decenas de utensilios inservibles "por si acaso", y que cuando finalmente necesitamos nos lleva una vida encontrar. Si en lugar de tenerlos todos en el mismo cajón tuviéramos pequeñas cajitas con etiquetas (cables, enchufes, botones, chinchetas...) su localización sería mucho más sencilla.

Adicionalmente, si no extraemos de esta forma, caeremos en la trampa de añadir líneas de código al mismo método cuando un nuevo requisito es demandado, y puesto que no pasa nada por no extraer, nuestro método crecerá sin control hasta convertirse en algo inmanejable ([big ball of mud](https://en.wikipedia.org/wiki/Big_ball_of_mud)).

Por último, una importante ventaja de esta estrategia de diseño de funciones es la facilidad para llevar a cabo benchmarking o medición del rendimiento de un método determinado. Así, si fuera necesario calcular el tiempo que lleva limpiar el estado del juego al finalizar sólo tendríamos que añadir las anotaciones necesarias al método `cleanUpGame` (en el ejemplo utilizo anotaciones del framework [JMH](http://tutorials.jenkov.com/java-performance/jmh.html)):

{% highlight java %}
@Benchmark
private void cleanUpGame(HttpServletResponse response, Game game) {
    gameRepository.removeGame(game);
    CookieUtils.clearCookie(response, Constants.GAME_ID_COOKIE_NAME);
}
{% endhighlight %}

### Extraer clases

Es muy habitual encontrarse con funciones largas, de más de 100 líneas, donde se llevan a cabo todo tipo de acciones de lo más heterogéneo, como parsear parámetros, validarlos, procesarlos, enviarlos a algún sitio, y además manejar todos los escenarios de error posibles. Bien, dichas funciones son más bien clases camufladas, y, como desarrolladores, deberemos pensar siempre en la mejor forma de aislar responsabilidades dentro de clases bien estructuradas en lugar de dar vida a métodos monstruosos.

He mencionado anteriormente como extraer funciones es una actividad de diseño. Entre otras cosas lo es porque al extraer y extraer, surgen nuevas responsabilidades que piden a gritos ser encapsuladas dentro de una clase. En el ejemplo del ahorcado, está muy claro que sería conveniente mover todos los métodos de gestión de errores a una clase con esa única responsabilidad, por ejemplo:

{% highlight java %}
@ControllerAdvice
public class ErrorHandler {

    @ExceptionHandler(GameNotFoundException.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public @ResponseBody ErrorInfo gameNotFound() {
        return new ErrorInfo(Constants.UNEXPECTED_SERVER_ERROR);
    }

    @ExceptionHandler(GameIsPlayedConcurrentlyException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public @ResponseBody ErrorInfo gamePlayedConcurrently() {
        return new ErrorInfo(Constants.CONCURRENT_GAMES_ERROR);
    }

    @ExceptionHandler(InvalidBetPlayedException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public @ResponseBody ErrorInfo invalidBetPlayed() {
        return new ErrorInfo(Constants.INCORRECT_PLAY_ERROR);
    }
}
{% endhighlight %}

Y también sería conveniente encapsular la lógica de gestión de juego fuera de la clase controladora, pero eso lo dejo como ejercicio :)


### Extraer paquetes

Por último, tenemos que prestar atención al tamaño de nuestros `packages` o `namespaces`, que deben estar bien cohesionados, y enfocados en hacer "una sola cosa". No me extenderé más en esto, porque ya lo mencioné en [el post anterior](/2016/09/haz-una-sola-cosa/).

### Conclusión: empatía

Leí hace poco que [Kent Beck](https://es.wikipedia.org/wiki/Kent_Beck), a la pregunta de "¿Cuál es la cualidad más importante del Software?" respondió "Empatía". Seguir este mantra es la forma más fácil de alcanzar esa empatía, porque nos obligará a pensar permanentemente en el programador que viene detrás de nosotros y en como facilitarle la lectura del código.
