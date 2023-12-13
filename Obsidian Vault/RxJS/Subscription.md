
Una Subscription es un objeto que representa un recurso desechable, usualmente la ejecución de un Observable. Una Subscription tiene un método importante, `unsubscribe`, que no toma argumentos y solo desecha el recurso sostenido por la suscripción.

```javascript
import { interval } from 'rxjs';

const observable = interval(1000);
const subscription = observable.subscribe(x => console.log(x));
// Later:
// This cancels the ongoing Observable execution which
// was started by calling subscribe with an Observer.
subscription.unsubscribe();
```

_Una Subscription esencialmente tiene una sola función `unsubscribe()` para liberar recursos o cancelar ejecuciones de Observables._

