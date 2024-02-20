Considere este problema:

_“Si la dirección de entrega de un pedido se cambia dentro de los límites de tiempo permitidos, envíe una notificación al proveedor de la API de logística. Enviar también un email y un SMS a ese cliente notificándole su decisión”_

En este caso

**El Subject** es pedido de comercio electrónico.

**El Event** es un cambio de dirección.

**Las acciones** a tomar son: Notificar al proveedor de logística conectado, Enviar un correo electrónico y Enviar un SMS. En otras palabras, tenemos tres oyentes interesados ​​en el evento "Cambio de dirección". Este es un caso perfecto de Patrón de Observador.

## ¿Qué es el patrón de observador?

Es un patrón de diseño de comportamiento que define una relación de uno a muchos entre el subject y los observers de manera poco acoplada mediante el envío de notificaciones de eventos a los observadores registrados sobre el cambio en el estado del Sujeto.

![[0_eqmqtR0UzyxbIqpN.webp]]

Esto proporciona una buena medida de acoplamiento flexible (excepto para el registro de observadores). Si desea una solución mantenible y débilmente acoplada para este problema, el patrón Observer es una opción ideal para monolitos mientras que el patrón Pub-Sub lo es para microservicios. 

### ¿Qué problema resuelve el patrón observable?

1. Permitir que varios observadores sean notificados cuando el sujeto cambia de estado. Esto permite actualizar inmediatamente cualquier número de objetos dependientes.

2. Proporcionar un acoplamiento flexible entre el sujeto y los observadores.

3. Promover la reacción asincrónica de eventos dentro del sistema al permitir el procesamiento paralelo (a través de [Jobs y Queues,](https://laravel.com/docs/10.x/queues#introduction) por ejemplo).

## ¿En qué se diferencia el patrón Observer del patrón Pub-Sub?

Pub-Sub promovió un mayor acoplamiento flexible mediante Event Bus o Broker(entidad centralizada que facilita la comunicación entre los diferentes componentes del sistema).
En este mecanismo “Sujeto” y “Observadores” podrán actuar de manera independiente. _En la mayoría de situaciones prácticas, el patrón de observador es mejor para monolitos y pub-sub para microservicios_ .


## Pero ¿cómo puedo implementar esto en Laravel?

Lo que estamos implementando se muestra en el siguiente diagrama:

![[0_zYuFHT5fvjj3gczB.webp]]

Para empezar, la acción **updateAddress() de** **AddressController** valida la entrada usando **AddressChangeRequest** y la convierte en un objeto **AddressChangeDTO** . (Recuerde que [el patrón Adaptador](https://en.wikipedia.org/wiki/Adapter_pattern) funciona como un puente entre dos interfaces incompatibles. Nunca debemos pasar instancias de solicitud HTTP a la clase de Servicio, ya que las solicitudes pueden originarse desde Terminal u otros clientes que no sean HTTP también). Pasa el objeto AddressChangeDTO a **OrderService.**

```php
namespace App\Http\Controllers;

use App\DTO\AddressChangeDTO;
use App\Http\Requests\AddressChangeRequest;
use App\Services\OrderService;

class AddressController extends Controller
{
    private $orderService;
    public function __construct(OrderService $orderService)
    {
        $this->orderService = $orderService;
    }
    public function updateAddress($id, AddressChangeRequest $request)
    {
        $data = $request->validated();
        $valueObject = new AddressChangeDTO(
	        $data['line1'],
			$data['line2'],
			$data['city'],
			$data['pincode']
		);
        $result = $this->orderService->updateAddressForOrder($id, $valueObject);
        return response()->json(
	        $result['message'],
	        $result['status'] == 'SUCCESS' ? 200 : 400
		);
    }
}
```

OrderService usa OrderRepository para actualizar la dirección y pasa el objeto de pedido a AddressChangeNotifService.

```php
namespace App\Services;

use App\DTO\AddressChangeDTO;
use App\Models\Order;
use App\Respository\OrderRepository;
use Exception;

class OrderService
{
    private $orderRepository;
    private $addressChangeService;
    
    public function __construct(
	    OrderRepository $orderRepository,
	    AddressChangeNotifService $addressChangeService
	){
        $this->orderRepository = $orderRepository;
        $this->addressChangeService = $addressChangeService;
    }
    
    public function updateAddressForOrder($id, AddressChangeDTO $data)
    {
        try {
            //should be
            //$order = Order::find($id);
            //just for demo
            $order = new Order();
            //Do update using repo
            //---
            $this->addressChangeService->sendNotification($order);
            return ['message' => "Record Updated", 'status' => 'SUCCESS'];
        } catch (Exception $ee) {
            return ['message' => $ee->getMessage(), 'status' => 'FAULIRE'];
        }
    }
}
```

**AddressChangeNotifService** genera el evento **AddressChangeEvent** (patrón de observador) utilizando Order como asunto. Hemos registrado 3 event-listeners en `App\Services\EventServiceProvider` ( _recuerde que en el patrón Observer, el sistema debe “saber” acerca de este acoplamiento_ ).

```php
protected $listen = [  
	AddressChangeEvent::class => [  
		AddressChangeListenerForLogistics::class,  
		AddressChangeListenerForEmail::class,  
		AddressChangeListenerForSMS::class,  
	]  
];
```

```php
namespace App\Services;

use App\Events\AddressChangeEvent;
use App\Models\Order;

class AddressChangeNotifService
{
    public function sendNotification(Order $order)
    {
        AddressChangeEvent::dispatch($order);
    }
}
```

**El evento es**

```php
namespace App\Events;

use App\Models\Order;
use Illuminate\Foundation\Events\Dispatchable;

class AddressChangeEvent
{
    use Dispatchable;
    public $shortMessage = '';
    public $longMessage = '';

    public function __construct(
        public Order $order
    ) {
        //Get Order from Database
        //Get deliveryAddress from Order
        //Compose a Short Message for SMS and Long SMS for others
        $this->shortMessage = "
	        This is Short Message for change in Delivery Address";
	        
        $this->longMessage = "
	        This is a Long Long Message for change in Delivery Address";
    }
}
```


Se supone que estos listeners deben utilizar Laravel Queue para realizar sus tareas. En nuestro ejemplo, simplemente lo estamos registrando. Puede ver el resultado en `Storage/logs/laravel.log`.

Considere este listener:

```php
namespace App\Listeners;

use App\Events\AddressChangeEvent;
use Illuminate\Support\Facades\Log;

class AddressChangeListenerForLogistics
{
    public function handle(AddressChangeEvent $event): void
    {
        //Real world: Invoke Provider's API
        Log::debug(
	        'Inside AddressChangeListenerForLogistics :' . $event->longMessage
		);
    }
}
```

El método **handle()** de la clase de escucha realiza el trabajo real. Contiene el objeto Evento como argumento: **handle(AddressChangeEvent $event).**

En el mundo real, los listener deben delegar en colas las tareas de bloqueo que consumen mucho tiempo.

Puede descargar la fuente desde [este](https://github.com/mansha99/laravel-observer-pattern) repositorio.

Happy Coding :)