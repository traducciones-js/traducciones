Traducido de https://ponyfoo.com/articles/es6-iterators-in-depth

# Iteradores de ES6 en profundidad

### _Esta es una edición más de *ES6 en Profundidad*. Primera vez aquí? Bienvenido! Hasta ahora hemos cubierto desestructuramiento, plantillas de cadenas de texto, funciones flecha, el operador propagación y los parámetros rest, mejoras que van a venir a los objetos literales, el nuevo azúcar sintáctico clases por sobre prototipos, y un artículo sobre `let`, `const`, y la "Zona Temporal Muerta". La sopa del dia es: *Iteradores*_.

--------------------------------------------------------------------

Como hice en artículos previos en la serie, me gustaría mencionar que probablemente deberías poner a funcionar Babel y seguir los ejemplos con una REPL (Bucle de Leer, Evaluar, Imprimir) o la CLI (interfaz de línea de comandos) `babel-node` y un archivo.
Eso hará que sea tanto más fácil que internalices los conceptos discutidos en la serie.
Si no sos el tipo de ser humano al que le gusta _"instalar cosas en su computadora"_, puede que prefieras ir a CodePen y luego hacer click en el ícono de engranaje para JavaScript - _ellos tienen un preprocesador de Babel que hace que probar ES6 sea pan comido_.
Otra alternativa que también es bastante útil es usar la REPL online de Babel - _ésta te mostrará código ES5 compilado a la derecha de tu código de ES6 para una comparación rápida_.

--------------------------------------------------------------------

Antes de meternos de lleno, dejame que _te pida tu apoyo sin tapujos_ si estás disfrutando mi serie ES6 en profundidad.
Tus contribuciones irán a que me mantenga al día con el calendario de artículos, cuentas de servidores, alimento, y mantener a Pony Foo como una verdadera fuente de chucherías de JavaScript.

Gracias por escuchar eso, y sin más que agregar.. _me permite_?

# El protocolo iterador y el protocolo iterable

--------------------------------------------------------------------

_Hay mucha terminología nueva y mezclada acá. Por favor seguime mientras nos saco algunas de estas explicaciones de encima_.

--------------------------------------------------------------------

JavaScript obtiene dos nuevos protocolos en ES6, _Iteradores_ e _Iterables_.
En términos llanos, podés pensar en los protocolos como _convenciones_.
En la medida que sigas una determinada convención en el lenguaje, obtendrás un efecto secundario.
El protocolo _iterable_ te permite definir el comportamiento cuando los objetos en JavaScript están siendo iterados.
Debajo del capó, enterrado en el mundo de intérpretes de JavaScript y las _palabras ininteligibles_ de la especificacón del lenguaje, tenemos el método `@@iterator`. Este método subyace al protocolo _iterable_ y, en el mundo real, podés asignar a él usando algo conocido como el "bien conocido símbolo `Symbol.iterator`".

Vamos a volver a hablar sobre símbolos _mas adelante en la serie_.
Antes de perder el foco, deberías saber que el método `@@iterator` es llamado *una vez, cada vez que el objeto debe ser iterado*.
Por ejemplo, al principio de un bucle `for..of`_(al cual también vamos a volver en unos minutos)_, al `@@iterator` se le pedirá un _iterador_. El _iterador_ devuelto será utilizado para sacar valores del objeto.

Usemos el pedazo de código a continuación como una ayuda para entender los conceptos detrás de la iteración.
Lo primero que notarás es que estoy haciendo que mi objeto sea iterable asignando a la mística propiedad `@@iterator` del mismo a través de la propiedad `Symbol.iterator`.
No puedo usar el símbolo como un nombre de propiedad directamente.
En su lugar, tengo que envolverlo en corchetes, lo cual significa que es una propiedad computada que evalua a la _expresión_ `Symbol.iterator` - como puede que recuerdes de mi artículo sobre objetos literales.
El objeto retornado por el método asignado a la propiedad `[Symbol.iterator]` debe adherir al protocolo _iterador_.
El protocolo _iterador_ define cómo extraer valores de un objeto, y debemos devolver un `@@iterator` que adhiera al protocolo _iterador_.
El protocolo indica que debemos tener un objeto con un método `next`.
El método `next` no recibe argumentos y debe retornar un objeto con las siguientes propiedades:

 - `done` simboliza que la secuencia terminó cuando es `true`, y `falso` implica que aún quedan valores.
 - `value` es el ítem actual en la secuencia.
 
 En mi ejemplo, el método iterador retorna un objeto que tiene una lista finita de items la cual emite esos ítems hasta que ya no quede ninguno.
 El código a continuación es un objeto iterable en ES6.
 
```
var foo = {
  [Symbol.iterator]: () => ({
    items: ['p', 'o', 'n', 'y', 'f', 'o', 'o'],
    next: function next () {
      return {
        done: this.items.length === 0,
        value: this.items.shift()
      }
    }
  })
}
```

Para iterar efectivamente sobre el objeto, podríamos usar `for..of`.
Cómo se vería eso?
Miremos a continuación.
El método de iteración `for..of` también es nuevo en ES6, y pone punto final a la guerra sobre iterar colecciones de JavaScript y encontrar cosas al azar que no pertenecen al set de resultados que estabas esperando.

