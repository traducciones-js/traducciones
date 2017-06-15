Traducido de https://ponyfoo.com/articles/es6-destructuring-in-depth

# Desestructuración de ES6 en profundidad

### _**Mencioné brevemente algunas características de ES6 (y cómo comenzar con Babel) en las series de artículos de React que estuve escribiendo, y ahora quiero enfocarme en las características mismas del lenguaje. Leí una tonelada de cosas sobre ES6 y ES7 y es hora de que empecemos a debatir características de ES6 y ES7 aquí en Pony Foo.**_

Este artículo advierte **sobre exaltarse demasiado** con las características del lenguaje ES6. Luego comenzaremos la serie con la desesestructuración de ES6, y cuándo es más útil, así también como sus trampas y algunas advertencias.
 
## Un consejo
 
Cuando estés en duda, probablemente debas volver por defecto a ES5 o sintaxis anteriores en vez de adoptar ES6 sólo porque puedas. Con esto no quiero decir que usar sintaxis ES6 sea una mala idea - al contrario, como verán ¡estoy escribiendo un artículo de ES6! Mi preocupación es que cuando adoptemos las características de ES6, lo hagamos **sólo si mejoran en absoluto la calidad de nuestro código** y no sólo por el _"factor cool"_ - lo que sea que signifique eso.
 
El enfoque que estuve tomando hasta ahora es escribir cosas en puro ES5 y luego agregar un poco de azúcar ES6 encima, donde genuinamente mejoraría mi código. Supongo que con el tiempo, podré identificar más rápidamente contextos donde una característica de ES6 valga la pena usarse sobre ES5. Pero cuando empieces, sería una buena idea -no- entusiasmarse demasiado pronto. En vez de eso, analizar con cuidado que encajaría mejor en tu código y **ser consciente al adoptar ES6.**
 
De esta forma, **aprenderás a usar** las nuevas características a tu favor en vez de solo aprender la sintaxis.
 
¡Ahora pasemos a lo cool!
 
 
# Desestructuración
 
Esta es, fácilmente, una de las características que más estuve usando. También es una de las más simples. Une propiedades a tantas variables como necesitas y funciona con colecciones (arrays) y objetos.

```javascript 
var foo = { bar: 'pony', baz: 3 }
var {bar, baz} = foo
console.log(bar)
// <- 'pony'
console.log(baz)
// <- 3
``` 
 
Hace que sea muy rápido quitar una propiedad específica de un objeto. También puedes corresponder propiedades con alias.
```javascript 
var foo = { bar: 'pony', baz: 3 }
var {bar: a, baz: b} = foo
console.log(a)
``` 
También puedes obtener propiedades en otros niveles tanto como quieras, y también podrías darle un alias a esos enlaces profundos. 
```javascript 
var foo = { bar: { deep: 'pony', dangerouslySetInnerHTML: 'lol' } }
var {bar: { deep, dangerouslySetInnerHTML: sure }} = foo
console.log(deep)
// <- 'pony'
console.log(sure)
// <- 'lol'
// <- 'pony'
console.log(b)
// <- 3
``` 
Por defecto, propiedades no encontradas serán `undefined`, al igual que cuando accedes a propiedades de un objeto con el punto o la llave.
```javascript 
var {foo} = {bar: 'baz'}
console.log(foo)
// <- undefined
``` 
Si intentas acceder a una propiedad anidada profundamente de una clase padre que no existe, entonces obtendrás una excepción.
```javascript 
var {foo:{bar}} = {baz: 'ouch'}
// <- Exception
``` 
Eso tiene mucho sentido si piensas que desestructurar es como el azúcar del ES5, como en el código debajo.
```javascript 
var _temp = { baz: 'ouch' }
var bar = _temp.foo.bar
// <- Exception
``` 
Una buena propiedad de desestructurar es que permite que intercambies variables sin la necesidad de la infame variable `aux`.
```javascript 
function es5 () {
  var left = 10
  var right = 20
  var aux
  if (right > left) {
    aux = right
    right = left
    left = aux
  }
}
```
```javascript
function es6 () {
  var left = 10
  var right = 20
  if (right > left) {
    [left, right] = [right, left]
  }
}
``` 
Otro aspecto cómodo de desestructurar es la habilidad de obtener claves cuando usas [nombres de propiedades computadas](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names).
```javascript 
var key = 'such_dynamic'
var { [key]: foo } = { such_dynamic: 'bar' }
console.log(foo)
// <- 'bar'
``` 
En ES5, desde tu lado, llevaría una declaración extra y una asignación de variable.
```javascript 
var key = 'such_dynamic'
var baz = { such_dynamic: 'bar' }
var foo = baz[key]
console.log(foo)
``` 
También puedes definir valores por defecto, para el caso donde la propiedad extraída evalúa a `undefined`.
```javascript 
var {foo=3} = { foo: 2 }
console.log(foo)
// <- 2
var {foo=3} = { foo: undefined }
console.log(foo)
// <- 3
var {foo=3} = { bar: 2 }
console.log(foo)
// <- 3
```

