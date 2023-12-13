
Los operadores son las piezas esenciales que permiten componer fácilmente código asincrónico complejo de forma declarativa.

#### Que son los operators?

Son operadores o operators funciones. Hay dos tipos de operadores:

**Pipeable Operators** son un tipo de operadores que pueden ser `"piped"` a Observables usando la sintaxis `observableInstance.pipe(operator)`, o mas comúnmente, `observableInstance.pipe(operatorFactory())`.
Operadores de fabrica incluyen, `filter(...)`, y `mergeMap(...)`.

Cuando un Pipeable Operator es llamado, no cambia la instancia Observable. Se retorna un nuevo Observable, con la lógica de suscripción basada en el Observable original.  

_Un Pipeable Operator es una función que toma un Observable como un input y retorna otro Observable. Es una función pura: el Observable previo se mantiene sin modificar. _

**Creation Operators** son el otro tipo de operador, que se puede llamar como funciones independientes para crear un nuevo Observable. Por ejemplo: `of(1, 2, 3)` crea un observable que emite 1, 2 y 3, uno justo luego del otro. 

Por ejemplo el operador `map` es análogo al método de `Array` con el mismo nombre. Como `[1, 2, 3].map(x => x * x)` retorna `[1, 4, 9]`.

```javascript
import { of, map } from 'rxjs';

of(1, 2, 3)
  .pipe(map((x) => x * x))
  .subscribe((v) => console.log(`value: ${v}`));

// Logs:
// value: 1
// value: 4
// value: 9
```

#### Piping 

Pipeable operators son funciones, entonces deberían poder usarse como funciones ordinarias `op()(observable)`, pero en la practica, tiende a haber muchos de ellos convolucionados y rápidamente se vuelven ilegibles: `op4()(op3()(op2()(op1()(observable))))`. Por esta razón, los Observables tienen un método llamado `.pipe()` que logra lo mismo y es mucho más fácil de leer:

```javascript
observable.pipe(op1(), op2(), op3(), op4());
```

Por ejemplo, se podría ejecutar la función map de la siguiente manera:

```javascript
map(x => x * x)(of(1,2,3)).subscribe(v => console.log(`value: ${v}`));
```

Pero de la siguiente manera es mas legible, sobre todo si se quieren aplicar mas operadores:

```javascript
of(1, 2, 3)
	.pipe(map((x) => x * x))
	.subscribe((v) => console.log(`value: ${v}`));
```


#### Creation Operators

Que son? A diferencia de los pipeable operators, los operadores de creación son funciones que pueden ser usados para crear un Observable con algunos comportamientos predefinidos en común o para unirse a otros Observables.

Y ejemplo típico puede ser `interval`. Este toma un numero(no un Observable) como argumento de entrada, y produce un Observable como salida: 

```javascript
import { interval } from 'rxjs';

const observable = interval(1000 /* number of milliseconds */);
```

Crea un Observable que emite números secuenciales cada especificado intervalo de tiempo.

#### Algunos ejemplos de Operators


#### Merge

Crea un Observable de salida que emite simultáneamente todos los valores de cada Observable de entrada dado.

Aplana varios Observables al combinar sus valores en un Observable.

Fusionar dos observables: intervalo de 1 s y clics

```javascript
import { merge, fromEvent, interval } from 'rxjs';

const clicks = fromEvent(document, 'click');
const timer = interval(1000);
const clicksOrTimer = merge(clicks, timer);
clicksOrTimer.subscribe(x => console.log(x));
```

Este emitirá tanto los valores de `interval` como los clics de `fromEvent` en el orden que ocurran.

#### Buffer

Recolecta valores del pasado como una matriz, y emite ese conjunto sólo cuando otro Observable emite.

```javascript
import { fromEvent, interval, buffer } from 'rxjs';

const clicks = fromEvent(document, 'click');
const intervalEvents = interval(1000);
const buffered = intervalEvents.pipe(buffer(clicks));
buffered.subscribe(x => console.log(x));
```

En cada click emitirá los eventos del intervalo mas reciente: `[], [1, 2], [3], [4, 5, 6, 7]`


#### Sample

Emite el valor mas reciente emitido por un Observable fuente

Ejemplo, en cada click, muestra el mas reciente segundo del temporizador:

```javascript
import { fromEvent, interval, sample } from 'rxjs';

const seconds = interval(1000);
const clicks = fromEvent(document, 'click');
const result = seconds.pipe(sample(clicks));

result.subscribe(x => console.log(x));
```

