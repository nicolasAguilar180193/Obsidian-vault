_Manténgalo ligeramente acoplado, manténgalo en capas, proporcione separación de preocupaciones._

## ¿Qué es la arquitectura monolítica?

La arquitectura monolítica es un enfoque de diseño de software en el que todos los aspectos de la aplicación se entrelazan para formar un módulo grande y, por lo tanto, se desarrolla, implementa y mantiene como una sola unidad.

![[0_bqg3FZAYol-AJvo6.webp]]

Esto contrasta con **_la arquitectura de microservicios_** , donde los aspectos se dividen en varios servicios independientes débilmente acoplados que pueden implementarse y mantenerse independientemente unos de otros. Los microservicios se comunican entre sí mediante un bus de eventos o una cola de mensajes.

![[0_bqg3FZAYolss-AJvo6.webp]]

**Tenga en cuenta que:**

1. Los sistemas monolíticos pueden tener el concepto de servicios. Sin embargo, dado que todos los servicios están empaquetados como una sola unidad, es difícil modificarlos, implementarlos o escalarlos de forma independiente.
2. Los microservicios necesitan [API Gateway](https://www.redhat.com/en/topics/api/what-does-an-api-gateway-do) y [Event Bus](http://www.rribbit.org/eventbus.html) como componentes adicionales.
3. La API gateway acepta todas las llamadas API, maneja tareas comunes como registro, autenticación, limitación de velocidad y estadísticas y devuelve la respuesta al cliente.
4. Un Event Bus permite la comunicación entre microservicios al permitir que algunos microservicios actúen como publisher de eventos mientras que otros como subscribers.
5. Aspectos como [el almacenamiento en caché](https://redisson.org/glossary/redis-caching.html) , [la búsqueda elástica](https://www.elastic.co/what-is/elasticsearch) , las notificaciones, etc. también pueden integrarse en aplicaciones monolíticas.

**¿Deberíamos utilizar arquitectura monolítica o arquitectura de microservicios?**

Las aplicaciones monolíticas son fáciles de desarrollar, implementar, probar y depurar, _siempre que sigamos principios de diseño como la separación de preocupaciones y adoptemos una arquitectura en capas._

Si queremos escalar aplicaciones o algunos servicios según la demanda; La arquitectura de microservicios tiene sentido. Los microservicios son fáciles de escalar, mantener, implementar y proporcionan un mejor aislamiento de fallas.

_La idea es crear una aplicación monolítica teniendo en cuenta la arquitectura en capas, el acoplamiento flexible, los principios SÓLIDOS y la separación de preocupaciones_ . Esto permitirá una fácil transición de servicios monolíticos a microservicios.


## **Conceptos básicos**

#### **Servicio**

Un servicio es un componente sin estado débilmente acoplado diseñado para resolver un problema o realizar una tarea específica y tiene la capacidad de comunicarse con otros componentes del sistema, creando así una aplicación compuesta. Service components deben ser reutilizables, reemplazables, independientes del contexto (en la medida de lo posible), extensibles e independientes.

#### **Bajo acoplamiento**

Service Components aka “Server Component”, están diseñados para proporcionar algún tipo de servicio a los “Client Components”. Por ejemplo, podemos diseñar un componente "GithubUserService" diseñado para proporcionar detalles del usuario de github a un Controlador. Dicho Controlador puede denominarse client component.

![[0_RervChK7bPfdsdsdsOdkuR.webp]]

Los Service Components deben diseñarse teniendo en cuenta el acoplamiento flojo. _El acoplamiento es el grado de dependencia de un componente respecto de otro_ . **El acoplamiento flexible (dependencia mínima) es una buena idea ya que permite que los componentes se desarrollen y utilicen de forma independiente unos de otros. El acoplamiento flojo se logra mediante el uso de interfaces.** Esto significa que en lugar de que el Componente X y el Componente Y hablen entre sí, se comunican mediante una interfaz común I. Esta interfaz es implementada por el Service component y el Client component interactúa con el Service component mediante la interfaz I. Esto permite que los Componentes del Servidor se actualicen o modificado sin notificar a los Componentes del Cliente.

![[0_syACfT2qy2n-trsXXX.webp]]

##### Usando interfaces

![[0_wh-eehUGVnc93xZZZj.webp]]

##### **El acoplamiento estrecho es una mala idea**

```php
namespace App\Services;

class FooVehicleService
{
    public function find($vehicleNumber)
    {
        //logic to find Vehicle, probably in database 
    }
}
```

```php
namespace App\Http\Controllers;

class VehicleSearchController extends Controller
{
    public function searchVehicle(VehicleSearchRequest $request)
    {
        $data = $request->validated();
        //Instantiate (FooVehicleService is directly EXPOSED)
        $service = new FooVehicleService();
        //Use
        $response = $service->find($data['vehcileNumber']);
        return response()->json([
		        'data' => $response],
		        $response == null ? 404 : 200);
    }
}
```

##### **La separación basada en acoplamiento flojo/interfaz es genial**

Imagine que **FooVehicleService** se utiliza en varios lugares de la aplicación y luego queremos reemplazar FooVehicleService con **BarVehicleService.** Tenemos que hacer cambios en muchos lugares. Claramente estamos rompiendo la regla DRY (No repetir lo mismo). Para resolver esto, usemos un acoplamiento flojo de la siguiente manera:

1. Crear interfaz de servicio
2. Deje que FooVehicleService implemente esa interfaz
3. Cree una carpeta que devuelva la implementación de la instancia de servicio en la interfaz
4. Utilice la interfaz en Client Controller en lugar de la implementación

##### **Interfaz para acoplamiento flojo**

```php
interface IVehicleService
{
    public function find($vehicleNumber);
}
```

##### **Clase de implementación**

```php
namespace App\Services;

class FooVehicleService implements IVehicleService
{
    public function find($vehicleNumber)
    {
        //logic to find Vehicle, probably in database 
    }
}
```

##### **Binder**

```php
class VehicleServiceBinder
{
    //Singleton Pattern
    private static $instance;
    
    public static function getVehicleService()
    {
        if (self::$instance == null) {
            self::$instance = new FooVehicleService();
        }
        return self::$instance;
    }
}
```

##### **Usando la interfaz en lugar de la implementacion**

```php
namespace App\Http\Controllers;

class VehicleSearchController extends Controller
{
    public function searchVehicle(VehicleSearchRequest $request)
    {
        $data = $request->validated();
        //Instantiate
        $service = VehicleServiceBinder::getVehicleService();
        //Use
        $response = $service->find($data['vehcileNumber']);
        return response()->json([
	        'data' => $response],
	        $response == null ? 404 : 200);
    }
}
```

Ahora, si la implementación cambia, solo tenemos que actualizar Binder sin notificar al Componente del Cliente.

```php
if(self::$instance ==null){  
	self::$instance = new BarVehicleService();  
}
```

## **Inyección de dependencia**

Normalmente, si el Componente-X requiere el Componente-Y, el Componente-X crea una instancia del Componente-Y para su uso. Esto hace que X dependa directamente de Y.

¿Qué pasa si queremos proporcionar un acoplamiento flojo?

¿Qué sucede si queremos realizar pruebas sencillas en las que el Componente-X pueda probarse independientemente del Componente-y?  
La solución es [**la inyección de dependencia**](https://www.codemag.com/Article/2212041/Dependency-Injection-and-Service-Container-in-Laravel) .

> La inyección de dependencia proporciona inversión de control al permitir que un agente "Z", como "Contenedor" o "Binder", inyecte "Y" en "X" a través del constructor u otro mecanismo.

Laravel proporciona [un contenedor de servicios](https://laravel.com/docs/10.x/container) para gestionar la inyección de dependencias.

Algunos ejemplos son

Si inyectamos una variable de clase de servicio dentro del constructor de un componente de cliente, el contenedor de servicio creará automáticamente una instancia del componente de servicio al crear una instancia del componente de cliente.

```php
class GithubUserController extends Controller
{
    private GithubUserService $githubUserService;
    public function __construct(GithubUserService $githubUserService)
    {
        $this->githubUserService = $githubUserService;
    }
}
```

Si inyectamos una variable de interfaz de servicio dentro del constructor de un componente de cliente, el contenedor de servicio creará una instancia del componente de implementación utilizando los enlaces apropiados.

```php
class GithubUserService
{
    private GithubUserRepoInterface $githubUserRepo;
    public function __construct(GithubUserRepoInterface $githubUserRepo)
    {
        $this->githubUserRepo = $githubUserRepo;
    }
}
```

Todo lo que tenemos que hacer es agregar una línea al método Register() de App\Providers\ **AppServiceProvider de la siguiente manera:**

```php
$this->app->bind(GithubUserRepoInterface::class, GithubUserRepoHttpImpl::class);
```

Esto le indicará al contenedor de servicios que cree una instancia de la clase **GithubUserRepoHttpImpl** si se inyecta **GithubUserRepoInterface** .

## **Separación de intereses**

La separación de preocupaciones consiste en dividir una aplicación en componentes separados por límites de responsabilidades. Por ejemplo, es una mala idea mantener la lógica de validación, la lógica de acceso a la base de datos, la lógica de presentación, etc. en el mismo lugar. Deberíamos crear servicios separados para cada uno y unirlos dentro del componente Cliente. La separación de preocupaciones es un paso importante en el diseño de aplicaciones que siguen una arquitectura en capas.

Considere este [Código Spaghetti](https://en.wikipedia.org/wiki/Spaghetti_code) (código fuente no estructurado y difícil de mantener) que no tiene en cuenta la preocupación por la separación.

![[0_mA8fDOt4aXc4kdWo.webp]]

+ ¿Qué pasa si la interacción de la base de datos o la lógica empresarial abarca varias líneas?
+ ¿Qué pasa si se requiere la misma lógica empresarial en otros lugares?
+ ¿No estamos rompiendo la regla DRY?
+ ¿Es este código fácil de mantener?

La solución está en **separar las preocupaciones.**

Como se muestra en el diagrama, las preocupaciones se separan y el código espagueti se reemplaza por las respectivas invocaciones de servicio.

![[0_VrORwhIQ6NnGFu5X.webp]]

**MVCS (Model View Controller Services) : MVC con esteroides**

Laravel sigue el patrón de diseño MVC para proporcionar la separación de preocupaciones.
Como señaló [Ryan Rebo](https://medium.com/marmolabs/skinny-models-skinny-controllers-fat-services-e04cfe2d6ae) , una versión más evolucionada delegaría la lógica y las reglas de la aplicación a servicios separados.

![[0_PRzij1R6lLBEuqK3.webp]]

Deberíamos reducir o limpiar nuestros modelos gordos para facilitar la lectura del código. Es mejor crear servicios, cada uno de los cuales se ocupará de un problema distinto. Los objetos de servicio son un gran recurso para ayudar a mejorar la legibilidad de su código y mantener las cosas con una sola responsabilidad.

# Empecemos a construir

La aplicación que estamos diseñando

1. Hay un controlador (GithubUserController) para manejar solicitudes http y un comando (GetGithubUserCommand) acepta la entrada del usuario. En ambos casos, se ingresa "nombre de usuario" y los atributos "bio", "html_url" y "grupo" son la salida. Aquí "grupo" es un atributo calculado. si seguidores>1000, el grupo es "Experto" o "Principiante".
2. Tanto el Controlador como el Comando interactúan con "GithubUserService" para obtener el perfil de Github para un usuario determinado.
3. GitHubService está utilizando el repositorio "GithubUserRepo". Este repositorio invoca el punto final HTTP-API **https://api.github.com/users/username** para obtener información del usuario.
4. **“GithubUserRepoHttpImpl” es una implementación http de** GithubUserRepo (acoplamiento flexible). Esto nos ayuda a cambiar a la implementación grpc, por ejemplo, en el futuro.
5. Una vez que el servicio Github obtiene datos de GithubUserRepo, aplica la transformación utilizando la biblioteca PHP-Fractal y devuelve la respuesta al cliente. Fractal proporciona una capa de presentación y transformación para la salida de datos complejos, actuando así como una "barrera" entre los datos de origen y la salida, de modo que los cambios de esquema no afecten a los usuarios.

![[0_wGiyhqjutI9RRg1L.webp]]

**Repositorio**

```php
namespace App\Contracts;

interface GithubUserRepo
{
    public function getUser(string $username);
}
```

```php
namespace App\Repos;

use App\Contracts\GithubUserRepo;
use Exception;
use Illuminate\Support\Facades\Http;

class GithubUserRepoHttpImpl implements GithubUserRepo
{
    public function getUser(string $username)
    {
        $response = Http::get('https://api.github.com/users/' . $username);
        if ($response->status() == 404) {
            throw new Exception("User " . $username . " does not exist");
        }
        return $response->json();
    }
}
```

**Service and Transformer**

```php
namespace App\Services;

use App\Contracts\GithubUserRepo;
use App\Transformers\GithubUserTransformer;
use Exception;

class GithubUserService
{
    private GithubUserRepo $githubUserRepo;
    private $transformer;

    public function __construct(
	    GithubUserRepo $githubUserRepo,
	    GithubUserTransformer $transformer
	){
        $this->githubUserRepo = $githubUserRepo;
        $this->transformer = $transformer;
    }
    
    public function getUser($username)
    {
        try {
            $user = $this->githubUserRepo->getUser($username);
            return [
	            'status' => 'success',
	            'user' => $this->transformer->transform($user)
			];
        } catch (Exception $ex) {
            return ['status' => 'failure', 'message' => $ex->getMessage()];
        }
    }
}
```

```php
namespace App\Transformers;

use League\Fractal\TransformerAbstract;

class GithubUserTransformer extends TransformerAbstract
{
    public function transform($record)
    {
        return [
            'bio' => $record['bio'],
            'html_url' => $record['html_url'],
            'group' => $record['followers'] > 1000 ? 'Expert' : 'Novice'
        ];
    }
}
```

**Http Controller as Client Component**

```php
namespace App\Http\Controllers;

use App\Http\Requests\GithubUserRequest;
use App\Services\GithubUserService;

class GithubUserController extends Controller
{
    private GithubUserService $githubUserService;
    
    public function __construct(GithubUserService $githubUserService)
    {
        $this->githubUserService = $githubUserService;
    }
    
    public function getUser(GithubUserRequest $request)
    {
        $data = $request->validated();
        $result = $this->githubUserService->getUser($data['username']);
        return response()->json($result);
    }
}
```

**Console Command as Client**

```php
namespace App\Console\Commands;

use App\Services\GithubUserService;
use Illuminate\Console\Command;

class GetGithubUserCommand extends Command
{
    protected $signature = 'app:github-user {username}';

    protected $description = 'Use this command to get Github user details';

    private GithubUserService $githubUserService;
    
    public function __construct(GithubUserService $githubUserService)
    {
        parent::__construct();
        $this->githubUserService = $githubUserService;
    }
    
    public function handle()
    {
        $result = $this->githubUserService->getUser($this->argument('username'));
        if ($result['status'] == "success") {
            $this->info('Bio: ' . $result['user']['bio']);
            $this->info('User is ' . $result['user']['group']);
            $this->info('More details at ' . $result['user']['html_url']);
        } else {
            $this->error(
	            'User ' . $this->argument('username') . ' does not exist'
			);
        }
    }
}
```

Usar comando:

```bash
php artisan app:github-user twitter
```

> **La siguiente parte de este tutorial que explica el patrón Observer está** [[Laravel/Patrones de diseño/Observer Pattern|Observer Pattern]].

Happy Coding :)