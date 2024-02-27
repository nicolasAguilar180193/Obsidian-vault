___Las clases de alto nivel no deben depender de las clases de bajo nivel. Ambas deben depender de abstracciones. Las abstracciones no deben depender de detalles. Los detalles deben depender de abstracciones.___

* __Clases de bajo nivel__: implementan operaciones básicas, como trabajar con un disco, conectar con una base de datos, etc.
* __Clases de alto nivel__: contienen la lógica de negocio compleja que ordena a las clases de bajo nivel que hagan algo.  

Cuando desarrollas primero las de bajo nivel y sobre estas las de alto nivel, es decir las de la lógica de negocios, estas tienden a hacerse dependientes de las clases de bajo nivel.

Este principio sugiere cambiar la dirección de esta dependencia. 

1. Para empezar, escribir interfaces para operaciones de bajo nivel en las que se basaran las de alto nivel, preferiblemente en términos de negocio. Ejemplo, la lógica de negocio de invocar un método `abrirInforme(archivo)` en lugar de una serie de métodos `abrirArchivo(x)`, `leerBytes(n)`, etc. 
2. Ahora puedes hacer las clases de alto nivel dependientes de esas interfaces, en lugar de las concretas de bajo nivel, Esta dependencia sera mucho mas débil. 
3. Una vez que las clases de bajo nivel implementan esas interfaces, se vuelven dependientes del nivel de la lógica de negocio, invirtiendo la dirección de la dependencia original. 

El principio de inversión de la dependencia suele ir de la mano del principio de abierto/cerrado: puedes extender clases de bajo nivel para utilizarlas con distintas clases de lógica de negocio sin descomponer clases existentes.

### Ejemplo

La clase de alto nivel — que se ocupa de informes presupuestarios — utiliza una clase de base de datos de bajo nivel para leer y almacenar su información.

![[Captura desde 2024-02-27 13-28-05-negate.png]]

Puedes arreglar este problema creando una interfaz de alto nivel que describa operaciones de leer/escribir y haciendo que la clase de informes utilice esa interfaz en lugar de la clase de bajo nivel.

![[Captura desde 2024-02-27 13-28-27-negate.png]]

Como resultado, la dirección de dependencia origina se ha invertido: las clases de bajo nivel dependen ahora de abstracciones de alto nivel. 