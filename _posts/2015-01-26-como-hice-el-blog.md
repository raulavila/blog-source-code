---
layout: post
title: Cómo hice el blog
permalink: 2015/01/como-hice-el-blog
tags:
- Blog
comments: true
---

![Jekyll y GitHub](/public/pictures/jekyll-github.png)

En Internet hay un montón de alternativas para crear un blog propio. Cuando me propuse crear una web personal tuve claro desde el principio que los contenidos generados deberían ser míos. Me explico: en ocasiones, al utilizar plataformas como [blogger](http://www.blogger.com) ocurre que los posts quedan alojados en un servidor sobre el que no tienes ningún control. Y así te puedes encontrar con que nuestro amigo Google ha decidido que no estás cumpliendo las *condiciones de uso* y cerrártelo de buenas a primeras. En la misma liga juegan los blogs de [WordPress](https://es.wordpress.com/), por ejemplo.

<!--break-->

No hablo de primera mano, pero sí que me he encontrado con gente en esta situación leyendo foros y demás. Siempre es posible hacer copias de seguridad, pero en caso de catástrofe recuperar los contenidos de la forma en que se encontraban supondría un esfuerzo que no sé si estaría dispuesto a asumir (y digo esto cuando aún no he publicado la segunda entrada de mi blog :)).

