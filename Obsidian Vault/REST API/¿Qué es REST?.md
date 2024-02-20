<br>
La transferencia de estado representacional (REST) es una arquitectura de software que impone condiciones sobre cómo debe funcionar una API.

En el contexto de servicios web y desarrollo web, REST se basa en la idea de que los recursos (como datos o servicios) son identificables por URIs (Uniform Resource Identifiers).

La frase "Transferencia de Estado Representacional" en REST se puede desglosar de la siguiente manera para entender su significado:

1. **Transferencia de Estado:** Implica que cuando un cliente realiza una solicitud a un servidor, la representación del estado del recurso solicitado se transfiere al cliente. El "estado" se refiere a la información actual o la condición del recurso en un momento específico.
    
2. **Representacional:** Significa que la información transferida entre el cliente y el servidor está en forma de representaciones de recursos, es decir no el recurso como tal si no una representación actual del mismo. Una representación puede ser en formato JSON, XML, HTML, o cualquier otro formato que ambas partes acuerden. El cliente, al recibir esta representación, debería tener toda la información necesaria para comprender y manipular el recurso. 


Los desarrolladores de API pueden diseñar API por medio de varias arquitecturas diferentes. Las API que siguen el estilo arquitectónico de REST se llaman API REST. Los servicios web que implementan una arquitectura de REST son llamados servicios web RESTful. El término API RESTful suele referirse a las API web RESTful. Sin embargo, los términos API REST y API RESTful se pueden utilizar de forma intercambiable.<br>


### Principios del estilo arquitectónico de REST:
<br>
#### Interfaz uniforme (Uniform Interfaz)

La petición deberá identificar los recursos que esta buscando(a través de una URL).
El servidor transfiere información en un formato estándar. El recurso formateado se denomina representación en REST. Este formato puede ser diferente de la representación interna del recurso en la aplicación del servidor. Por ejemplo, el servidor puede almacenar los datos como texto, pero enviarlos en un formato de representación HTML.

La interfaz uniforme impone cuatro limitaciones de arquitectura:

1. Las solicitudes deben identificar los recursos. Lo hacen mediante el uso de un identificador uniforme de recursos.<br>
2. Los clientes tienen información suficiente en la representación del recurso como para modificarlo o eliminarlo si lo desean. El servidor cumple esta condición por medio del envío de los metadatos que describen el recurso con mayor detalle.<br>
3. Los clientes reciben información sobre cómo seguir procesando la representación. El servidor logra esto enviando mensajes autodescriptivos que contienen metadatos sobre cómo el cliente puede utilizarlos de mejor manera.<br>
4. Los clientes reciben información sobre todos los demás recursos relacionados que necesitan para completar una tarea. El servidor logra esto enviando hipervínculos en la representación para que los clientes puedan descubrir dinámicamente más recursos.


#### Sin estado

En la arquitectura de REST, la tecnología sin estado se refiere a un método de comunicación en el cual el servidor completa todas las solicitudes del cliente independientemente de todas las solicitudes anteriores. Los clientes pueden solicitar recursos en cualquier orden, y todas las solicitudes son sin estado o están aisladas del resto. Esta limitación del diseño de la API REST implica que el servidor puede comprender y cumplir por completo la solicitud todas las veces.

### Sistema por capas

Los servicios REST están orientados a la escalabilidad y el cliente no sabe si la petición se realiza directamente a un servidor , un sistema de caches, o por ejemplo un balanceador que se encarga de redirigirlo a un servidor final. Estas capas se mantienen invisibles para el cliente.


### Almacenamiento en caché

Los servicios web RESTful admiten el almacenamiento en caché, que es el proceso de almacenar algunas respuestas en la memoria caché del cliente o de un intermediario para mejorar el tiempo de respuesta del servidor. Por ejemplo, suponga que visita un sitio web que tiene imágenes comunes en el encabezado y el pie de página en todas las páginas. Cada vez que visita una nueva página del sitio web, el servidor debe volver a enviar las mismas imágenes. Para evitar esto, el cliente guarda en la memoria caché o almacena estas imágenes después de la primera respuesta y, luego, utiliza las imágenes directamente desde la memoria caché. Los servicios web RESTful controlan el almacenamiento en caché mediante el uso de respuestas de la API que se pueden guardar en la memoria caché o no.

### Code-on-demand

O código bajo demanda. El servidor puede extender o personalizar temporalmente la funcionalidad del cliente mediante la transferencia de lógica. Por ejemplo, cuando completa un formulario de inscripción en cualquier sitio web, su navegador resalta de inmediato cualquier error que usted comete, como un número de teléfono incorrecto. El navegador puede hacer esto gracias al código enviado por el servido

