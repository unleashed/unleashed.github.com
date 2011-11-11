---
layout: post
title: "Cuando programar es mejor que trabajar"
date: 2011-11-08 23:57
comments: true
categories: [logica, ruby]
---
Hay veces en las que me sorprendo de lo útil que es saber programar en lenguajes que te permiten hacer un prototipado rápido de una idea. Y sorprender es poco si además se trata de lenguajes generalistas.

Ayer, en mi habitual (por acuciante falta de tiempo) _rush_ para entregar a tiempo los deberes de la [#aiclass](http://www.ai-class.com), me encontré con algunos viejos temas que vi en la universidad. Tenía que hacer unos ejercicios sobre lógica proposicional, y determinar si las proposiciones eran [tautologías](http://es.wikipedia.org/wiki/Tautolog%C3%ADa) (siempre ciertas), contradicciones (siempre falsas), o su valor dependía del valor que tomaran sus predicados por separado. Aquí podéis ver la descripción del problema de ayer en particular:

<iframe width="560" height="315" src="http://www.youtube.com/embed/WP_97aspqrc" frameborder="0" allowfullscreen></iframe>
<!--more-->
Una vez se estudian los distintos operadores se pueden construir tablas de verdad para resolver este problema, pero lo cierto es que el papel es demasiado caro como para ir llenándolo de tablas de verdad... Añadiendo a esto que no recordaba del todo bien el tema y que no me había refrescado la memoria con los vídeos disponibles, se me ocurrió hacer... bueno, alguna trampa. Sí recordé -tras muchos años de haberla estudiado- una relación básica de lógica proposicional que se me quedó grabada en el cerebro, y que a la postre fue muy útil para resolver los problemas con mi "_sistema_" sin tener que exprimirme mucho la cabeza: si una hipótesis es falsa, se puede extraer cualquier conclusión:

> p -> q <=> ¬p v q

Esta relación indica que tanto **p -> q** como **¬p v q** son equivalentes, comparten la misma [tabla de verdad](http://es.wikipedia.org/wiki/Tabla_de_verdad).

Con esto puedo expresar el operador -> (en el vídeo se usa "=>" en lugar de "->", y "<=>" en lugar de "<->") en sencillas operaciones **not**, **and** y **or** de cualquier lenguaje de programación, y así escribir fácilmente las proposiciones que necesito comprobar, sin complicarme la vida. Y con un lenguaje de programación potente no tengo que empezar a desarrollar las proposiciones ni llevar los cálculos de una tabla de verdad. En mi caso, como buen rubyist, hago las pruebas en el entorno "irb" de Ruby:

{% codeblock lang:ruby %}
def tabla_de_verdad(&block)
  [true, false].repeated_permutation(block.arity).map(&block)
end
{% endcodeblock %}

La simplicidad y elegancia de esa línea de código se puede admirar cuando se explica que con solamente eso se puede obtener la tabla de verdad para cualquier proposición, independientemente del número de predicados que contenga, de forma sencillísima. Más se sorprende uno si se compara cuánto tiempo y esfuerzo llevaría hacer algo similar en un lenguaje como Java o C++. Especial atención merece el hecho de que este método crea permutaciones con repetición de valores booleanos para el número de variables (predicados) que espera el bloque de código que se le pasa sin que tengamos que preocuparnos de especificar nada. Así nuestra única preocupación será construir la proposición lógica, ya que los diferentes casos de pruebas necesarios para toda la tabla de verdad son generados -y sus resultados recogidos- de forma automática.

Ése es el tipo de características que hacen de este tipo de lenguajes herramientas ideales para estas tareas.

Para poder comprobar las tablas de verdad, recordé que **p <-> q** en realidad requería comprobar la relación **p -> q** y luego la relación en sentido inverso, **q -> p**. Cuando el resultado es una tautología, se suele usar el símbolo "<=>" (si, y solo si), y cuando solamente uno de los sentidos es una tautología se suele usar el símbolo "=>" (implica).

{% codeblock lang:ruby %}
def si_y_solo_si?(p, q)
  # p <-> q
  # (p -> q) ^ (q -> p)
  # (¬p v q) ^ (¬q v p)
  (!p or q) and (!q or p)
end
{% endcodeblock %}

La expresión **p <-> q** comparte la misma tabla de verdad que **p = q**, y por lo tanto es equivalente (es, de hecho, la igualdad lógica, ya que "**p** implica **q** y **q** implica **p**"). No es necesario definir el código anterior si disponemos de un operador de igualdad lógica, como es el caso en Ruby con "==".

De esta forma pude comprobar rápidamente las tablas de verdad del ejercicio y responder sin calcular nada, tan sólo con escribir las proposiciones con sus operadores en Ruby, sustituyendo "<=>" por el operador de igualdad y "->" por la expresión indicada anteriormente:

{% codeblock (smoke -> fire) <-> (smoke v ¬fire) lang:ruby %}
tabla_de_verdad do |smoke, fire|
  (!smoke or fire) == (smoke or !fire)
end
{% endcodeblock %}

{% codeblock (smoke -> fire) <-> (¬smoke -> ¬fire) lang:ruby %}
tabla_de_verdad do |smoke, fire|
  (!smoke or fire) == (smoke or !fire)
end
{% endcodeblock %}

{% codeblock (smoke -> fire) <-> (¬fire -> ¬smoke) lang:ruby %}
tabla_de_verdad do |smoke, fire|
  (!smoke or fire) == (fire or !smoke)
end
{% endcodeblock %}

{% codeblock big v dumb v (big -> dumb) lang:ruby %}
tabla_de_verdad do |big, dumb|
  big or dumb or (!big or dumb)
end
{% endcodeblock %}

{% codeblock big ^ dumb <-> ¬(¬big v ¬dumb) lang:ruby %}
tabla_de_verdad do |big, dumb|
  (big and dumb) == !(!big or !dumb)
end
{% endcodeblock %}

Se puede comprobar que usando el método "si_y_solo_si?" se obtienen los mismos resultados.

Lo importante de todo esto no es sólo el hecho de haberme ahorrado tener que andar con tablas de verdad y cálculos booleanos, cuya importancia es relativa, sino que en muy poco tiempo, debido a conocer un lenguaje que es apto para el prototipado rápido, dispongo de la capacidad de evaluar la tabla de verdad de cualquier proposición lógica, por complicada que sea  con un número arbitrario de predicados.

Nótese que se podría mejorar fácilmente el código para asociar los valores de las variables a su resultado en la tabla y mostrar valores y resultado, como en las tablas de verdad sobre papel.
