
Facade, o también llamado fachada, es un patrón de diseño estructural que proporciona una interfaz simplificada a una biblioteca, framework o cualquier otro grupo complejo de clases. 

#### Problema

Imagina que necesitas que tu código trabaje con un amplio grupo de objetos que perteneces a una sofisticada biblioteca o framework. Normalmente, debes inicializar todos los objetos, llevar registro de las dependencias, ejecutar métodos en orden, etc.

#### Solución

Una fachada es una clase que proporciona una interfaz simple a un subsistema complejo. 
También puede ser útil cuando necesitas solo algunas funcionalidades de muchas que ofrece este subsistema.

#### Laravel

Laravel proporciona varios facades en su core para ayudarnos, por ejemplo el Auth Facade:

```php
Auth::login($user);  
Auth::check();
Auth::attempt($credentials);
```

Laravel tambien provee helpers con las mismas funcionalidades:

```php
auth()->login($user);  
auth()->check();
auth()->attempt($credentials);
```

Las fachadas de Laravel sirven como "proxies estáticos" para las clases subyacentes en el contenedor de servicios, brindando el beneficio de una sintaxis concisa y expresiva al tiempo que mantienen más capacidad de prueba y flexibilidad que los métodos estáticos tradicionales

Todas las fachadas de Laravel están definidas en el `Illuminate\Support\Facades`.

Por ejemplo `Route::get` o `Route::post` son facades.

#### Crear Facades en Laravel

Hay una manera normal, y una "mágica" de crear facades en laravel. 
La menera "magica" se llama "Real Time Facades", y consiste basicamente en agregar la palabra Facades al importar una clase con "use" en nuestro codigo, por ejemplo:

```php
<?php

namespace App\Services;

class PriceFormatter {
    public function __construct(
    	private string $currency = 'EUR';
    ) {}

    public function currency(string $currency): static
    {
    	$this->currency = $currency;
        return $this;
    }

    public function format(int|float $price): string
    {
        return number_format($price, 2, ',', '.') . $this->currency;
    }
}
```

Ahora, para usarla, tendríamos que instanciarla y llamar al método `format`:

```php
use App\Services\PriceFormatter;

$formatter = new PriceFormatter();
$price = $formatter->format(10); // 10,00€
```

Para convertir esta clase en una _facade_ automáticamente, solo tenemos que añadir _`Facades\`_ al inicio del `use`:

```php
use Facades\App\Services\PriceFormatter;

$price = PriceFormatter::format(10); // 10,00€
```

Para hacerlo de manera convencional hay que realizar algunos pasos extra:

1. **Crear el Facade**: los facades son típicamente clases estáticas que actúan como "envoltorios" para acceder a instancias de clases subyacentes. Puedes crear tu propio facade extendiendo la clase `Illuminate\Support\Facades\Facade`

```php
// app/Facades/PriceFormatterFacade.php
namespace App\Facades;

use Illuminate\Support\Facades\Facade;

class PriceFormatterFacade extends Facade
{
    protected static function getFacadeAccessor()
    {
        return 'price-formatter';
    }
}
```

2. **Registra el servicio y el alias del facade**: Registra tu servicio `PriceFormatter` en el contenedor de servicios de Laravel para que puedas acceder a él a través del facade.
3. 
```php
// app/Providers/AppServiceProvider.php
use App\Services\PriceFormatter;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->singleton('price-formatter', function () {
            return new PriceFormatter();
        });
    }

    // ...
}
```



