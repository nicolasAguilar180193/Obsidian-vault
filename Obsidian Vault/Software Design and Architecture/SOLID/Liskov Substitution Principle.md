___Al extender una clase, recuerda que debes tener la capacidad de pasar objetos de las subclases en lugar de objetos de la clase padre, sin descomponer el código cliente.___

El principio de sustitución de Liskov dice que la subclase debe permanecer compatible con el comportamiento de la superclase. Al sobrescribir un método, extiende el comportamiento base en lugar de cambiarlo por algo totalmente distinto. 

Es un grupo de comprobaciones que ayudan a predecir si una subclase permanece compatible con el código que pudo funcionar con objetos de la superclase. Es un concepto fundamental para desarrollar librerías y frameworks. 

#### Requisitos para las subclases

___Los tipos de parámetro en el método de una subclase deben coincidir o ser mas abstractos que los de parámetro del método de la superclase. 

Por ejemplo: un método `alimentar(Gato c)`, en donde el cliente siembre pasa objetos de gatos. 

* __Bien__: Una subclase que sobrescribió el método para que pueda alimentar a cualquier animal: `alimentar(Animal)`. Si pasas esta subclase al cliente en lugar de la superclase como puede alimentar a todos los animales, también puede alimentar a los gatos pasados por este.
* __Mal__: La subclase ahora restringe el método a gatos de Bengala: `alimentar(GatoDeBengala c)` . Esta especificación de gatos no servirá para los gatos genéricos pasados por el cliente y descompondrá su funcionamiento. 

___Los Tipos de retorno en el método de una subclase debe coincidir o ser un subtipo del tipo de retorno del método de la superclase.

Los requisitos de retorno son inversos a los del tipos de parámetro.

Ejemplo: Una clase con un método `comprarGato(): Gato`. El código cliente espera cualquier gato como retorno. 

* __Bien__: Una subclase sobrescribe el método: `comprarGato(): GatoDeBengala`. El cliente obtiene un gato de Bengala, que sigue siendo un gato.
* __Mal__: Una subclase sobrescribe el método: `comprarGato(): Animal`. El código de cliente se descompone por que recibe un animal genérico desconocido(un perro? un oso?). 


___Un método de una subclase no debe arrojar tipos de excepciones que no se espere que arroje el método base.

En otras palabras, los tipos de excepciones deben coincidir o ser subtipos de los que el método base es capaz de arrojar. 
Los `try-catch` del código cliente se dirigen a tipos específicos de excepciones para el método base, una excepción inesperada puede destrozar la aplicación. 


___Una subclase no debe fortalecer las condiciones previas.

Por ejemplo, el método base tiene un parámetro `int`. Si la subclase lo sobrescribe y requiere que el argumento sea positivo(lanzando una excepción si es negativo), esto amplia las condiciones previas. El cliente ahora se rompe si pasa un numero negativo al método si cambia una instancia de la superclase por esta nueva subclase.

___Una subclase no debe debilitar las condiciones posteriores.

Digamos que tienes una clase con un método que trabaja con una base de datos. Se supone que un método de la clase siempre debe cerrar todas las conexiones de las bases de datos abiertas al devolver un valor.
Creaste una subclase y la cambiaste, de modo que las conexiones de la base de datos permanezcan abiertas para poder reutilizarlas. Como ahora nadie tiene la responsabilidad de cerrar estas conexiones el sistema se contamina con conexiones de bases de datos fantasma.


___Las invariantes de una superclase deben preservarse.

Los invariantes son condiciones en las cuales un objeto tiene sentido. Por ejemplo, los invariantes de un gato es tener cuatro patas, una cola, capacidad de maullar, etc. 

Esta regla es la mas fácil de vulnerar, por que puedes no comprender o ser consciente de todos los invariantes de una clase. Por lo tanto, la forma mas segura de extender una clase es introducir nuevos campos y métodos y no tocar con los existentes de la superclase. Por supuesto, esto no siempre es posible.


___Una subclase no debe cambiar los valores de campos privados de la superclase.

Algunos lenguajes de programación permiten acceder a miembros privados de clase mediante mecanismos reflexivos. Otros lenguajes (python, javascript) no tiene protección en absoluto para los miembros privados.

### Ejemplo

```php
class Document
{
	private data;
	private filename;

	public function open(){}
	public function save(){}
}

class ReadOnlyDocument extends Document
{
	public function save(){
		trhow new Exception(
			"No se puede guardar un archivo de solo lectura"
		);
	}
}
```


El método guardar no tiene sentido en un documento de solo lectura, por lo que la subclase intenta resolverlo sobrescribiendolo. Este arroja una excepción si alguien intenta invocarlo. El código cliente se descompondrá si no comprábamos el tipo de documento antes de guardarlo. 

Se puede resolver el problema rediseñando la jerarquía de clases. El documento de solo lectura se convierte en la clase base, y el de escritura ahora es una subclase de este que añade el comportamiento de guardar.

```php
class Document
{
	private data;
	private filename;

	public function open(){}
}

class WritableDocument extends Document
{
	public function save(){}
}
```


