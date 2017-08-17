
# Clase 1

Tuvimos 3 partes

* Aspectos administrativos
* Intro a la materia: vision y motivación
* Sistemas de Tipos

# Aspectos Administrativos

* Tenemos un [sitio](https://sites.google.com/site/programacionhm) con todo el material y donde vamos a publicar enunciados y a hacer el seguimiento de las notas, planificacion
* Vamos a ver 3 tenemas aplicados en ejercicios y un TP para cada tema: 
  * Mixins
  * Metaprogramación
  * FP + OOP (influencia de funcional en Objetos) 
* La evaluación es
  * con 3 TPs, uno por cada tema
  * los TPs son en **grupos de a 2**
* Al final del cuatrimestre habrá oportunidad de recuperar los TPs, ya sea con otros TPs o bien con examenes.

En estos días vamos a estar creando un formulario para que armen los grupos.

# Intro a al materia

Hicimos un repaso de los conceptos de programación vistos en otras materias y dijimos que 
* Muchos de esos conceptos no son específicos de un "paradigma". Ej, recursividad, o delegación, o "divide & conquer" son ideas aplicables a funcional, a objetos y procedural.
* Concluímos en que los **paradigmas** son simplemente un conjunto de estos conceptos, útiles para aprender inicialmente los elementos principales de ese "estilo" de programación en forma más pura. Por ejemplo, aprender polimorfismo y a modelar en objetos sin "marearse" con cosas extra.
* Sin embargo, muchas veces algunos problemas son más "fáciles" o resulta más "natural" resolverlos con uno u otro "paradigma" o estilo.
* Los lenguajes modernos y en el último tiempo se están rompiendo cada vez más los límites entre paradigmas. Existen lenguajes "multiparadigma", e influencias cruzadas de paradigmas.

Finalmente dijimos en cuanto al Paradigma de objetos que
* Si uno considera al paradigma como la visión "original de Smalltalk", entonces ya no hay nada nuevo que aprender más que lo que vieron en Objetos 1 y Objetos 2.
* Entonces necesariamente hay que salir de esa visión rígida de objetos para ver "variantes".

Vimos 2 ejemplos de esas variantes en Javascript (ECMAScript en realidad)

## Prototipado

* Prototipado: como trabajar con objetos sin necesidad de clases. El rol de reutilizar código lo cumple un objeto "común".
  * Cada objeto puede tener un "prototipo" bajo la propiedad `__proto__`
  * El lenguaje tiene delegación automática. Al mandar un mensaje si el objeto no lo entiende, automáticamente se busca en su prototipo,.. y en el prototipo de ese..e tc
  * Esto es muy poderoso, porque la relación es **dinámica**, puede cambiarle el prototipo a los objetos, que sería como la capacidad de convertir un objeto e instancia de otra clase distinta en runtime.

Ejemplos de código (se puede probar en una consola en chrome. Boton derecho "Inspect", y vista "console")
```js
# creamos un objeto vacío
const firulais = {}

# tira error, no entiende el mensaje
firulais.ladrar()

# definimos un objeto que modela lo que es un Perro (como la clase)
const Perro = {
  ladrar() {
    console.log('guau')
  }
}

# firulais es ahora un Perro
firulais.__proto__ = Perro

# ahora sí 
firulais.ladrar()
```

Podemos cambiar el prototipo en ejecución
```js
const PerroDisfonico = {
   ladrar() {
      console.log('...')
   }
}

firulais.__proto__ = PerroDisfonico
firulais.ladrar()     // muestra ...

firulais.__proto__ = Perro
firulais.ladrar()     // muestra guau
```

El objeto puede sobrescribir un metodo del prototipo

```js
const perroAnglosajon = {
  ladrar() {
     console.log('ouf ouf')
  }
}

perroAnglosajon.__proto__ = Perro

perroAnglosajon.ladrar()   /// muestra ouf ouf

```

El dispatching se hace igual que en un lenguaje con clases. Cuando un método del prototipo llama a `this` entonces eso no es él mismo, sino el objeto que recibió el mensaje originalmente.

```js
const PerroConHumor = {
  ladrar() {
    // llama a this, pero él no entiende ese mensaje "humor"
    console.log(this.humor === 'BUENO' ? 'guau guau (bueno humor)' : 'grrrrrr')
}

// objeto que entiende el mensaje humor
const unPerro = {
   humor: 'BUENO'
}

unPerro.__proto__ = PerroConHumor

unPerro.ladrar()   // guau guau (buen humor)

unPerro.humor = 'MALO'
unPerro.ladrar()   // grrrrr
``` 

## Programación declarativa

Luego hicimos un pequeño ejercicio en ecmascript para implementar "validaciones sobre un objeto usuario".
La primera implementación

```js
const validar = usuario => {
  const errores = []
  if (!usuario.nombre)
    errores.push('Nombre es requerido')
  if (!usuario.email)
    errores.push('Apellido es requerido')
  if (usuario.nombre && usuario.nombre.length < 4)
    errores.push('Nombre debe ser mayor a 4 caracteres')

  return errores
}
```
Era bastante fea. Repetía código. No modelaba los conceptos de "checkeo". Y era procedural (muy atada al orden de ejecucion: primero defino una lista de errores que voy a ir mutando y luego retorno esa lista).

Entonces pensamos en cómo nos gustaría escribir sólo las reglas (el QUE) y no cómo se ejecutan (el COMO)

```js
const validaciones = {
  'Nombre es Requerido': usuario => usuario.nombre !== undefined,
  'Apellido es Requerido': usuario => usuario.apellido !== undefined,
  'Email es Requerido': usuario => usuario.email !== undefined
}
```
Esto es un objeto pero que usamos como Mapa/Diccionario.
* claves: mensajes de error a mostrar si no se cumple la regla
* valor: una regla, modelada como una función `f(Usuario) => Boolean`. Si la función da false entonces no cumple con la regla y se debe producir un error con el mensaje.

Entonces ahora sí implementamos el "motor" de validaciones que usa esas "declaraciones" de reglas

```js
const validar = usuario => Object.keys(validaciones)
  .filter(key => 
    !validaciones[key](usuario)
  )
```
Fácil, del objeto validaciones obtiene la lista de claves (con `Object.keys(objeto)`). Y nos quedamos sólo con las que falla.
Para eso en el filter hacemos `validaciones[key]` que es la forma de acceder a los valores (parte derecha) de una entrada en el Mapa/Objeto.
Como eso es una función la ejecutamos `validaciones[key](usuario)`

Luego le dimos una segunda vuelta a las declaraciones porque vimos que el código de las reglas era casi igual en cada una.

```js
usuario => usuario.nombre !== undefined
usuario => usuario.apellido !== undefined
usuario => usuario.email !== undefined
```

Lo único que cambia es la propiedad que checkeamos

```js
usuario => usuario.<<propiedad>> !== undefined
```

Bueno resulta que como dijimos, objetos y mapas en javascript son lo mismo, o se usan en forma similar.
Entonces podemos pensar el objeto "usuario" como un mapa y accederle a cualquier propiedad en forma de mapa

```js
usuario["nombre"]  ==  usuario.nombre

// y usando variables
const nombrePropiedad = "apellido"

usuario[nombrePropiedad]  == usuario.apellido 
```

Entonces podemos expresar esa regla así

```js
const requerido = (usuario, propiedad) => usuario[propiedad] !== undefined

# El !== undefined se puede escirbir en version más corta

const requerido = (usuario, propiedad) => !!usuario[propiedad]
```

Ahora las reglas nos quedan

```js
const validaciones = {
  'Nombre es Requerido': usuario => requerido(usuario, 'nombre'),
  'Apellido es Requerido': usuario => requerido(usuario, 'apellido'),
  'Email es Requerido': usuario => requerido(usuario, 'email')
}
```
Excelente, pero igual nos parecía molesto tener que hacer una función para simplemente llamar a otra.
Entonces analizamos la función `requerido(usuario, nombre)` y entendimos que esos parámetros se saben en "diferentes momentos". Primero a la hora de escribir la regla ya sabemos el `nombre de la propiedad`, pero el `usuario` se va a saber recién cuando se ejecuta la regla.

El "contrato" del mapa de validaciones es "clave - funcion(usuario) -> boolean".
O sea.. uno puede definir la función ahí mismo, como venimos haciendo, ó.. podemos aprovechar la idea de que una función puede recibir por parámetro otra función o bien **retornar una función**.
Entonces pensemos en tener funciones que sean como "fabricas" para crear la función definitiva que valida al usuario.

Entonces retomamos la idea
Al momento de escribir la reglas yo se el nombre de la property, con eso ya puedo crear una función que va a validar al usuario despues.

Primer versión larga
```js
function requiere(nombre) {
   return function(usuario) {
      return usuario[nombre] !== undefined
   }
}
```
Acá se ve que este "requiere" es una función constructora de funciones. Yo le doy el nombre de propiedad y me sabe dar una función que al ejecutarse con un usuario, checkea esa propiedad 

Si quisiera probar un checkeo a mano haría

```js
const checkeoDeEmail = requiere('email')   // me da una función !

const usuario = { nombre: 'Pepe', apellido: 'Muleiro' }

checkeo(usuario)   //   retorna error en email
```

O en un solo paso

```js
requiere('email')(usuario)
```
Llamamos una require, que al darnos otra función llamamos instantaneamente con el usuario.

Nuestras reglas quedarían

```js
const validaciones = {
  'Nombre es Requerido': usuario => requerido('nombre')(usuario),
  'Apellido es Requerido': usuario => requerido('apellido')(usuario),
  'Email es Requerido': usuario => requerido('email')(usuario)
}
```

Sin embargo ahí vemos que es algo no muy inteligente lo que estamos haciendo, porque estamos definiendo una función que recibe un usuario, para dentro, crear una que también recibe al usuario (la que devuelve requerido()) y la estamos ejecutando. 
Entonces el cuerpo de nuestra función es un "pasa manos".
Si requerido() ya nos da una funcion(usuario) entonces podemos usar eso como valor en el mapa ! y que luego lo ejecute el motor directamente, sin el pasamanos

Para eso hay que entender que la parte derecha del mapa, es código que se evalúa al momento de crear el mapa.
Ejemplos pavos
```js
const mapa = {
  edad: 10          // evalúa el 10 que bueno.. no hace nada, y se evalúa a sí mismo
  anios:  23 + 2    // aca es más interesa, evalúa la suma y en el mapa queda el resultado 25. Ac
  sumar: (a,b) => a + b       // metemos una función que sabe sumar. Esto es como el "10" Se evalúa pero no hace nada, en el mapa queda la función
  sumar: miSuma               // si ya tenemos una función "miSuma(a,b) entocnes la podemos usar en lugar de declararla ! 
}
```

Entiendo entonces que al momento de construir el mapa se evalúa lo de la derecha nuestras reglas ahora quedan así

```js
const siempre = () => (usuario) => true

const requerido = (propiedad) => (usuario) => !!usuario[propiedad]

const validaciones = {
  'Nombre es Requerido': requerido('nombre'),
  'Apellido es Requerido': requerido('apellido'),
  'Email es Requerido': requerido('email'),
  'Nunca da error': siempre()
}
```

Y de paso metimos otra regla más "siempre" que nunca falla. Y cambiamos la sintaxis de las funciones para escribirlas más cortas usando "arrow functions".


### Refactorizando aún más (extra no visto en clase)

Es posible seguir achicando este código, con algo que no vimos en clase. El 'operador spread'.
Entendamos ese operador primero.

Dado un mapa/objeto simple

```js
const mapa1 = {
  a: 1,
  b: 2,
  c: 3
}
```
Si queremos crear uno nuevo, que incluya las claves/valores de ese mapa1, existe un operador para esto. 
Sería así

```js
const mapa2 = {
  ...mapa1,
  d: 4
}
// mapa2 es { a: 1, b: 2, c: 3, d: 4 }
```

Esto es muy interesante porque nos permite armar un mapa/objeto con "partes". Que en este es la variable "mapa1", pero podrían venir de una función que retorna un objeto/mapa

Ej

```js
const claveValor = (clave, valor) => {
   return  {
     [clave]: valor
   }
```
Por qué usamos corchetes en las claves ??
Bueno, porque como dijimos antes, la parte de la "derecha" al definir un objeto, se evaluaba y el resultado era lo que quedaba como valor del mapa.
Sin embargo la parte de la izquierda no se evalúa, es "fijo"lo que escribimos.
Ej   `{ edad: 23 }`  "edad" acá no es una variable, es el nombre de la clave.
Y qué pasa si mi código no sabe qué nombre le tiene que dar a la clave, sino que la recibimos como parametro, como en este caso. Necesitmoas que se evalúe la parte izquierda en ese caso. Y eso hacen justamente los corchetes.

Ahora sí podemos armar un mapa de esta otra forma (arbitraria)
```js
const mapa1 = {
   ...claveValor('nombre', 'pepe'),
   ...claveValor('edad', 23)
}
```
Esto "ensambla" un nuevo mapa/objeto copiado otros dos objetos.
El primero es `{ nombre: 'pepe' }`, el segundo es `{ edad: 23 }`. Mapa1 entonces queda `{ nombre: 'pepe', edad: 23 }`

Excelente
Ahora para qué vimos todo esto ?
En qué nos sirve para las validaciones ?

Bueno, lo que hicimos hasta ahora fue reutilizar código que se repetía en el checkeo (parte derecha del mapa).
Pero si miramos bien, cada línea, o sea cada "clave-valor" es muy parecida ! no solo la parte de la derecha, sino todo clave-valor.

```js
  'Nombre es Requerido': requerido('nombre'),
  'Apellido es Requerido': requerido('apellido'),
  'Email es Requerido': requerido('email'),
```

Patron (obviando mayusculas)
```js
'<<<XXXX>>> es Requerido': requerido('<<<XXX>>>'),
```

Entonces qué tal si hacemos una función que sabe crear una de esas "clave-valor" sólo recibiendo lo que es distinto (nombre de la property)

```js
const requiere = (propiedad) => {
  return {
    [`${propiedad} es requerida`]: requerido(propiedad)
  }
}
```
Como la anterior, es una función que genera un objeto/mapa con una única clave-valor. El valor es igual a lo que teníamos.
La clave es "dinámica" de acuerdo al parametro "propiedad". Por eso los [ ]
Estamos usando string interpolations para armar el string sin concatenar (que apesta).

Ahora sí las validaciones quedan como ensamblar varios de esos objetos

```js
const validaciones = {
  ...requiere('nombre'),
  ...requiere('apellido'),
  ...requiere('email'),
  'Nunca da error': siempre()
}
```

### Y se podría haber hecho con objetos ?

Otra versión que usa clases.

Modelamos la idea de un "validador" como un objeto que sabe checkear una condición y retorna una lista de errores o bien una lista vacía (ok, se podría haber hecho de otra forma menos hackerosa)

```js
class Validador {
  constructor(mensaje, condicion) {
    this.mensaje = mensaje
    this.condicion = condicion
  }
  validar(usuario) {
    return this.condicion(usuario) ? [this.mensaje] : []
  }
}
```

Luego las validaciones serían

```js
const validadores = [
  new Validador('Nombre es requerido', usuario => !usuario.nombre),
  new Validador('Apellido es requerido', usuario => !usuario.apellido),
  // etc
]
```
Ojo que es un array de objetos en este caso ya que el objeto tiene ambas cosas: mensaje y lógica

Luego el motor se puede implementar de varias formas. Les queda como ejercicio implementarlo :)
Un tip es usar `reduce` y/o flat/tten

Luego esto también se puede iterar para no repetir tanto código entre regla y regla.
Acá varias opciones desde la más básica a la que menos código requiere y resulta más "expresiva/declarativa" (dice mucho menos el COMO y más el QUE)

```js

const validadores = [
  // forma basica
  new Validador('Nombre es requerido', usuario => !usuario.nombre),

  // soporta otras validaciones distintas
  new Validador('Password mas de 4 caracteres', ({ password }) => password.length > 4)

  // reutilizando el checkeo
  new Validador('Apellido es requerido', campoRequerido('apellido')),

  // reutilizando el validador completo (mensaje y checkeo)
  new CampoRequeridoValidador('titulo'),

  // shortcut para el anterior
  requiere('apellido'),
]
``` 


# Sistemas de Tipos

Finalmente hablamos de sistemas de tipos ya que vamos a estar usando lenguajes bastante distintos en este aspecto.
Y porque los sistemas de tipos son también un tema "caliente" en la industria que está en pos de reinventarse, o de cambiarse en estos momentos.

Acá las diapositivas

https://docs.google.com/presentation/d/1F7wX_ScphEgGiN9wbDxvvru6G2c-UEG3H8mEbL8BdPg/