```
for (let pony of foo) {
  console.log(pony)
  // <- 'p'
  // <- 'o'
  // <- 'n'
  // <- 'y'
  // <- 'f'
  // <- 'o'
  // <- 'o'
}
```

Podés usar `for..of` para iterar sobre cualquier objeto que adhiera al protocolo _iterable_.
En ES6, esto incluye arreglos, cualquier objeto con un método `[Symbol.iterator]`, _generadores_, colecciones de nodos del DOM devueltas por `.querySelectorAll` y similares, etc.
Si querés "_convertir_" cualquier iterable en un arreglo, un par de alternativas tersas podrían ser usar el operador propagación y `Array.from`.

```
console.log([...foo])
// <- ['p', 'o', 'n', 'y', 'f', 'o', 'o']
console.log(Array.from(foo))
// <- ['p', 'o', 'n', 'y', 'f', 'o', 'o']
```

Para resumir, nuestro objeto `foo` adhiere al protocolo _iterable_ al asignar un método al `[Symbol.iterator]` - _cualquier lugar dentro de la cadena prototípica de `foo` funcionaría.
Esto significa que el objeto es _iterable_: puede ser iterado.
Dicho m{etodo devuelve un objeto que adhiere al protocolo _iterador_.
El método iterador es llamado una vez cada vez que queremos empezar a iterar el objeto, y el _iterador_ devuelto es utilizado para extraer valores de `foo`.
Para iterar iterables, podemos usar `for..of`, el operador propagación, o `Array.from`.

# Qué Significa Todo Esto?

En esencia, el atractivo de los protocolos de iteración, `for..of`, `Array.from`, y el operador propagación es que proveen métodos expresivos para iterar sin esfuerzo sobre colecciones y símil-arreglos _(como `arguments`)_.
Tener la habilidad de definir cómo cualquier objeto puede ser iterado es enorme, ya que permite que cualquier librería como *lo-dash* converja bajo un protocolo que el lenguaje entiende nativamente - _iterables_.
Esto es *gigante*.

tweet

Sólo para darte otro ejemplo, te acordás cómo siempre me quejo de que los objetos envueltos de jQuery no son verdaderos arreglos, o cómo `document.querySelectorAll` tampoco devuelve un arreglo?
Si jQuery implementase el protocolo iterador en el prototipo de sus colectiones, luego podrías hacer algo como lo siguiente:

```
for (let item of $('li')) {
  console.log(item)
  // <- el <li> envuelto en un objeto jQuery
}
```

Por qué envuelto?
Porque es más expresivo.
También podés iterar sencillamente tan profundo como necesites.

```
for (let list of $('ul')) {
  for (let item of list.find('li')) {
    console.log(item)
    // <- el <li> envuelto en un objeto jQuery
  }
}
```

Esto me recuerda un aspecto importante de los iterables y los iteradores.

# Perezosos por naturaleza

Los iteradores son _perezosos por naturaleza_.
Esto es jerga elevada para decir que la secuencia es accedida de a un ítem por vez.
Inclusive puede ser una secuencia infinita - un escenario legítimo con muchos casos de uso.
Dado que los iteradores son perezosos, que jQuery envuelva cada resultado en la secuencia con su objeto envoltorio no tendría un costo inicial alto.
En lugar de eso, un envoltorio es creado cada vez que retiramos un valor del _iterador_.


Qué aspecto tendría un iterador infinito?
El ejemplo a continuación muestra un iterador cun un rango de `1..Infinity`.
Notarás que nunca produce `done: true`, señalando que la secuencia está completa.
Intentar amoldar el objeto `foo` iterable en un arreglo ya sea usando `Array.from(foo)` o `[...foo]` haria fallar nuestro programa, puesto que la secuencia _nunca termina_.
Debemos ser muy cautelosos con este tipo de secuencias ya que pueden hacer que nuestro proceso de Node falle y se prenda fuego, o la pestaña del browser.

```
var foo = {
  [Symbol.iterator]: () => {
    var i = 0
    return { next: () => ({ value: ++i }) }
  }
}
```

La manera correcta de trabajar con un iterador semejante es con una condición de escape que prevenga que el bucle continúe indefinidamente.
El ejemplo a continuación itera por sobre nuestra secuencia infinita utilizando un bucle `for..of`, pero rompe el bucle tan pronto el valor supera `10`.

```
for (let pony of foo) {
  if (pony > 10) {
    break
  }
  console.log(pony)
}
```

El iterador no _sabe_ realmente que la secuencia es infinita.
En ese sentido, esto es similar al problema de la parada - no hay manera de conocer en código si una secuencia es infinita o no.

imagen de xkcd
Epígrafe: _El problema de la parada ilustrado por XKCD_

*Normalmente tenemos una noción muy buena* de si una secuencia es _finita_ o _infinita_, dado que nosotros construimos dichas secuencias.
Toda vez que tengamos una secuencia infinita es tarea nuestra agregar una condición de escape que asegure que nuestro programa no fallará en un intento de iterar por sobre cáda valor en la secuencia.

_Volvé mañana para una discusión sobre generadores!_

