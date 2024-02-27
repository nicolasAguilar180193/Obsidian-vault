
___Las clases deben estar abiertas a la extensión pero cerradas a la modificación.___

La idea fundamental de este principio es evitar que el código existente se descomponga cuando implementas nuevas funcionalidades. 

La clase esta ___abierta___ si se puede extender, crear una subclase y hacer lo que quieras con ella(añadir nuevos métodos o campos, sobrescribir el comportamiento base, etc.).

La clase esta ___cerrada___ si esta lista al 100% para que otras clases la utilicen; su interfaz esta claramente definida y no se cambiara en el futuro.

Cuando una clase se ha desarrollado, probado, revisado e incluido en un framework o utilizada en una aplicación de cualquier otro modo, es arriesgado hacer cambios en esta. En su lugar puedes crear una subclase y sobrescribir lo que quieras que se comporte de otra forma, y así no descompones los clientes de la clase original. 

### Ejemplo

En una aplicación de comercio electrónico con una clase `pedido` que calcula los costos de envió, todos los métodos de envió están dentro de la clase. Para agregar uno nuevo hay que cambiar el código de la clase pedido.

```php
class Order
{
	private lineItems;
	private shipping;
	
	public getTotal(): float {}
	public getTotalWeigth(): float {}
	
	public getShippingCost(): float {
		if(shipping == 'ground') {
			// Envio por tierra gratuito en grandes pedidos
			if(getTotal()>100)
				return 0;
			// $1.5 por killo, pero $10 de minimo.
			return max(10, getTotalWeigth()*1.5);
		}
		if(shipping == 'air') {
			//$3 por kilo, pero $20 minimo
			return max(20, getTotalWeigth()*3);
		}
	}
}
```

ANTES: Hay que cambiar la clase pedido para agregar un nuevo método de envió.

Se puede resolver aplicando el patrón ___Strategy___. Extrayendo métodos de envió y colocandolos dentro de clases separadas con una interfaz común. 

```php
interface ShippingStrategy {
    public function calculateShippingCost(Order $order): float;
}

class GroundShippingStrategy implements ShippingStrategy 
{
    public function calculateShippingCost(Order $order): float {
        $total = $order->getTotal();
        $weight = $order->getTotalWeight();
        if ($total > 100) {
            return 0;
        }
        return max(10, $weight * 1.5);
    }
}

class AirShippingStrategy implements ShippingStrategy 
{
    public function calculateShippingCost(Order $order): float {
        $weight = $order->getTotalWeight();
        return max(20, $weight * 3);
    }
}

class Order
{
	private lineItems;
	private ShippingStrategy $shippingStrategy;

	public function __construct(ShippingInterface $shippingStrategy)  
	{  
		$this->shippingStrategy = $shippingStrategy;  
	}

	public function setShippingStrategy(ShippingStrategy $strategy): void 
	{ 
		$this->shippingStrategy = $strategy;
	}
	
	public getTotal(): float {}
	public getTotalWeigth(): float {}

	public function getShippingCost(): float 
	{ 
		return $this->shippingStrategy->calculateShippingCost($this);
	}
}
```

#### Ejemplo de uso

```php
$order = new Order(new AirShippingStrategy());

$totalCost = $order->getTotal() + $order->getShippingCost();

echo "Costo total: $" . $totalCost . PHP_EOL;

// Cambiar la estrategia
$order->setShippingStrategy(new GroundShippingStrategy());
```

### En laravel

Podríamos pensar en algo similar pero con los medios de pago

```php
// PaymentGateway.php  
interface PaymentGateway  
{  
	public function processPayment(): void;  
}  
  
// PayPalPaymentGateway.php  
class PayPalPaymentGateway implements PaymentGateway  
{  
	public function processPayment(): void 
	{  
		// PayPal-specific logic  
	}  
}  
  
// StripePaymentGateway.php  
class StripePaymentGateway implements PaymentGateway  
{  
	public function processPayment(): void
	{  
		// Stripe-specific logic  
	}
}
```