Total, que las opciones se iban reduciendo. Durante varias semanas casi llegué a decidirme por contratar un hosting e instalar WordPress ([no es lo mismo Wordpress.org que Wordpress.com](http://www.creartiendavirtual.com.es/diferencias-entre-wordpress-org-y-wordpress-com-y-tu-padre-y-tu-madre-cual-te-conviene-elegir/)), pero, para ser sinceros, no me apetecía demasiado el desembolso inicial, y seguía teniendo la sensación de estar limitado por una plataforma, plataforma que es infinita en cuanto a opciones por otra parte, y permite conseguir resultados tan chulos como [esta web](http://carlosalcaniz.com/).

La opción definitiva la descubrí de la mano de [Trisha Gee](http://trishagee.github.io/), referencia de la [London Java Community](https://twitter.com/ljcjug). Descubrí que su blog está alojado en [GitHub Pages](https://pages.github.com/), una especia de servicio de hosting que da GitHub a sus usuarios, ¡de forma gratuita! La idea es que tú generas tu web como quieras y la subes a un repositorio GitHub para finalmente ser publicada en la URL [usuario].github.io. Genial.

Profundizando más, existe una plataforma llamada [Jekyll](http://jekyllrb.com/), que es utilizada por GitHub pages para generar webs a partir de contenido estático creado mediante [Markdown](http://es.wikipedia.org/wiki/Markdown). Esta plataforma corre con Ruby, lenguage del que no tengo ni idea, pero del que apenas hay que conocer nada para utilizar Jekyll. En resumen, que me decidí por Jekyll + GitHub pages, y diría que no puedo estar más contento con la decisión. Aparte de permitirme mantener una copia local de mi web puedo jugar todo lo que quiera con estilos, plantillas, etc. Otra ventaja muy interesante es que al subirse como contenidos estáticos, sin bases de datos o plugins respaldando, la seguridad frente a ataques es bastante alta.

Reproduciré a continuación, a modo de referencia, los diferentes pasos que llevé a cabo para poner en marcha esta web (y así queda registrado por si tengo que volver a hacerlo algún día):

1. Dar de alta una cuenta en [GitHub](https://github.com/). El nombre de usuario será el que dará nombre a la web una vez publicada (en mi caso raulavila.github.io).
2. Crear el repositorio que alojará el blog, [según se explica aquí](https://pages.github.com/).
3. [Instalar Jekyll](http://www.webhostwhat.com/guide-how-to-host-jekyll-blog-on-github-using-a-mac/). Si no tienes un Mac sería necesario [instalar Ruby primero](http://jekyllrb.com/docs/installation/).
4. Buscar una plantilla de referencia para, a partir de ahí, dar forma al blog. Esta opción es más sencilla que crear la página desde cero, aunque también sería una posibilidad, claro. La plantilla que elegí fue [Poole](http://getpoole.com/), es muy minimalista, e incluye varios archivos útiles como el Feed, 404, config, etc.
5. Añadir la plantilla al repositorio, junto con un fichero .gitignore (para que no se incluyan ciertos archivos en los commits). Si lo subimos a GitHub podremos ver la web publicada por primera vez pasados unos 30 minutos (en posteriores commits los cambios se actualizan automáticamente).
6. Configurar correctamente las variables del fichero _config.yml
7. Tunear los ficheros CSS a mi gusto, así como el layout de las plantillas.
8. Cambiar el idioma de los textos de las plantillas a Castellano. Algún día escribiré un post explicando por qué decidí escribir en Castellano y no en inglés.
9. Añadir una página de [Archivos](/archivos), que recorre todos los posts publicados y los lista por fecha. Para ello utilizo el lenguaje de plantillas [Liquid](http://jekyllrb.com/docs/templates/). También añadí una página [Sobre mí](/sobre-mi), pero tampoco tiene demasiado misterio.
10. Jekyll genera de forma automática los permalinks, pero no me gustaba demasiado la forma en que lo hacía. Existe la opción de asignar el permalink que deseemos mediante la variable permalink [en la cabecera de los posts](http://jekyllrb.com/docs/frontmatter/).
11. Añadir botones de Twitter ([Follow Me](https://dev.twitter.com/web/follow-button) en "Sobre mí", y [Tweet](https://dev.twitter.com/web/tweet-button) en la cabecera de los posts). Es muy sencillo siguiendo las indicaciones de los enlaces.
12. Soporte para añadir tags en los posts, y crear páginas para cada tag específico. Seguí las indicaciones de [este blog](http://charliepark.org/tags-in-jekyll/), no obstante me trajo varios quebraderos de cabeza por motivos que explicaré en el anexo. La verdad es que el soporte para tags es uno de los puntos flacos de Jekyll+GitHub Pages.
13. Truncar posts en la página de inicio, según se explica [aquí](http://mikeygee.com/blog/truncate.html).
14. Para publicar bajo un dominio propio, en lugar de *.github.io, compré el nombre en [NameCheap](https://www.namecheap.com/). Es una opción bastante barata, e incluye el servicio [WhoIs Guard](https://www.namecheap.com/domains/whois.aspx).
15. Configurar las DNS de GitHub pages en NameCheap. [Este blog](http://davidensinger.com/2013/03/setting-the-dns-for-github-pages-on-namecheap/) lo explica perfectamente.
16. Añadir comentarios [Disqus](https://disqus.com/). Tras darme de alta en la web, las indicaciones fueron muy claras y funcionó sin ningún problema.
17. Añadir [Google Analytics](http://www.google.com/analytics/), para así poder saber si me visita alguien de vez en cuando. 
16. Tunear detallitos sin importancia aquí y allá.

Creo que no me dejo nada. Mención especial para [este post](http://joshualande.com/jekyll-github-pages-poole/), que me sirvió de referencia para muchos de los puntos. De hecho, utilizo la misma plantilla.

###Anexo: GitHub y los plugins

Me encontré con un problema inesperado a la hora de añadir el plugin de Ruby que creaba automáticamente las páginas para los tags. Al subirlo a GitHub pages descubrí que dichas páginas ¡no se generaban! Tras investigar un poco, lo que ocurre es que GitHub pages, al generar los sites a partir de las plantillas de Jekyll no ejecuta, por seguridad, los plugins adicionales que pueda haber en la carpeta **_plugins**, y que sí se ejecutan a nivel local.

La solución a este dilema no fue nada sencilla. Pensé en descartar el uso de plugins para la web, pero me parecía limitarme demasiado. No solo me llevaba a crear una solución más chapucera para los tags, a medio plazo no podría añadir cualquier otra cosa si lo consideraba necesario.

La salida más razonable está explicada en [este post](http://www.sitepoint.com/jekyll-plugins-github/). En lugar de dejar a GitHub la tarea de generar la web la genero yo localmente y la subo al repositorio de GitHub pages. El código fuente está subido en un repositorio diferente. Aunque existe una solución que juega con las ramas dentro del mismo repositorio, me pareció más adecuado así ([Separation of concerns](http://en.wikipedia.org/wiki/Separation_of_concerns) y tal).

Por tanto, finalmente mi blog está dividido en dos repositorios:

* [Uno con el código fuente](https://github.com/raulavila/blog-source-code)
* [Y otro con el site generado en local con Jekyll](https://github.com/raulavila/raulavila.github.io)

Es increíble la cantidad de trabajo que puede llevar montar una página tan sencillita, pero estoy bastante orgulloso del resultado.


