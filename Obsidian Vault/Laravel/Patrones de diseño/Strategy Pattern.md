Strategy es un patrón de diseño de comportamiento que te permite definir una familia de algoritmos, colocar cada uno de ellos en una clase separada y hacer sus objetos intercambiables.

### Problema

Supongamos que tenemos una aplicación de navegación para viajeros, con un mapa para ayudar a orientarse en cualquier ciudad.

La función mas popular era la de planificación de rutas, de un origen a un destino.  

La primera versión solo podía generar rutas sobre carreteras, para autos. Luego se agrego para rutas a pie, luego para poder usar transporte publico, y así fue creciendo.

Mantener esta clase principal con tantos algoritmos se vuelve una tarea titanica. 

### Solución

El patrón Strategy sugiere extraer todos esos algoritmos a clases separadas llamadas **_estrategias_**. 

La clase original, llamada **_contexto_**, debe tener un campo para almacenar una referencia a una de las estrategias en particular. Este debe delegar el trabajo a la estrategia vinculada.

La clase **_contexto_** no es responsable de seleccionar la **_estrategia_**, es decir el algoritmo adecuado para la tarea. En lugar de eso, el cliente es el que pasa la estrategia deseada a la clase contexto, el cual expone un único método para dispara el algoritmo encapsulado dentro de la estrategia seleccionada.

De esta forma, el contexto se vuelve independiente de las estrategias, y facilita agregar nuevas o modificar las existentes.


### Ejemplo

Un ejemplo puede ser usado en laravel para implementar diferentes estrategias de chache:

```php
interface CacheStrategy  
{  
	public function cache(string $key, $value);  
}  
  
class FileCacheStrategy implements CacheStrategy  
{  
	public function cache(string $key, $value)  
	{  
		// Cache the value in a file.  
	}  
}  
  
class DatabaseCacheStrategy implements CacheStrategy  
{  
	public function cache(string $key, $value)  
	{  
		// Cache the value in the database.  
	}  
}  
  
class CacheContext  
{  
	private $cacheStrategy;  
	  
	public function __construct(CacheStrategy $cacheStrategy)  
	{  
		$this->cacheStrategy = $cacheStrategy;  
	}  
	  
	public function cache(string $key, $value)  
	{  
		return $this->cacheStrategy->cache($key, $value);  
	}  
}  
  
$cacheContext = new CacheContext(new FileCacheStrategy());  
$cacheContext->cache('key', 'value');
```

