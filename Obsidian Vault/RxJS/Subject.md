
**Que es un Subject?** Un Subject RxJS es un tipo especial de Observable que permite que los valores se multidifundan(multicasted) a muchos Observers. Mientras que los Observables simples son unicast(cada Observer suscrito posee una ejecución independiente del Observable), Subjects son multicast, de multidifusion.

_Un Subject es como un Observable, pero puede realizar multidifusion a muchos Observers. Subjects son como EventEmitters: ellos mantienen un registro de muchos listeners_

**Todo Subject es un Observable**. Dado un Subject, puedes suscribirte a el, proporcionando un Observer, que comenzara a recibir valores normalmente. Desde la perspectiva de un Observer, no puede saber si la ejecución proviene de un Observable de unidifusion simple(unicast) o de un Subject.

Internamente el Subject no invoca una nueva ejecución que ofrezca valores. Simplemente registra al Observer en una lista de Observers. 

**Todo Subject es un Observer**. Es un objeto con los métodos `next(v)`, `error(e)` y `complete()`. Para alimentar de un nuevo valor al Subject, solo llame a `next(value)`, y se transmitido a los Observadores registrados.

En el siguiente ejemplo, tenemos 2 Observers adjuntos a un Subject y le suministramos algunos valores:

```javascript
import { Subject } from 'rxjs';

const subject = new Subject();

let sus1 = subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

subject.next(1);
subject.next(2);

sus1.unsubscribe();

subject.next(3);
subject.next(4);

// Logs:
// observerA: 1
// observerB: 1
// observerA: 2
// observerB: 2
// observerB: 3
// observerB: 4
```

Dado que un Subject es un Observer, esto también significa que puede proporcionar un Subject como un argumento para la suscripción de cualquier Observable: 

```javascript
import { Subject, from } from 'rxjs';

const subject = new Subject();

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

const observable = from([1, 2, 3]);

observable.subscribe(subject); // You can subscribe providing a Subject

// Logs:
// observerA: 1
// observerB: 1
// observerA: 2
// observerB: 2
// observerA: 3
// observerB: 3
```

En otras palabras, el Subject funciona como el callback del observable, ya que este cuenta con la función `next` dado que es un Observer también. 
Con este enfoque, estamos convirtiendo una ejecución unicast de un Observable a un multicast a través de un Subject. 

También hay algunas especializaciones de Subject, como `BehaviorSubject`, `ReplaySubject` y `AsyncSubject`. 


### Observables multidifusion

Un Observable multidifusión utiliza un Sujeto bajo el capó para hacer que varios Observadores vean la misma ejecución del Observable.

Los Observers se suscriben el Subject subyacente, y el Subject se suscribe a el Observable fuente. El siguiente ejemplo es similar al ejemplo anterior que usaba `observable.subscribe(subject)`:

```javascript
import { from, Subject, multicast } from 'rxjs';

const source = from([1, 2, 3]);
const subject = new Subject();
const multicasted = source.pipe(multicast(subject));

// These are, under the hood, `subject.subscribe({...})`:
multicasted.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});
multicasted.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

// This is, under the hood, `source.subscribe(subject)`:
multicasted.connect();
```

`multicast` retorna un Observable que luce como un Observable normal, pero funciona como un Subject cuando trata de suscribirse. `multicast` devuelve un `ConectableObservable`, que es un tipo especial de Observable que no comienza a emitir valores hasta que se conecta explícitamente a través del método`conect()`.

El método `conect()` es importante para determinar exactamente cuando comenzara la ejecución del Observable compartido.

Este método o función `multicast` esta obsoleto a partir de la versión 7 y se eliminara en la versión 8. Se puede utilizar el Observable `conectable`, el operador `conect` o `share` en su lugar.

El ejemplo anterior se podría escribir de la siguiente manera con `conectable`:

```javascript
import { from, Subject, connectable } from 'rxjs';

const tick$ = connectable(from([1, 2, 3]), {
	connector: () => new Subject()
});

tick$.subscribe({
	next: (v) => console.log(`observerA: ${v}`),
});

tick$.subscribe({
	next: (v) => console.log(`observerB: ${v}`),
});  

tick$.connect();
```


### BehaviorSubject

Una de las variantes de Subjects, quien tiene una noción de "el valor actual". Este almacena el ultimo valor emitido a sus consumidores,  y cada vez que un nuevo Observer se suscribe recibirá inmediatamente el "valor actual".

_Los `BehaviorSubjects` son útiles para representar "valores a lo largo del tiempo". Por ejemplo, un flujo de eventos de cumpleaños es un Subject, pero un stream de edad de personas debería ser un `BehaviorSubjects`_

En el siguiente ejemplo, el `BehaviorSubject` se inicializa con el valor `0` que recibe el primer Observer cuando se suscribe. El segundo Observer recibe el valor 2 aunque se suscribió  después de que el valor 2 fuera enviado. 

```javascript
import { BehaviorSubject } from 'rxjs';
const subject = new BehaviorSubject(0); // 0 is the initial value

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

subject.next(1);
subject.next(2);

subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

subject.next(3);

// Logs
// observerA: 0
// observerA: 1
// observerA: 2
// observerB: 2
// observerA: 3
// observerB: 3
```

### ReplaySubject

Es similar a `BehaiviorSubject` en que puede enviar antiguos valores a los nuevos suscriptores, pero este puede también almacenar una parte de la ejecución del Observable.

_Un `ReplaySubject` registra múltiples valores desde la ejecución del Observable y reenvía estos a los nuevos suscriptores._

Cuando se crea un `ReplaySubject` se puede especificar cuantos valores reproducir:

```javascript
import { ReplaySubject } from 'rxjs';
const subject = new ReplaySubject(3); // buffer 3 values for new subscribers

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

subject.next(1);
subject.next(2);
subject.next(3);
subject.next(4);

subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

subject.next(5);

// Logs:
// observerA: 1
// observerA: 2
// observerA: 3
// observerA: 4
// observerB: 2
// observerB: 3
// observerB: 4
// observerA: 5
// observerB: 5
```

También puede especificar un _tiempo de ventana_ en milisegundos, además del tamaño del búfer, para determinar la antigüedad de los valores registrados. En el siguiente ejemplo utilizamos un tamaño de búfer grande de `100`, pero un parámetro de tiempo de ventana de solo `500`milisegundos.

```javascript
import { ReplaySubject } from 'rxjs';
const subject = new ReplaySubject(100, 500 /* windowTime */);

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

let i = 1;
setInterval(() => subject.next(i++), 200);

setTimeout(() => {
  subject.subscribe({
    next: (v) => console.log(`observerB: ${v}`),
  });
}, 1000);

// Logs
// observerA: 1
// observerA: 2
// observerA: 3
// observerA: 4
// observerA: 5
// observerB: 3
// observerB: 4
// observerB: 5
// observerA: 6
// observerB: 6
// ...
```


### AsyncSubject 

Es una variante en la que solo se envía el ultimo valor de la ejecución Observable a sus Observers, y solo cuando se completa la ejecución(se ejecuta `subject.complete()`).

```javascript
import { AsyncSubject } from 'rxjs';
const subject = new AsyncSubject();

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

subject.next(1);
subject.next(2);
subject.next(3);
subject.next(4);

subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

subject.next(5);
subject.complete();

// Logs:
// observerA: 5
// observerB: 5
```

