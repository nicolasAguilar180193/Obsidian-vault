RxJS es una librería para crear programas asíncronos y basados en eventos usando secuencias observables. Provee un tipo principal, el Observable, y tipos satelites(Observer, Schedulers, Subjects) y operadores basados en métodos de Arrays (map, filter, reduce, etc)
para manejar eventos asíncronos como colecciones. 

ReactiveX combina el [[Observer Pattern]] con el [[Iterator Pattern]] y la programación funcional con colecciones para gestionar secuencias de eventos.

Los conceptos esenciales:

* **Observable**: representa la idea de una colección invocable de futuros valores o eventos. 
* **Observer**: es una colección de callbacks que saben como escuchar a los valores derivados de un Observable. 
* **Subscription**: representa la ejecución de un Observable, es útil para cancelar la ejecución.
* **Operators**: son funciones puras que permiten un estilo de programación funcional de tratar las colecciones con operaciones como: map, filter, concat, reduce, etc.
* **Subject:** es equivalente a un EventEmitter, y la única forma de hacer multicasting, es decir enviar un valor o evento a varios Observers. 
* **Schedulers**: son despachadores centralizados para controlar la concurrencia, lo que nos permite coordinarnos cuando se hace el cómputo, por ejemplo. `setTimeout` o `requestAnimationFrame` o otros.


### Ejemplos

En javascript se registran listeners de eventos de esta menera:

```javascript
document.addEventListener('click', () => console.log('Clicked!'));
```

En RxJS se puede crear un observable: 

```javascript
import { fromEvent } from 'rxjs';

fromEvent(document, 'click').subscribe(() => console.log('Clicked!'));
```