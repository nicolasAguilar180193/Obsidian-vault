
Un Observer es un consumidor de valores entregados por un Observable. Observers son simplemente un conjunto de callbacks, una por cada tipo de notificación entregada por el Observable: `next`, `error` y `complete`. 

```javascript
const observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
  complete: () => console.log('Observer got a complete notification'),
};
```

Para usar el Observer, proveerlo al `subscribe` de un Observable:

```javascript
observable.subscribe(observer);
```

Los Observers en RxJS pueden ser parciales, si no quieres proveer uno de los callbacks, la ejecución del Observable ocurrirá con normalidad, excepto por ese tipo de notificación que sera ignorada, por que justamente no tiene un callback asociado en el Observer. 

Por ejemplo este `Observer` sin el `compolete` callback:

```javascript
const observer = {
  next: x => console.log('Observer got a next value: ' + x),
  error: err => console.error('Observer got an error: ' + err),
};
```

Cuando se suscribe a un `Observable`, se puede proporcionar el callback `next` como argumento, por ejemplo:

```javascript
observable.subscribe(x => console.log('Observer got a next value: ' + x))
```

Internamente en `observable.subscribe`, creara un objeto `Observer` usando el callback del argumento como el `next handler`. 

