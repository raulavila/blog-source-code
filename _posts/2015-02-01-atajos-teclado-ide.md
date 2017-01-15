---
layout: post
title: Atajos de teclado en IDE's
description: Atajos de teclado en Eclipse e IntelliJ IDEA
permalink: 2015/02/atajos-teclado-ide/
tags:
- IDE
- desarrollo
- productividad
comments: true
---

En mi [anterior post](/2015/01/entornos-integrados-desarrollo) hablé de IDE's y las diferentes alternativas que hay en el mercado. Mi intención inicial con ese post era hablar de atajos de teclado, pero me extendí demasiado con la introducción y preferí terminar ese hilo para hablar de ello más adelante.

En mi opinión, ningún desarrollador profesional puede proclamarse como tal si no domina al menos una gran parte de los atajos de teclado que detallaré a continuación. El desarrollo de software debe estar centrado en producir una buena arquitectura y un código mantenible y modular. Un código con estas características no es generado a la primera, requiriendo de un proceso sucesivo de depuración y refactorización. Las herramientas ofrecidas por los IDE's facilitan muchísimo esta labor, y me atrevería a decir que no conocerlas dificulta el proceso a seguir para llevar a cabo las acciones necesarias que nos llevarán a cumplir con el objetivo de entregar un código [SOLID](http://es.wikipedia.org/wiki/SOLID_%28object-oriented_design%29).

<!--break-->