Desestructurar funciona también para las colecciones _(arrays)_, como mencioné antes. Nota como ahora estoy **usando corchetes** del lado de la desestructuración de la declaración.
```javascript  
var [a] = [10]
console.log(a)
// <- 10
```
Aquí, de nuevo, podemos usar los valores por defecto y seguir las mismas reglas.
```javascript  
var [a] = []
console.log(a)
// <- undefined
var [b=10] = [undefined]
console.log(b)
// <- 10
var [c=10] = []
console.log(c)
// <- 10
``` 
Cuando se trata de colecciones _(arrays)_, puedes saltear los elementos que no te interesan.
```javascript  
var [,,a,b] = [1,2,3,4,5]
console.log(a)
// <- 3
console.log(b)
// <- 4
```
También puedes usar desestructuración en la lista de parámetros en una `function`.

```javascript  
function greet ({ age, name:greeting='she' }) {
  console.log(`${greeting} is ${age} years old.`)
}
greet({ name: 'nico', age: 27 })
// <- 'nico is 27 years old'
greet({ age: 24 })
// <- 'she is 24 years old'
```  
 
En términos generales, así es **cómo puedes** usar desestructuración. Pero ¿**para qué** es buena la desestructuración?
 
# Casos de uso de desustructuración
 
Existen muchas situaciones donde desestructurar puede ser útil. Aquí están alguna de las más comunes. Cuando sea que tengas un método que devuelve un objeto, desestructurar hace que sea más conciso de interactuar con él.
```javascript  
function getCoords () {
  return {
    x: 10,
    y: 22
  }
}
var {x, y} = getCoords()
console.log(x)
// <- 10
console.log(y)
// <- 22
``` 
Un caso similar de uso pero en la posición opuesta, es poder definir opciones por defecto cuando tienes un método con muchas opciones que necesitan valores por defecto.  Particularmente, es interesante como una alternativa para nombrar parámetros en otros lenguajes como Python y C#.
```javascript  
function random ({ min=1, max=300 }) {
  return Math.floor(Math.random() * (max - min)) + min
}
console.log(random({}))
// <- 174
console.log(random({max: 24}))
// <- 18
```
Si querías hacer que las opciones del objeto sean **completamente opcionales**, podrías cambiar la sintaxis a lo siguiente:
```javascript  
function random ({ min=1, max=300 } = {}) {
  return Math.floor(Math.random() * (max - min)) + min
}
console.log(random())
// <- 133
``` 
Algo que encajaría perfecto para desestructurar son las cosas como las expresiones regulares, en donde te encantaría nombrar propiedades sin tener que recurrir a números índices. Aquí hay un ejemplo analizando con un `RegExp` aleatorio que [encontré en StackOverflow](https://stackoverflow.com/questions/27745/getting-parts-of-a-url-regex/27755#27755).
```javascript  
function getUrlParts (url) {
  var magic = /^(https?):\/\/(ponyfoo\.com)(\/articles\/([a-z0-9-]+))$/
  return magic.exec(url)
}
var parts = getUrlParts('http://ponyfoo.com/articles/es6-destructuring-in-depth')
var [,protocol,host,pathname,slug] = parts
console.log(protocol)
// <- 'http'
console.log(host)
// <- 'ponyfoo.com'
console.log(pathname)
// <- '/articles/es6-destructuring-in-depth'
console.log(slug)
// <- 'es6-destructuring-in-depth'
```
 
#cCaso especial: declaraciones `import`
 
Aunque las declaraciones `import` no siguen las reglas de desestructuración, se comportan un poco similares. Este es probablemente el caso de uso _“casi-desestructuración”_ que utilizo la mayoría de las veces, aunque no sea desestructuración en realidad. Cuando estés escribiendo declaraciones de módulo `import`, puedes llamar lo que necesites de un módulo de API público. Un ejemplo usando [`contra`](https://github.com/bevacqua/contra):
```javascript  
import {series, concurrent, map } from 'contra'
series(tasks, done)
concurrent(tasks, done)
map(items, mapper, done)
```   
Observa que, sin embargo, las declaraciones `import` tiene una sintaxis diferente. Cuando lo comparas con la desestructuración, no funcionarán ninguna de las siguientes declaraciones import:
 
* Usar valores por defecto tales como `import {series = noop} from ‘contra’`
* Desestructuración “profunda” como `import {map: { series }} from ‘contra’`
* Sintaxis de alias `import {map: mapAsync} from ‘contra’`
 
 
La razón principal de estas limitaciones es que la declaración `import` trae un _enlace_ y no una referencia o valor. Esta es una diferenciación importante que exploraremos más en profundidad en un futuro artículo acerca de los módulos de ES6.
