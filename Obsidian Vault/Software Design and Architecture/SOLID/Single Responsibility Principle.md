
___Una clase debe tener una razon para cambiar___

Intenta hacer a cada clase responsable de una única parte de la funcionalidad proporcionada por el software, y haz que esa responsabilidad quede totalmente encapsulada por la clase.

El principal objetivo de este principio es reducir la complejidad.

### Ejemplo

La clase `empleado` tiene dos razones para cambiar.
1. Una razon relacionada con el trabajo principal de la clase: gestionar información de los empleados.
2. O algo relacionado con el formato del informe de horas de trabajo.


```php
class Employee {
	private name;
	function public getName(){}
	function public printTimeSheetReport(){}
}
```

Resuelve el problema moviendo el comportamiento relacionado con la impresión de informes de horas de trabajo a una clase separada.

```php
class Employee {
	private name;
	function public getName(){}
}

class TimeSheetReport {
	print(employe){}
}
```

### Ejemplo con Laravel

Considere el típico controlador. En lugar de cargarlo con varias responsabilidades podemos utilizar el SRP manteniendolo enfocado en manejar las solicitudes HTTP y delegar la lógica empresarial a servicios dedicados. Esto mejora la legibilidad y el mantenimiento. 

```php
class UserController extends Controller  
{
	public function show(User $user)  
	{  
		// Handle HTTP request  
		$userData = UserService::getUserData($user);  
		  
		// Return response  
		return view('user.show', compact('userData'));  
	}  
}
```

