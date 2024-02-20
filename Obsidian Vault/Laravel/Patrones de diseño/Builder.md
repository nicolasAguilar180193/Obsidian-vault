**Builder** es un patrón de diseño creacional que nos permite construir objetos complejos paso a paso. El patrón nos permite producir distintos tipos y representaciones de un objeto empleando el mismo código de construcción.


### Problema

Imagina un objeto complejo que requiere una inicialización laboriosa, paso a paso, de muchos campos y objetos anidados.
Normalmente, este código de inicialización está sepultado dentro de un monstruoso constructor con una gran cantidad de parámetros. O, peor aún: disperso por todo el código cliente.

Si tenemos un objeto que puede tener o no alguno de estos campos, por ejemplo una casa, una puede tener patio, piscina, otra no, otra estatuas pero no piscina, etc. 

Hay dos opciones:
1. Crear una superclase y por cada configuración de casa crear una subclase, pero esto podría generar demasiadas subclases. 
2. Otra es pasar por parámetro estos atributos al constructor de la clase casa y dependiendo cuales se pasen generar el comportamiento adecuado. Pero esto genera un constructor lleno de parámetros muchos de los cuales muchas veces no se usan.

### Solución

El patrón Builder sugiere que saques el código de construcción del objeto de su propia clase y lo coloques dentro de objetos independientes llamados constructores.

```php
class HouseBuilder
{
	function buildWalls()
	function buildDoors()
	function buildGarden()
	function buildGarage()

	function getResult(): House
}
```

El patrón organiza la construcción de objetos en una serie de pasos ( construirParedes , construirPuerta , etc.). Para crear un objeto, se ejecuta una serie de estos pasos en un objeto constructor. Lo importante es que no necesitas invocar todos los pasos, si no solo los necesarios para tu configuración. 

### En laravel el query builder

Usando el Query builder(Generador de consultas) de Laravel, imagina que queremos obtener detalles de un usuario llamado 'John' que tiene más de 18 años:

```php
DB::table('users')  
	->where('name', 'John')  
	->where('age', '>', 18)  
	->orderBy('age')  
	->get();
```

Desglosándolo:

- `DB::table('users')`:  establece el foco en la tabla 'users'.
- `->where('name', 'John')`: Filtra para buscar registros con el nombre 'John'.
- `->where('age', '>', 18)`: Mas filtros para la edad.
- `->orderBy('age')`: Organiza el resultado por la edad.
- `->get()`: Ejecuta la consulta creada y obtiene los resultados.

Internamente, crearía un SQL como este:

```sql
SELECT * FROM users WHERE name = 'John' AND age > 18 ORDER BY age
```


En esencia, el patrón Builder nos permite construir una consulta pieza por pieza, haciendo que el código sea más legible y flexible. Con Laravel, has estado utilizando este patrón sin siquiera darte cuenta.

