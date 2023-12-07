Observador es un patrón de diseño de software que define la dependencia uno a muchos entre objetos, de manera que cuando uno de los objetos cambia su estado, notifica este cambio a todos los dependientes. Se trata de un patrón de comportamiento.

### Objetivo

Definir una dependencia uno a muchos entre objetos, de tal forma que cuando el objeto cambie de estado, todos sus objetos dependientes sean notificados automáticamente. Se trata de desacoplar la clase de los objetos clientes del objeto, aumentando la modularidad del lenguaje, creando las mínimas dependencias y evitando bucles de actualización (espera activa o sondeo). En definitiva, normalmente, se usará el patrón observador cuando un elemento _quiere_ estar pendiente de otro, sin tener que estar comprobando de forma continua si ha cambiado o no.

### Motivación

Si se necesita consistencia entre clases relacionadas, pero con independencia, es decir, con un bajo acoplamiento. 

### Participantes

* Sujeto(subject): Proporciona una interfaz para agregar(attach) y eliminar(detach) observadores. Conoce a todos sus observadores.
* Observador(observer): Define el metodo que usa el sujeto para notificar cambios de estado(update/notify)
* Sujeto concreto: Mantiene el estado de interes para los observadores concretos y los notifica cuando cambia su estado.
* Observador concreto: Mantiene una referencia al sujeto concreto e implementa la interfaz de actualización, es decir, guardan la referencia del objeto que observan.
