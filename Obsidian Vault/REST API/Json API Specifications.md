JSON API es una especificación de cómo un cliente debe solicitar que se obtengan o modifiquen recursos, y cómo un servidor debe responder a esas solicitudes.

Estos son un conjunto de reglas descritas para diseñar API REST que, cuando se cumplen, pueden hacer que su API REST sea una API JSON compatible. No entraría en las especificaciones completas aquí, como ya se describe en el [sitio oficial](https://jsonapi.org/format/) .

## Cómo las API Json pueden ayudar a diseñar API REST escalables y mantenibles

Las especificaciones de la API JSON detallan el conjunto de reglas, pero me gustaría enumerar algunas características que se necesitan con mucha frecuencia en casi todas las API y cómo la especificación de la API JSON ayuda en estas:

#### Negociación de contenido

Describe las responsabilidades del Cliente y del Servidor o de ambos para negociar el contenido que se solicitará/devolverá. Estas son solo algunas de esas responsabilidades:

+ Clientes y servidores _**deben**_ enviar todos los payloads JSON:API usando el encabezado `Content-type`. 
+ Clientes y servidores _**deben**_ especificar el parámetro `ext` en el encabezado `Content-type` cuando apliquen uno o mas extensiones a el documento JSON:API. 
+ Clientes y servidores _**deben**_ especificar el parámetro `profile` en el encabezado `Content-type` cuando apliquen uno o mas perfiles a el documento JSON:API. 

Las extensiones proporcionan un medio para "extender" la especificación base.

Los perfiles proporcionan un medio para compartir un uso particular de la especificación entre implementaciones.

El tipo de medio JSON:API **NO DEBE** especificarse con ningún parámetro de tipo de medio distinto de `ext`y `profile`.

Ejemplos: 

```http
HTTP/1.1 200 OK 
Content-Type: application/vnd.api+json; ext="https://jsonapi.org/ext/version"
 
{ "type": "articles",
"id": "1",
"version:id": "42",
"attributes": {
"title": "Rails is Omakase" }}
```

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.api+json; profile="https://example.com/resource-timestamps" 

{ "type": "articles",
"id": "1",
"attributes": {
"title": "Rails is Omakase",
"timestamps": {
"created": "2020-07-21T12:09:00Z",
"modified": "2020-07-30T10:19:01Z" } } }
```

### **Esquema/estructura uniforme del documento.**

Un objeto JSON **DEBE** estar en la raíz de cada documento de solicitud y respuesta JSON:API que contenga datos. Este objeto define el “nivel superior” de un documento.

> Un documento **DEBE** contener al menos uno de los siguientes miembros de alto nivel:
> 
> `data`: los “datos primarios” del documento.
> 
> `errors`: una matriz de [objetos de error](https://jsonapi.org/format/#errors) .
> 
> `meta`: un [metaobjeto](https://jsonapi.org/format/#document-meta) que contiene metainformación no estándar.
> 
> un miembro definido por una [extensión](https://jsonapi.org/format/#extensions) aplicada .
> 
> Los integrantes `data`y `errors` **NO DEBEN** coexistir en un mismo documento.
> 
> Un documento **PUEDE** contener cualquiera de estos miembros de alto nivel:
> 
> `jsonapi`: un objeto que describe la implementación del servidor.
> 
> `links`: un [objeto de enlaces](https://jsonapi.org/format/#document-links) relacionado con los datos primarios.
> 
> `included`: una matriz de [objetos de recursos](https://jsonapi.org/format/#document-resource-objects) que están relacionados con los datos primarios y/o entre sí (“recursos incluidos”).