# Iteradores como generadores

### Al parecer, los iteradores pueden ser escritos usando generadores. Esto puede llevar a algunos casos de uso interesantes. Continúa leyendo para entender las propiedades sinergísticas entre estos dos conceptos de iteración en JavaScript.

Repasemos brevemente generadores _(lee nuestra introducción a generadores acá)_.
Funciones generadoras devuelven objetos generadores al ser invocadas.
Un objeto generador tiene un método `next`, el cual devuelve el próximo elemento en la secuencia.
El método `next` devuelve objetos en la forma `{ value, done }`.

En el siguiente ejemplo vemos un generador infinito de números fibonacci.
Luego instanciamos un objeto generador y leemos los primeros ocho valores en la secuencia.

```
function* fibonacci() {
  let previous = 0
  let current = 1
  while (true) {
    yield current
    const next = current + previous
    previous = current
    current = next
  }
}
const g = fibonacci()
console.log(g.next()) // <- { value: 1, done: false }
console.log(g.next()) // <- { value: 1, done: false }
console.log(g.next()) // <- { value: 2, done: false }
console.log(g.next()) // <- { value: 3, done: false }
console.log(g.next()) // <- { value: 5, done: false }
console.log(g.next()) // <- { value: 8, done: false }
console.log(g.next()) // <- { value: 13, done: false }
console.log(g.next()) // <- { value: 21, done: false }
```

Los iteradores siguen un patrón similar _(podés leer nuestra introducción a los iteradores acá)_.
Ellos fuerzan un contrato que dicta que debemos devolver un objeto con un método `next`.
Ese método debería devolver elementos secuenciales siguiendo la forma `{ value, done }`. Los siguientes ejemplos muestran un iterable `fibonacci` que es el equivalente tosco del generador que acabamos de ver.

```
const fibonacci = {
  [Symbol.iterator]() {
    let previous = 0
    let current = 1
    return {
      next() {
        const value = current
        const next = current + previous
        previous = current
        current = next
        return { value, done: false }
      }
    }
  }
}
const sequence = fibonacci[Symbol.iterator]()
console.log(sequence.next()) // <- { value: 1, done: false }
console.log(sequence.next()) // <- { value: 1, done: false }
console.log(sequence.next()) // <- { value: 2, done: false }
console.log(sequence.next()) // <- { value: 3, done: false }
console.log(sequence.next()) // <- { value: 5, done: false }
console.log(sequence.next()) // <- { value: 8, done: false }
console.log(sequence.next()) // <- { value: 13, done: false }
console.log(sequence.next()) // <- { value: 21, done: false }
```

Iteremos.
Un iterable debería devolver un objeto con un método `next`: las funciones generadoras hacen justamente eso.
El método `next` debería devolver objetos en la forma `{ value, done }`: las funciones generadoras hacen eso también.
Qué pasa si cambiamos al iterable `fibonacci` para que use una función generadora como su propiedad `Symbol.iterator`?
Resulta ser que simplemente funciona.

El próximo ejemplo muestra al objeto iterable `fibonacci` usando una función generadora como su iterador.
Noten cómo ese iterador tiene exactamente el mismo contenido que la función generadora `fibonacci` que vimos más temprano.
Podemos usar `yield, yield*`, y todas las semánticas contenidas en las funciones generadoras.

```
const fibonacci = {
  * [Symbol.iterator]() {
    let previous = 0
    let current = 1
    while (true) {
      yield current
      const next = current + previous
      previous = current
      current = next
    }
  }
}
const g = fibonacci[Symbol.iterator]()
console.log(g.next()) // <- { value: 1, done: false }
console.log(g.next()) // <- { value: 1, done: false }
console.log(g.next()) // <- { value: 2, done: false }
console.log(g.next()) // <- { value: 3, done: false }
console.log(g.next()) // <- { value: 5, done: false }
console.log(g.next()) // <- { value: 8, done: false }
console.log(g.next()) // <- { value: 13, done: false }
console.log(g.next()) // <- { value: 21, done: false }
```

Al mismo tiempo, el protocolo de iterables también se mantiene.
Para verificarlo podrían usar un constructo como `for..of`, en lugar de crear el objeto generador manualmente.
El siguiente ejemplo usa `for..of` e introduce un interruptor de circuito para evitar que un bucle infinito haga fallar al programa.

```
for (const value of fibonacci) {
  console.log(value)
  if (value > 20) {
    break
  }
}
// <- 1
// <- 1
// <- 2
// <- 3
// <- 5
// <- 8
// <- 13
// <- 21
```

Este fue un truco divertido.
Para que lo usarían en un programa del mundo real?

# Lectura Adicional

- Iteradores de ES6 en profundidad
- Generadores de ES6 en profundidad
- Símbolos de ES6 en profundidad
- Entendiendo `async/await` en JavaScript _(bonus track!)_
