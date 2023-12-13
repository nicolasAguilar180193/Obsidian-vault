RxJS es una librería para crear programas asíncronos y basados en eventos usando secuencias observables. Provee un tipo principal, el Observable, y tipos satélites(Observer, Schedulers, Subjects) y operadores basados en métodos de Arrays (map, filter, reduce, etc)
para manejar eventos asíncronos como colecciones. 

ReactiveX combina el [[Observer Pattern]] con el [[Iterator Pattern]] y la programación funcional con colecciones para gestionar secuencias de eventos.

Los conceptos esenciales:

* **[[Observable]]**: representa la idea de una colección invocable de futuros valores o eventos. 
* **[[Observer]]**: es una colección de callbacks que saben como escuchar a los valores derivados de un Observable. 
* **[[Subscription]]**: representa la ejecución de un Observable, es útil para cancelar la ejecución.
* **[[Operators]]**: son funciones puras que permiten un estilo de programación funcional de tratar las colecciones con operaciones como: map, filter, concat, reduce, etc.
* **[[Subject]]:** es equivalente a un EventEmitter, y la única forma de hacer multicasting, es decir enviar un valor o evento a varios Observers. 
* **Schedulers**: son despachadores centralizados para controlar la concurrencia, lo que nos permite coordinarnos cuando se hace el cómputo, por ejemplo. `setTimeout` o `requestAnimationFrame` o otros.


### Ejemplos

En javascript se registran listeners de eventos de la siguiente forma:

```javascript
document.addEventListener('click', () => console.log('Clicked!'));
```

En RxJS se puede crear un observable: 

```javascript
import { fromEvent } from 'rxjs';

fromEvent(document, 'click').subscribe(() => console.log('Clicked!'));
```


#### Pureza

Lo que hace a RxJS poderoso es la habilidad de producir valores usando funciones puras. Esto significa que tu código es menos propenso a errores. 
Normalmente se crean funciones impuras, en donde otras piezas de código puede cambiar su estado. 

```javascript
let count = 0;
document.addEventListener('click', 
  () => console.log(`Clicked ${++count} times`)
);
```

Usando RxJS puede aislar este estado:

```javascript 
import { fromEvent, scan } from 'rxjs';

fromEvent(document, 'click')
  .pipe(scan((count) => count + 1, 0))
  .subscribe((count) => console.log(`Clicked ${count} times`));
```

El operador **scan** funciona como el método **reduce** de arrays. Toma el primer valor que es expuesto al callback. El valor retornado por el callback se convierte en el siguiente elemento expuesto a la siguiente llamada del callback. Y así acumula los resultados anteriores para la ejecución actual. 

#### Flow 

RxJS tiene una gama de operadores para ayudar a controlar como fluyen los eventos a través de sus observables. 

Así es como en javascript se permitiría solo 1 click por segundo:

```javascript
let count = 0;
let rate = 1000;
let lastClick = Date.now() - rate;
document.addEventListener('click', () => {
  if (Date.now() - lastClick >= rate) {
    console.log(`Clicked ${++count} times`);
    lastClick = Date.now();
  }
});
```

Con RxJS

```javascript
import { fromEvent, throttleTime, scan } from 'rxjs';

fromEvent(document, 'click')
  .pipe(
    throttleTime(1000),
    scan((count) => count + 1, 0)
  )
  .subscribe((count) => console.log(`Clicked ${count} times`));
```

Otros controles de flujo son: filter, delay, take, distict, etc.

#### Valores

Se pueden transformar los valores pasados a través de obserbables.

Así es como se puede agregar la posición **x** del mouse por cada click en javascript vanilla:

```javascript
let count = 0;
const rate = 1000;
let lastClick = Date.now() - rate;
document.addEventListener('click', (event) => {
  if (Date.now() - lastClick >= rate) {
    count += event.clientX;
    console.log(count);
    lastClick = Date.now();
  }
});
```

Con RxJS:

```javascript
import { fromEvent, throttleTime, map, scan } from 'rxjs';

fromEvent(document, 'click')
  .pipe(
    throttleTime(1000),
    map((event) => event.clientX),
    scan((count, clientX) => count + clientX, 0)
  )
  .subscribe((count) => console.log(count));
```

#### Pipe

Es una función utilitaria que se utiliza para combinar múltiples operadores en una secuencia observable. Un ejemplo simple utilizando la función **from** que crea un Observable a través de un array, una Promesa, un objeto iterable, etc.

```javascript
import { from, map, filter } from 'rxjs';

const array = [1, 2, 3, 10, 20, 30];
const result = from(array);  

result.pipe(
	map(x => x * 2),
	filter(x => x > 2)
).subscribe(x => console.log(x));
```

