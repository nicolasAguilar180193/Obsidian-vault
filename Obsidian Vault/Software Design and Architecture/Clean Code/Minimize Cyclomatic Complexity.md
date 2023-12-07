La complejidad ciclomatica es una medida de la complejidad estructural de un programa, que esta determinada por el numero de rutas linealmente independientes a través del flujo de control del sistema. 

Algunos formas de minimizarla:

*  Dividir funciones complejas en funciones mas pequeñas y simples que realicen tareas especificas. 
* Utilizar estructuras de control como bucles y sentencias if-else de forma coherente y predecible.
* Utilizar conceptos y técnicas de programación funcional. 
* Utilizar patrones de diseño como como el state pattern. 
* Refactorizar para simplificar el flujo de control. 

Por ejemplo este código:

```typescript
function createCustomer(customer: Customer): Customer {
	if(customer) {
		if(customer.name){
			if(customer.age < 99 && customer.age > 18){
				const createdCustomer = PersistCustomer(customer);
				return createdCustomer;
			} else {
				throw new Error('Customer does not have the right age');
			}
		} else {
			customer.name = 'Unknown';
			if(customer.age < 99 && customer.age > 18){
				const createdCustomer = PersistCustomer(customer);
				return createdCustomer;
			} else {
				throw new Error('Customer does not have the right age');
			}
		}
	} else {
		throw new Error('No customer provided as argument');
	}
}
```

Aunque funciona es difícil de leer y modificar.
Las rutas linealmente independientes en este escenario es cada vez que se introduce una declaración if/else, los flujos de control tienen la opción de ir en 2 direcciones. Estos son caminos separados a tener en cuenta y los que introduce "complejidad". Y como minimizar el numero de caminos?

#### Codificación de ruta feliz 
Es un termino utilizado para describir una función en la que se tiene en cuenta el mejor viaje al codificar. Escribir el código como si nunca se produjeran errores y esperar que el usuario interactue con su interfaz de la manera que "correcta". Por ejemplo el código anterior. 

```typescript
function createCustomer(customer: Customer): Customer {
	const createdCustomer = PersistCustomer(customer);
	return createdCustomer;
}
```

En este escenario siempre se pasa el cliente como argumento y con el nombre y la edad correcta.

Ahora 3 comprobaciones(y sin código duplicado)

1. Comprobar si el cliente esta configurado.
2. Comprobar si el nombre no esta vació; de lo contrario sera "Unknown".
3. Comprar si tiene edad valida. 

```typescript
function createCustomer(customer: Customer): Customer {
    if(!customer)
        throw new Error('Invalid customer');

    if(!customer.name)
        customer.name = 'Unknown';

    if(customer.age < 99 && customer.age > 18)
        throw new Error('Invalid age');
    
	const createdCustomer = PersistCustomer(customer);
	return createdCustomer;
}
```

Sin código duplicado ni anidamientos innecesarios y mucho mas legible.
