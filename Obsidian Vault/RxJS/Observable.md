Are lazy Push collections of multiple values.

En el siguiente ejemplo un Observable inserta los valores 1, 2, 3 inmediatamente(de manera sincrona)  cuando se suscribe, y el valor 4 después de pasado 1 segundo desde la llamada a la suscripción:


```javascript
import { Observable } from 'rxjs';

const observable = new Observable((subscriber) => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});
```

Para invocar al Observable y ver estos valores, necesitamos suscribirnos a el:

```javascript
console.log('just before subscribe');
observable.subscribe({
  next(x) {
    console.log('got value ' + x);
  },
  error(err) {
    console.error('something wrong occurred: ' + err);
  },
  complete() {
    console.log('done');
  },
});
console.log('just after subscribe');
```

Una vez ejecutado veremos este resultado en la consola:

```bash
just before subscribe
got value 1
got value 2
got value 3
just after subscribe
got value 4
done
```

### Pull vs Push

Pull y Push son dos protocolos diferentes para describir como un Productor de datos puede comunicarse con un Consumidor de datos.

##### **Que es Pull?** 

En un sistema basado en Pull(Pull system), el Consumidor determina cuando recibe los datos de el Productor. El Productor no es consciente de cuando los datos serán entregados en Consumidor. 

Toda función en JavaScript es un Pull system. La función es un productor de datos, y el código que la llama la consume a través de "pulling out" un único valor de retorno de esta llamada.  

ES2015 introduce funciones generadores y iteradores(function*), otro tipo de sistema Pull. El código que llama a iterator.next() es el Consumidor, "pulling out" múltiples valores desde el iterador(El Productor). 

##### **Que es Push?** 

En los sistemas Push, el Productor determina cuando le envía datos al Consumidor. 
El Consumidor no es consciente de cuando recibirá los datos.

Las **Promesas** son los tipos de sistema Push mas comunes en JavaScript hoy en día. Una Promesa (el Productor) entrega el valor resuelto a los callbacks registrados (los Consumidores), pero no como las funciones, es la Promesa quien se encarga de determinar con precisión cuando es valor es "pushed" a los callbacks.

RxJS introduce los Observables, un nuevo sistema Push para JavaScript. Un Observable es un productor de múltiples valores, "pushing" estos a los Observers (Consumidores).

* **Función**: es un calculo perezosamente computado que retorna sincronicamente un único valor a la invocación.  
* **Generador** es un calculo perezosamente computado que sincronicamente retorna cero a (potencialmente) infinitos valores en la iteracion. 
* **Promesa** es un computo que puede (o no) eventualmente retornar un valor.
* **Observable** es un computo perezoso que puede sincronica o asincronicamente retornar cero a (potencialmente) infinitos valores desde el momento que se invoca en adelante.

#### Observables como generalización de funciones

Contrariamente a las afirmaciones populares, los Observables no son como EventEmitters ni son como Promesas para múltiples valores. Pueden actuar como EventEmitters en algunos casos, es decir, cuando son son un canal de multidifusion usando RxJS Subjects, pero normalmente no son con como  EventEmitters. 

Los Observables son como funciones con cero argumentos, pero que pueden retornar múltiples valores.

Considere el siguiente código:

```javascript
function foo() {
  console.log('Hello');
  return 42;
}

const x = foo.call(); // same as foo()
console.log(x);
const y = foo.call(); // same as foo()
console.log(y);
```

Debería retornar:

```bash
"Hello"
42
"Hello"
42
```

Se puede escribir el mismo comportamiento con Observables:

```javascript
import { Observable } from 'rxjs';

const foo = new Observable((subscriber) => {
  console.log('Hello');
  subscriber.next(42);
});

foo.subscribe((x) => {
  console.log(x);
});
foo.subscribe((y) => {
  console.log(y);
});
```

Y obtener el mismo resultado.

Esto ocurre por que ambos son lazy computation(computo perezoso). Si no se llama a la función, el `console.log('Hello')` no ocurre. Lo mismo con los Observables, si no lo llama (con `subscribe`), el `console.log('Hello')` no ocurre. Suscribirse a un Observable es análogo a llamar a una función. 

Observables puede emitir valores sincronicamente o asincronicamente.

La diferencia entre un Observable y una función radica en que **los Observables pueden "retornar" múltiples valores a través del tiempo**, algo que las funciones no pueden hacer:

```javascript
function foo() {
  console.log('Hello');
  return 42;
  return 100; // dead code. will never happen
}
```

Las funciones solo pueden retornar un valor, sin embargo los Observables pueden hacer esto: 

```javascript
import { Observable } from 'rxjs';

const foo = new Observable((subscriber) => {
  console.log('Hello');
  subscriber.next(42);
  subscriber.next(100); // "return" another value
  subscriber.next(200); // "return" yet another
});

console.log('before');
foo.subscribe((x) => {
  console.log(x);
});
console.log('after');
```

Con una salida sincrona:

```bash
"before"
"Hello"
42
100
200
"after"
```

Pero también puedes "retornar" valores asincronicamente:

```javascript
import { Observable } from 'rxjs';

const foo = new Observable((subscriber) => {
  console.log('Hello');
  subscriber.next(42);
  subscriber.next(100);
  subscriber.next(200);
  setTimeout(() => {
    subscriber.next(300); // happens asynchronously
  }, 1000);
});

console.log('before');
foo.subscribe((x) => {
  console.log(x);
});
console.log('after');
```

