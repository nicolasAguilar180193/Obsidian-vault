___No se debe forzar a los clientes a depender de métodos que no utilizan.___

Intentar que las interfaces sean estrechas para que las clases clientes no tengan que implementar comportamientos que no necesitan.

Interfaces detalladas y especificas para que los clientes implanten solo lo que necesitan. Aprovechando que una clase puede implementar mas de una interfaz. 

### Ejemplo

Creaste una librería para la integración de aplicaciones con varios proveedores de computación en la nube. En un principio solo soportaba AWS con todos los servicios que este proporciona.

Al principio asumiste que todos los proveedores tendrían el mismo espectro de funciones que AWS. Pero cuando quisiste agregar uno nuevo varios de estos métodos no eran necesarios en este.

```php
interface CloudProvider {
	public function storeFile(name);
	public function getFile(name);
	public function createServer(region);
	public function listServers(region);
	public function getCDNAddress();
}

class Amazon implements CloudProvider
{
	public function storeFile(name){...};
	public function getFile(name){...};
	public function createServer(region){...};
	public function listServers(region){...};
	public function getCDNAddress(){...};
}

class Dropbox implements CloudProvider
{
	public function storeFile(name){...};
	public function getFile(name){...};
	public function createServer(region){/*No hace nada*/};
	public function listServers(region){/*No hace nada*/};
	public function getCDNAddress(){/*No hace nada*/};
}
```

ANTES: No todos lo clientes puede satisfacer los requisitos de la interfaz.

```php
interface CloudHostingProvider {
	public function createServer(region);
	public function listServers(region);
}

interface CloudStorageProvider {
	public function storeFile(name);
	public function getFile(name);
}

interface CDNProvider {
	public function getCDNAddress();
}

class Amazon implements CloudHostingProvider, CloudStorageProvider, CDNProvider
{
	public function storeFile(name){...};
	public function getFile(name){...};
	public function createServer(region){...};
	public function listServers(region){...};
	public function getCDNAddress(){...};
}

class Dropbox implements CloudStorageProvider
{
	public function storeFile(name){...};
	public function getFile(name){...};
}
```

Como sucede con otros principios, con éste también puedes ir demasiado lejos. No dividas una interfaz que ya es bastante específica. Recuerda que, cuantas más interfaces crees, más se complicará tu código. Mantén el equilibrio.