Bien es verdad que todas las acciones están disponibles mediante los menús, pero usarlas de esta forma te hace no llegar a interiorizarlas jamás. Es el uso continuado mediante los atajos de teclado (o shorcuts) lo que realmente consigue que sea totalmente natural pulsar Alt + Shift + M para [extraer un método](http://refactoring.com/catalog/extractMethod.html), por ejemplo.

Como ya comenté, actualmente me he pasado a [IntelliJ IDEA](https://www.jetbrains.com/idea/), pero durante muchos años utilicé [Eclipse](https://eclipse.org/). Además, nunca se sabe si tendré que volver a Eclipse en algún momento, así que personalmente, consideré mucho más fácil utilizar en IntelliJ el Keymap de Eclipse, con alguna pequeña modificación. Recientemente me he pasado a Mac en casa, y eso añade otra capa de complejidad al asunto, porque los teclados de Mac tienen la tecla Cmd, que no existe en los teclados Windows. La mayoría de las veces, los shorcuts que en Windows son Ctrl + Algo, en Mac son Cmd + Algo, pero no siempre, porque Mac sigue teniendo la tecla Ctrl. Un lío, vaya.

En las siguiente tablas detallo todos los atajos que considero son imprescindibles en el día a día. Puede que se pueda vivir sin un 10-20% de ellos, pero yo no sería capaz de programar sin utilizar la gran mayoría. Separaré las tablas por categorías, y detallaré el atajo en Windows y en Mac. Repito que este Keymap es prácticamente común para Eclipse e IntelliJ, aunque alguna accción no está implementada en Eclipse. En este último caso añadiré una nota en la columna correspondiente. Si hay varios shortcuts, los separaré por comas, y en ocasiones he modificado el shortcut por defecto o añadido uno (no siempre existe shortcut por defecto). La intención del post es dar a conocer las acciones disponibles, y ya es cuestión de cada uno personalizarlo a su medida.

Vamos a ello.

### Opciones de navegación

<table class="font-small">
	<thead>
		<tr>
			<th>
				Acción
			</th>
			<th>
				Windows
			</th>
			<th>
				OS X
			</th>
			<th>
				Notas
			</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				Navegar entre ficheros
			</td>
			<td>
				Ctrl + Tab
			</td>
			<td>
				Ctrl + Tab
			</td>
			<td>
				Eclipse: modificada sobre el default
			</td>
		</tr>
		<tr>
			<td>
				Ir a la declaración seleccionada (Clase o método)
			</td>
			<td>
				Ctrl + Click, F3
			</td>
			<td>
				Ctrl + Click, F3
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Cerrar fichero abierto
			</td>
			<td>
				Ctrl + W
			</td>
			<td>
				Cmd + W
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Cerrar todos los fichero abiertos
			</td>
			<td>
				Ctrl + Shift + W
			</td>
			<td>
				Cmd + Shift + W
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Ver ficheros abiertos recientemente
			</td>
			<td>
				Ctrl + E
			</td>
			<td>
				Cmd + E
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Ver ficheros editados recientemente
			</td>
			<td>
				Ctrl + Shift + E
			</td>
			<td>
				Cmd + Shift + E
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Ir al último punto editado
			</td>
			<td>
				Ctrl + Q
			</td>
			<td>
				Ctrl + Q
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Ir al método siguiente/anterior
			</td>
			<td>
				Ctrl + Shift + Down/Up
			</td>
			<td>
				Ctrl + Alt + Down/Up
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Mostrar usos (llámadas a un método, por ejemplo)
			</td>
			<td>
				Ctrl + G
			</td>
			<td>
				Cmd + G
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Abrir definición en popup
			</td>
			<td>
				Ctrl + Shift + I
			</td>
			<td>
				Alt + Space
			</td>
			<td>
				No existe en Eclipse
			</td>
		</tr>
		<tr>
			<td>
				Navegar en la historia del cursor
			</td>
			<td>
				Alt + Left/Right
			</td>
			<td>
				Ctrl + Alt + Left/Right
			</td>
			<td>
				-
			</td>
		</tr>
		<tr>
			<td>
				Ir al inicio/final del fichero
			</td>
			<td>
				Ctrl + Home/End
			</td>
			<td>
				Cmd + Up/Down
			</td>
			<td>
				-
			</td>
		</tr>
		<tr>
			<td>
				Mover cursor al final/inicio de bloque ({})
			</td>
			<td>
				Ctrl + Shift + P
			</td>
			<td>
				Ctrl + Shift + P
			</td>
			<td>
				-
			</td>
		</tr>
	</tbody>
</table>

Me dejo algún atajo como Navegar adelante/atrás, a modo explorador. La verdad es que en este caso sigo utilizando las flechas de la barra de herramientas, y no pienso reflejar comandos que no utilizo para darme el pisto :). Si en algún momento los añado a mi toolbox diario lo actualizaría aquí.

### Asistencia en generación de código

<table class="font-small">
	<thead>
		<tr>
			<th>
				Acción
			</th>
			<th>
				Windows
			</th>
			<th>
				OS X
			</th>
			<th>
				Notas
			</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				Copiar
			</td>
			<td>
				Ctrl + C
			</td>
			<td>
				Cmd + C
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Cortar
			</td>
			<td>
				Ctrl + X
			</td>
			<td>
				Cmd + X
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Pegar
			</td>
			<td>
				Ctrl + V
			</td>
			<td>
				Cmd + V
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Pegar de la historia
			</td>
			<td>
				Ctrl + Shift + V
			</td>
			<td>
				Cmd + Shift + V
			</td>
			<td>
				No existe en Eclipse
			</td>
		</tr>
		<tr>
			<td>
				Deshacer
			</td>
			<td>
				Ctrl + Z
			</td>
			<td>
				Cmd + Z
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Rehacer
			</td>
			<td>
				Ctrl + Y
			</td>
			<td>
				Cmd + Shift + Z
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Duplicar línea
			</td>
			<td>
				Ctrl + Alt + Down
			</td>
			<td>
				Cmd + Alt + Down
			</td>
			<td>
				Eclipse: no hay shortcut por defecto
			</td>
		</tr>
		<tr>
			<td>
				Borrar línea
			</td>
			<td>
				Ctrl + D
			</td>
			<td>
				Cmd + D
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Completar statement / bloque
			</td>
			<td>
				Ctrl + Shift + Enter
			</td>
			<td>
				Cmd + Shift + Enter
			</td>
			<td>
				No existe en Eclipse
			</td>
		</tr>
		<tr>
			<td>
				Quick Fix
			</td>
			<td>
				Ctrl + 1, Alt + Enter
			</td>
			<td>
				Cmd + 1, Cmd + Enter
			</td>
			<td>
				-
			</td>
		</tr>
		<tr>
			<td>
				Sugerencias (autocompletado)
			</td>
			<td>
				Ctrl + Space
			</td>
			<td>
				Ctrl + Space
			</td>
			<td>
				-
			</td>
		</tr>
		<tr>
			<td>
				Sugerencias incluyendo static
			</td>
			<td>
				Ctrl + Alt + Space
			</td>
			<td>
				Ctrl + Alt + Space
			</td>
			<td>
				-
			</td>
		</tr>
		<tr>
			<td>
				Generar código
			</td>
			<td>
				Alt + Insert
			</td>
			<td>
				Ctrl + Enter
			</td>
			<td>
				-
			</td>
		</tr>
		<tr>
			<td>
				Rodear con (try/catch, if, etc)
			</td>
			<td>
				Alt + Shift + Z
			</td>
			<td>
				Cmd + Shift + Z
			</td>
			<td>
				-
			</td>
		</tr>
	</tbody>
</table>

Las opciones reflejadas en esta tabla son, sin duda, las que mayor repercusión tienen en el incremento de la productividad. Veamos con detalle las más complejas:

* "Pegar de la historia", en IntelliJ, guarda un histórico del portapapeles, y permite seleccionar el fragmento a pegar de una lista. Es bastante interesante.
* Completar statement / bloque: una de las opciones más potentes de IntelliJ de las que no existen en Eclipse. Cierra y formatea la línea o bloque en curso, añadiendo los paréntesis o llaves necesarios para balancear correctamente la instrucción, así como el punto y coma si es un statement de una sola línea.
* Quick fix: un clásico, ofrece opciones para solucionar problemas, por ejemplo: implementar métodos ausentes (si estamos implementado una interfaz y faltan métodos), convertir un método a static si no lo es y lo estamos referenciando como tal, etc.
* Generar código: por ejemplo, getters y setters, constructores con ciertos parametros, etc. Aunque existe en Eclipse, el popup de generación de código que aparece en IntelliJ no existe como tal:

![Generate Intellij](/public/pictures/intellij_autocomplete.png)


### Opciones de búsqueda

<table class="font-small">
	<thead>
		<tr>
			<th>
				Acción
			</th>
			<th>
				Windows
			</th>
			<th>
				OS X
			</th>
			<th>
				Notas
			</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				Buscar en fichero
			</td>
			<td>
				Ctrl + F
			</td>
			<td>
				Cmd + F
			</td>
			<td>
				IntelliJ: no hay shortcut por defecto (increíblemente)
			</td>
		</tr>
		<tr>
			<td>
				Reemplazar en fichero
			</td>
			<td>
				Ctrl + R
			</td>
			<td>
				Cmd + R
			</td>
			<td>
				IntelliJ: no hay shortcut por defecto (increíblemente)
			</td>
		</tr>
		<tr>
			<td>
				Buscar fichero en proyecto
			</td>
			<td>
				Ctrl + Shift + R
			</td>
			<td>
				Cmd + Shift + R
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Buscar clase en proyecto (y dependencias)
			</td>
			<td>
				Ctrl + Shift + T
			</td>
			<td>
				Cmd + Shift + T
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Buscar símbolo
			</td>
			<td>
				Ctrl + Shift + S
			</td>
			<td>
				Cmd + Shift + S
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Buscar implementación (de interface, clase abstracta...)
			</td>
			<td>
				Ctrl + T
			</td>
			<td>
				Cmd + T
			</td>
			<td>
				IntelliJ: no hay shortcut por defecto
			</td>
		</tr>
		<tr>
			<td>
				Buscar en todas partes
			</td>
			<td>
				Double Shift
			</td>
			<td>
				Double Shift
			</td>
			<td>
				-
			</td>
		</tr>
	</tbody>
</table>

### Refactoring

<table class="font-small">
	<thead>
		<tr>
			<th>
				Acción
			</th>
			<th>
				Windows
			</th>
			<th>
				OS X
			</th>
			<th>
				Notas
			</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				Renombrar
			</td>
			<td>
				Alt + Shift + R
			</td>
			<td>
				Alt + Shift + R
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Extraer método
			</td>
			<td>
				Alt + Shift + M
			</td>
			<td>
				Alt + Shift + M, Cmd + Alt + M
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Extraer variable
			</td>
			<td>
				Alt + Shift + L
			</td>
			<td>
				Alt + Shift + L, Cmd + Alt + L
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Extraer constante
			</td>
			<td>
				Ctrl + Alt + C
			</td>
			<td>
				Cmd + Alt + K
			</td>
			<td>
				No hay shortcut por defecto
			</td>
		</tr>
		<tr>
			<td>
				Extraer atributo
			</td>
			<td>
				Ctrl + Alt + F
			</td>
			<td>
				Cmd + Alt + F
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Cambiar declaración del método
			</td>
			<td>
				Alt + Shift + C
			</td>
			<td>
				Cmd + Alt + C
			</td>
			<td>
			-
			</td>
		</tr>
	</tbody>
</table>

### Formateo y organización de código

<table class="font-small">
	<thead>
		<tr>
			<th>
				Acción
			</th>
			<th>
				Windows
			</th>
			<th>
				OS X
			</th>
			<th>
				Notas
			</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				Organizar imports
			</td>
			<td>
				Ctrl + Shift + O
			</td>
			<td>
				Cmd + Shift + O
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Mover línea/método arriba/abajo
			</td>
			<td>
				Alt + Up/Down
			</td>
			<td>
				Alt + Up/Down
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Formatear código
			</td>
			<td>
				Ctrl + Shift + F
			</td>
			<td>
				Cmd + Shift + F
			</td>
			<td>
			-
			</td>
		</tr>
	</tbody>
</table>

Evidentemente, la opción "Formatear código" lo hará según lo tengamos configurado en las preferencias => Code Style. En proyectos corporativos es muy importante tratar de ser consistente en el estilo de código ejecutado por los desarrolladores. Una plantilla muy buena para esto es [la de Google](https://google-styleguide.googlecode.com/svn/trunk/javaguide.html). De este tema hablaré en un post futuro, seguramente.

### Acciones genéricas

<table class="font-small">
	<thead>
		<tr>
			<th>
				Acción
			</th>
			<th>
				Windows
			</th>
			<th>
				OS X
			</th>
			<th>
				Notas
			</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>
				Seleccionar todo
			</td>
			<td>
				Ctrl + A
			</td>
			<td>
				Cmd + A
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Insertar nueva línea después de la actual
			</td>
			<td>
				Shift + Enter
			</td>
			<td>
				Shift + Enter
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Comentar línea
			</td>
			<td>
				Ctrl + /
			</td>
			<td>
				Cmd + /
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Comentar bloque
			</td>
			<td>
				Ctrl + Shift + /
			</td>
			<td>
				Cmd + Shift + /
			</td>
			<td>
			-
			</td>
		</tr>
		<tr>
			<td>
				Encontrar Acción
			</td>
			<td>
				Ctrl + Shift + A
			</td>
			<td>
				Ctrl + Shift + A
			</td>
			<td>
				No es el shortcut por defecto de IntelliJ
			</td>
		</tr>
		<tr>
			<td>
				Ejecutar clase / tests
			</td>
			<td>
				Ctrl + Shift + F10
			</td>
			<td>
				Ctrl + Shift + F10 (personalizado), Ctrl + Shift + R (default)
			</td>
			<td>
				Eclipse: estas no son las opciones por defecto
			</td>
		</tr>
		<tr>
			<td>
				Debug clase / tests
			</td>
			<td>
				Ctrl + Shift + F11
			</td>
			<td>
				Ctrl + Shift + F11
			</td>
			<td>
				-
			</td>
		</tr>
		<tr>
			<td>
				Seleccionar ejecución
			</td>
			<td>
				Ctrl + Shift + F8
			</td>
			<td>
				Ctrl + Shift + F8
			</td>
			<td>
				-
			</td>
		</tr>
		<tr>
			<td>
				Seleccionar ejecución (debug)
			</td>
			<td>
				Ctrl + Shift + F9
			</td>
			<td>
				Ctrl + Shift + F9
			</td>
			<td>
				-
			</td>
		</tr>
		<tr>
			<td>
				Extender / disminuir selección
			</td>
			<td>
				Alt + Shift + Up / Down
			</td>
			<td>
				Alt + Shift + Up / Down
			</td>
			<td>
				No es el shortcut por defecto en OS X
			</td>
		</tr>
	</tbody>
</table>

La última opción de la tabla existe sólo en IntelliJ, y es muy interesante. Permite, desde la posición actual del cursor, aumentar el texto seleccionado por bloques compilables. Probadla, porque seguro que la introducís en vuestro día a día.

### Conclusión

Una buena forma de aprender estos atajos de teclado es introducirlos de manera progresiva en la dinámica de trabajo. No es posible dominar más de cuarenta shortcuts en un par de días, pero aprender tres o cuatro por semana es perfectamente posible, y seguramente ya conozcáis muchos de ellos. De hecho me daría por contento si gracias a este post os he dado a conocer dos o tres atajos que os resulten útiles de ahora en adelante.

Personalmente, mi intención con este artículo ha sido más la de recolectar de forma ordenada los diferentes shortcuts que utilizo para que me sirva de recordatorio si algún día tengo que configurar de cero el Keymap en una nueva instalación.