Con la siguiente salida:

```bash
"before"
"Hello"
42
100
200
"after"
300
```

Conclusión:

* `func.call()` significa "dame un valor sincronicamente"
* `observable.subscribe()` significa "dame cualquier cantidad de valores, ya sea sincronica o asincronicamente"


#### Anatomía de un Observable

Observables son creados usando `new Observable` o un operador de creación, son suscritos con un Observer, ejecutan `next/error/complete` para entregar notificaciones al Observer, y podrá disponerse su ejecución. Estos 4 aspectos están codificados en la instancia del Observable, pero algunos de estos aspectos están relacionados con otros tipos. 

##### Creando Observables

El constructor del `Observable` tomo un argumento: la función `subscribe`.
En el siguiente ejemplo se crea un Observable que emite el string `'hi'` cada segundo a un suscriptor.

```javascript
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  const id = setInterval(() => {
    subscriber.next('hi');
  }, 1000);
});
```

_Los Observables pueden ser creados con `new Observable` . Comúnmente se crean usando funciones de creación como `of`, `from`, `interval`, etc_

En el ejemplo anterior, la función `subscribe` es la parte mas importante para describir a un Observable.

##### Suscribirse a un Observable

```javascript
observable.subscribe((x) => console.log(x));
```

No es coincidencia que `observable.subscribe` y la `subscribe` en `new Observable(function subscribe(subscriber) {...}` tengan el mismo nombre. En la librería son diferentes, pero para propósitos prácticos se consideran conceptualmente iguales. 

Cada llamada a `observable.subscribe` activa su propia configuración para ese suscriptor determinado. 

_Suscribirse a un Observable es como llamar a una función, proporcionando callbacks donde los datos serán entregados_

Esto es muy diferente a los event handlres APIs como `addEventListener`/`removeEventListener`. Con `observable.subscribe`, el Observer no es registrado como un listener en el Observable. El Observable no mantiene una lista de Observers adjunta.

Una llamada a `subscribe` simplemente es una manera de iniciar una `"Observable execution"` y retornar los valores o eventos a un Observer de esa ejecución. 


##### Ejecutando Observables

El código dentro de `new Observable(function subscribe(subscriber) {...}` representa una `"Observable execution"`, un computo perezoso que solo ocurre cuando cada Observer se suscribe. La ejecución produce múltiples valores en el tiempo, ya sea sincronica o asincronicamente. 

Hay tres tipos de valores que una `"Observable execution"` puede ofrecer:

* `"Next"` : envía un valor como un Number, String, Object, etc.
* `"Error"` : envía un error JavaScript o una excepción. 
* `"Complete"`: no envía un valor.

`"Next"` es la mas importante y la mas comun, representa el valor actual que va a ser retornado al `subscriber`. `"Error"` y `"Complete"` solo ocurren una vez durante la ejecucion del Observable, y solo puede haber una de ellas. 

_En una ejecución de un Observable, de cero a infinitas notificaciones "Next" pueden ser entregadas. Si un notificación "Error" o "Complete" es ejecutada,  entonces no podrá ejecutar nada mas después._

Los observables se adhieren estrictamente al `Observable Contract`, por lo que no se entregara el valor 4 en este ejemplo:

```javascript
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  subscriber.complete();
  subscriber.next(4); // Is not delivered because it would violate the contract
});
```

Es una buena practica envolver cualquier código en la función `subscribe` con `try/cath`.

```javascript
import { Observable } from 'rxjs';

const observable = new Observable(function subscribe(subscriber) {
  try {
    subscriber.next(1);
    subscriber.next(2);
    subscriber.next(3);
    subscriber.complete();
  } catch (err) {
    subscriber.error(err); // delivers an error if it caught one
  }
});
```


##### Desechando Ejecuciones Observables

Debido a que las ejecuciones observables pueden ser infinitas, es común que un Observador quiera abortar la ejecución. Dado que cada ejecución es exclusiva de un Observador, un vez este obtiene los valores tiene que poder detener esta ejecución y así evitar desperdiciar recursos de computo y memoria. 

La suscripción representa una ejecución en curso, y tiene una API que permite cancelarla  con `subscription.unsubscribe()`.

```javascript
import { from } from 'rxjs';

const observable = from([10, 20, 30]);
const subscription = observable.subscribe((x) => console.log(x));
// Later:
subscription.unsubscribe();
```

_Cuando te suscribes, obtienes una Suscripción, que representa la ejecución en curso. Solo llama a `unsubscribe()`  para cancelarla._

Se pueden personalizar la función de `unsubscribe()` por ejemplo:

```javascript
import { Observable } from 'rxjs';

const subscribe = subscriber => {
	// Keep track of the interval resource
	const intervalId = setInterval(() => {
		subscriber.next('hi');
	}, 1000);
	
	// Provide a way of canceling and disposing the interval resource
	return function unsubscribe() {
		console.log("c'est fini")
		clearInterval(intervalId);
	};
};

const observable = new Observable(subscribe); 
const subscription = observable.subscribe(value => console.log(value));

setTimeout(() => {
	subscription.unsubscribe();
}, 5000);
```
