
**Singleton** es un patrón de diseño creacional que nos permite asegurarnos de que una clase tenga una única instancia, a la vez que proporciona un punto de acceso global a dicha instancia.

El patrón tiene prácticamente los mismos pros y contras que las variables globales. Aunque son muy útiles, rompen la modularidad de tu código.

No se puede utilizar una clase que dependa del Singleton en otro contexto. Tendrás que llevar también la clase Singleton. La mayoría de las veces, esta limitación aparece durante la creación de pruebas de unidad.

### Problema

El patrón Singleton resuelve dos problemas al mismo tiempo, vulnerando el _Principio de responsabilidad única_:

+ **Garantizar que una clase tenga una única instancia**:
	Funciona así: imagina que has creado un objeto y al cabo de un tiempo decides crear otro nuevo. En lugar de recibir un objeto nuevo, obtendrás el que ya habías creado.
	
	Ten en cuenta que este comportamiento es imposible de implementar con un constructor normal, ya que una llamada al constructor siempre **debe** devolver un nuevo objeto por diseño.
	
+ **Proporcionar un punto de acceso global a dicha instancia**:
	Al igual que una variable global, el patrón Singleton nos permite acceder a un objeto desde cualquier parte del programa. No obstante, también evita que otro código sobreescriba esa instancia.

### Solución

Todas las implementaciones del patrón Singleton tienen estos dos pasos en común:

+ Hacer privado el constructor por defecto para evitar que otros objetos utilicen el operador `new` con la clase Singleton.
+ Crear un método de creación estático que actúe como constructor. Tras bambalinas, este método invoca al constructor privado para crear un objeto y lo guarda en un campo estático. Las siguientes llamadas a este método devuelven el objeto almacenado en caché.

Si tu código tiene acceso a la clase Singleton, podrá invocar su método estático. De esta manera, cada vez que se invoque este método, siempre se devolverá el mismo objeto.

```php
<?php

namespace RefactoringGuru\Singleton\RealWorld;

/**
 * If you need to support several types of Singletons in your app, you can
 * define the basic features of the Singleton in a base class, while moving the
 * actual business logic (like logging) to subclasses.
 */
class Singleton
{
    /**
     * The actual singleton's instance almost always resides inside a static
     * field. In this case, the static field is an array, where each subclass of
     * the Singleton stores its own instance.
     */
    private static $instances = [];

    /**
     * Singleton's constructor should not be public. However, it can't be
     * private either if we want to allow subclassing.
     */
    protected function __construct() { }

    /**
     * Cloning and unserialization are not permitted for singletons.
     */
    protected function __clone() { }

    public function __wakeup()
    {
        throw new \Exception("Cannot unserialize singleton");
    }

    /**
     * The method you use to get the Singleton's instance.
     */
    public static function getInstance()
    {
        $subclass = static::class;
        if (!isset(self::$instances[$subclass])) {
            // Note that here we use the "static" keyword instead of the actual
            // class name. In this context, the "static" keyword means "the name
            // of the current class". That detail is important because when the
            // method is called on the subclass, we want an instance of that
            // subclass to be created here.

            self::$instances[$subclass] = new static();
        }
        return self::$instances[$subclass];
    }
}

/**
 * The logging class is the most known and praised use of the Singleton pattern.
 * In most cases, you need a single logging object that writes to a single log
 * file (control over shared resource). You also need a convenient way to access
 * that instance from any context of your app (global access point).
 */
class Logger extends Singleton
{
    /**
     * A file pointer resource of the log file.
     */
    private $fileHandle;

    /**
     * Since the Singleton's constructor is called only once, just a single file
     * resource is opened at all times.
     *
     * Note, for the sake of simplicity, we open the console stream instead of
     * the actual file here.
     */
    protected function __construct()
    {
        $this->fileHandle = fopen('php://stdout', 'w');
    }

    /**
     * Write a log entry to the opened file resource.
     */
    public function writeLog(string $message): void
    {
        $date = date('Y-m-d');
        fwrite($this->fileHandle, "$date: $message\n");
    }

    /**
     * Just a handy shortcut to reduce the amount of code needed to log messages
     * from the client code.
     */
    public static function log(string $message): void
    {
        $logger = static::getInstance();
        $logger->writeLog($message);
    }
}

/**
 * Applying the Singleton pattern to the configuration storage is also a common
 * practice. Often you need to access application configurations from a lot of
 * different places of the program. Singleton gives you that comfort.
 */
class Config extends Singleton
{
    private $hashmap = [];

    public function getValue(string $key): string
    {
        return $this->hashmap[$key];
    }

    public function setValue(string $key, string $value): void
    {
        $this->hashmap[$key] = $value;
    }
}

/**
 * The client code.
 */
Logger::log("Started!");

// Compare values of Logger singleton.
$l1 = Logger::getInstance();
$l2 = Logger::getInstance();
if ($l1 === $l2) {
    Logger::log("Logger has a single instance.");
} else {
    Logger::log("Loggers are different.");
}

// Check how Config singleton saves data...
$config1 = Config::getInstance();
$login = "test_login";
$password = "test_password";
$config1->setValue("login", $login);
$config1->setValue("password", $password);
// ...and restores it.
$config2 = Config::getInstance();
if ($login == $config2->getValue("login") &&
    $password == $config2->getValue("password")
) {
    Logger::log("Config singleton also works fine.");
}

Logger::log("Finished!");
```

```
2018-06-04: Started!
2018-06-04: Logger has a single instance.
2018-06-04: Config singleton also works fine.
2018-06-04: Finished!
```


### Ejemplos de clases que implementan Singleton en Laravel

#### Request Class

La clase request es un singleton, una sola clase es creada. Esto es así para que la información acerca de la petición actual pueda ser consultada desde cualquier parte de la aplicación. 

#### Event Class 

La clase Event es responsable de enviar eventos, que son como notificaciones de que algo ha sucedido en la aplicación. Los eventos se pueden escuchar y responder por cualquier código de la aplicación.

#### Database Connection

La capa de base de datos de Laravel también se implementa mediante singletons. Esto significa que solo hay una conexión a la base de datos por solicitud. Esto ayuda a mantener la capa de base de datos de la aplicación limpia y fácil de usar.

#### Facades

Las fachadas también son en su mayoría objetos únicos. Una fachada es una clase que proporciona acceso a un objeto desde el marco de Laravel. Por ejemplo, la fachada Auth da acceso a las funciones de autenticación de Laravel.


### Crear clases singleton en laravel con Services Provideres

```php
class AppServiceProvider extends ServiceProvider
{

	public function register(): void
	{
		$this->app->singleton(TokenService::class, function ($app) {
			$key = config('app.key');
			return new TokenService($key);
		});
	}
}
```
