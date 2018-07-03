# Estandares para servicios REST

# Tabla de contenidos

- [Lineamientos](#lineamientos)
- [Definición](#definición)
  * [Métodos Soportados](#métodos-soportados)
  * [Problemas Comunes](#problemas-comunes)
  * [Estado de URL](#estado-de-url)
  * [Estructura de URL](#estructura-de-url)
  * [Ambiente y Versionamiento](#ambiente-y-versionamiento)
- [Respuestas](#respuestas)
  * [Objetos](#objetos)
    + [Elemento u objeto](#elemento-u-objeto)
    + [Colección de objetos](#colección-de-objetos)
    + [Gran colección de datos](#gran-colección-de-datos)
    + [Agregar colecciones de datos](#agregar-colecciones-de-datos)
    + [Orden de resultados](#orden-de-resultados)
    + [Filtrar resultados](#filtrar-resultados)
    + [Obtener campos específicos](#obtener-campos-específicos)
    + [Paginación](#paginación)
        * [Paginación parcial o de servidor](#paginación-parcial-o-de-servidor)
        * [Paginación total o del lado del cliente](#Paginación-total-o-del-lado-del-cliente)
- [Manejo de errores](#manejo-de-errores)
- [Seguridad](#seguridad)
- [Mejores prácticas](#mejores-prácticas)
- [Ejemplos](#ejemplos)


<!-- toc -->

## Lineamientos

Este documento contiene los lineamientos y estándares para mantener consistencia, mantenibilidad y mejores prácticas a través de los servicios web.

El beneficio de este documento es proveer consistencia entre los diferentes equipos y promover un punto central de documentación y decisiones en diseño.

El contenido de este documento puede ser modificado tantas veces sea necesaria para adaptar cualquier cambio de implementación, mejores prácticas, pero siempre manteniendo consistencia.

## Definición

REST define un conjunto de principios de arquitectura con los cuales se puede realizar servicios web enfocados en recursos del sistema, incluyendo en como obtener y transferir información por HTTP a un gran rango de clientes, los cuales están definidos en múltiples lenguajes de programación.

Una implementación REST debe seguir los siguientes principios:

* Usar métodos HTTP (GET, POST, PUT, DELETE, HEAD, OPTIONS)
* No poseer estado.
* Utilizar una estructura de directorios como URLs
* Transferir objetos XML, JSON o ambos.


### Métodos Soportados

Una implementación REST establece una conexión uno a uno entre operaciones y los métodos HTTP, siempre respetando su idempotencia* de acuerdo a la siguiente tabla

Método | Descripción | HTTP Status | Idenpotencia
------ | ----------- | ----------- | ------------ 
GET | Obtener un recurso | 200 | Si
POST | Crear uno o muchos objetos en base a los parámetros | 200 201(header location) | No
PUT | Reemplazar un objeto completo | 200 | Si 
DELETE | Remover o deshabilitar un recurso | 200 | Si
HEAD | Obtener metadata de una petición GET. Se utiliza para evitar grandes operaciones de métodos GET | 200 | Si
PATCH | Aplicar una actualización parcial de un objeto. Puede utilizarse como un UPSERT, Actualizar en caso de que exista el objeto y crear en caso de que no exista el objeto | 200 | No
OPTIONS | Obtener información de una petición | 200 | Si

*Idempotencia: Múltiples y sucesivas solicitudes generan el mismo resultado.

Referencia: https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html

### Problemas comunes

Un problema que sucede comúnmente al definir un servicio web es utilizar un método HTTP para motivos indebidos. Por ejemplo un método GET identifica un recurso y puede contener un conjunto de parámetros por “querystring” que definen un criterio de búsqueda de un recurso. En muchos casos se utiliza un método HTTP GET para realizar una transacción de agregar o modificar un recurso, en estos casos se está utilizando erróneamente el método GET y un URL de manera que no representa la metodología RESTful.

```bash
GET /adduser?name=Robert HTTP/1.1
```

El ejemplo de arriba se utiliza una operación de cambio de estado utilizando un método GET. El principal problema puede ser principalmente semántica, pero los métodos HTTP GET están hechos para obtener recursos utilizando los parámetros como criterio de filtro y no para agregar recursos en base de datos. En conclusión de acuerdo al protocolo HTTP/1.1-compliant servicios utilizándolos de está manera son inconsistentes.

Otro problema no despreciable, “search engines”(crawlers) podrían hacer cambios con el simple hecho de tener acceso al enlace URL.

```bash
POST /users HTTP/1.1
Host: myserver
Content-Type: application/xml
<?xml version="1.0"?>
<user>
<name>Robert</name>
</user>
```

Hay que destacar que para una mejor representación de los servicios REST, específicamente los URI’s es lógico utilizar recursos como entidades (adjetivos) en vez de verbos, como por ejemplo /users y sus respectivas acciones como métodos HTTP, debido a que estas acciones ya se encuentran definidas en el protocolo. Idealmente para mantener  las operaciones de manera explícita, los servicios web no deben definir acciones como verbos, como por ejemplo /addUsers ó /updateUser. Este principio también aplica para los parámetros del body de una petición HTTP, la cual está definida para transportar solo estados de los recursos y no el nombre del método o acción a ejecutar.

### Estado de url

Un recurso no debe mantener ningún tipo de estado entre peticiones. Cada petición debe ser independiente de otra. Una arquitectura de Microservicios y Nanoservicios se adapta perfectamente a esta definición.

### Estructura de url

Los URI de servicios REST deben ser intuitivos y fácil de entender o adivinar su función. Un URI debería ser como un documento  que no requiere ninguna explicación o referencia para un desarrollador la relación con un recurso, por esto mismo un URI debe ser predecible y fácil de entender.

Una manera de lograr este nivel de usabilidad es el de definir URI’s como estructura de directorios. Está estructura es jerárquica, con una única raíz “root path” o punto de entrada y se ramifica para crear “subpath” que exponen áreas específicas de un servicio. De acuerdo a esto un servicio no es solo adjetivos divididos por barras si no un árbol con nodos. Por ejemplo

> http://www.example.com/discussion/topics/{topic}

La raíz /discussion contiene topics como un subconjunto. Debajo de este conjunto hay nombres de topics y cada uno de estos apuntan a una discusión específica. Con está estructura fácilmente se puede obtener discusiones sólo con proporcionar el parámetro luego de /topics/.

Una manera de definir la estructura de un URI’s es la siguiente:

* Ocultar extensión de archivos (.jsp, .php, .asp)
* Mantener nombre en minúscula
* Sustituir espacios por guiones o guión bajo.
* Evitar “query string”
* Si el recurso no existe “404”, siempre proporcionar una página default como respuesta (Página web)
* Definir adjetivos en vez de verbos
* Para que sea un estándar deben definirse el plural

### Ambiente y versionamiento

Un API puede tener identificadores estables para manejar diferentes ambientes o versiones. Un ambiente puede definirse como

> http://api.example.com/qa/store - Ambiente de pruebas, solo acceso interno

> http://api.example.com/prod/store - Ambiente final de clientes

Se pueden definir tantos ambientes sean necesarios al igual que el versionamiento. Este último específicamente para mantener soporte a cambios en un mismo ambiente y no necesariamente deben existir en todos los ambientes, como por ejemplo

> http://api.example.com/qa/V1/store 

> http://api.example.com/prod/V2/store 

Para ver más información: https://www.ibm.com/developerworks/webservices/library/ws-restful/

## Respuestas

Un servicio representa el estado actual de un recurso y sus atributos en el momento de una petición. Los MIME types mas comunes en servicios RESTful son

JSON(application/json)

XML(application/xml)

XHTML(application/xhtml+xml)

Esto permite utilizar diferentes clientes en diferentes plataformas y dispositivos y consumir los servicios de manera común utilizando MIME types y HTTP Accept header.

Copec define la respuesta de sus servicios como un objeto JSON, no es obligatorio que los clientes utilicen el Header Accept para definir el tipo de respuesta que desea recibir.

### Objetos

Elemento u objeto

Un objeto en una respuesta ya sea colección de elementos o un elemento único debe poseer siempre un identificador único clave/valor representado como “id” preferiblemente o una clave distinta solo para casos especiales donde el objeto ya posea una clave única. Esta clave también puede ser utilizada como llave del objeto.

Objeto JSON
```JSON
{
   "code":"CL",
   "name":"Chile",
   "default_timezone":"Pacific/Easter",
   "timezone":[
       {
           "id":"Pacific/Easter",
           "name":"(GMT-06:00) Easter Island"
       },
       {
           "id":"America/Santiago",
           "name":"(GMT-04:00) Santiago"
       }
   ]
}
```
Colección de objetos
```JSON
{
  "value":[
    { "id": "Item 1","price": 99.95,"sizes": null},
    { … },
    { … },
    { "id": "Item 99","price": 59.99,"sizes": null}
  ],
  "@next": "12734"
}
```

### Gran colección de datos

Cuando un servicio devuelve una colección de objetos muy grande, es importante poder paginar los resultados de manera óptima. Para más detalles, ver la referencia a paginación.

### Agregar colecciones de datos

Debido a que el método POST es idempotente, se pueden crear tantos objetos como se le pase al recurso, diferente que una solicitud PUT que requiere el identificador exacto del objeto para ser actualizado.

### Orden de resultados

El orden se puede definir con el parámetro “orderBy” y debe poseer uno o muchos valores separados por coma con sea necesario. Si se desea ordenar de manera ascendente o descendente se puede utilizar tanto el sufijo “asc” y “desc” respectivamente o los símbolos “+”, “-”.

```bash
GET https://api.example.com/v1/people?$orderBy=+name,email
```

### Filtrar resultados

Se pueden filtrar un resultado de igual manera utilizando el parámetro “filter” y las operaciones siguientes

Operador | Descripción | Ejemplo
-------- | ----------- | -------
Operadores de comparación
eq | Equal - Igual | city eq ’Santiago’
ne | Not Equal - No igual | city ne ’Lima’
gt | Greater than - Mayor que | price gt 20
ge | Greater than or equal - Mayor que o igual | price ge 10
lt | Less than - menor que | price lt 20
le | Less than or equal - menor que o igual | price le 200
Operadores lógicos
and | price le 200 and price gt 10
or | price le 10 and price gt 200
not | not price le 3.1
Operador de agrupación
() | Precedencia | (priority eq 1 or city eq ‘Redmond)’ and price gt 100

Ejemplos

```bash
GET https://api.example.com/v1.0/products?$filter=name eq 'Milk'
GET https://api.example.com/v1.0/products?$filter=name eq 'Milk' and price lt 2.55
GET https://api.example.com/v1.0/products?$filter=(name eq 'Milk' or name eq 'Eggs') and price lt 2.55
```

### Obtener campos específicos

Si se desea obtener campos específicos únicamente, se puede incluir los campos de retorno que debe devolver el servicio. El campo “id” como identificador único siempre debe estar incluido y de igual manera se pueden definir campos obligatorios que no pueden ser removidos de la respuesta de un servicio.

```bash
GET https://api.example.com/v1/people?$fields=name,email
```

Una actualización a este estandard es el uso de GraphQL para filtro de parametros

### Paginación

Al devolver colecciones de datos, siempre es recomendable solo retornar datos parciales en forma de páginas finitas. Se pueden utilizar dos tipos de paginación dependiendo del caso, siempre teniendo en cuenta la optimización de consultas.

#### Paginación parcial o de servidor

La paginación retorna un resultado parcial, e incluye un token de continuación  que refleja el inicio de la próxima página. Si este token no existe, quiere decir que no existen más resultados.

En base de datos relacionales, se utiliza este método para optimizar las consultas. Se deja de utilizar consultas que incluyan “offset” que pueden ser realmente peligrosos en bases de datos muy grandes, se realizan consultas parciales con base en un elemento, y se continúa retornando resultados sucesivamente de acuerdo a un elemento “pivote”. Este método tiene su lado negativo.

Se pierde la opción de saltar entre páginas, 1, 2, …, N, N+1 (Es necesario?)

No se puede regresar, o al menos la implementación puede ser más compleja.

Si se agregan nuevos registros es posible que el resultado de una página cambie

```JSON
{
  "value":[
    { "id": "Item 1","price": 99.95,"sizes": null},
    { … },
    { … },
    { "id": "Item 99","price": 59.99,"sizes": null}
  ],
  "@next": "12734"
}
```

#### Paginación total o del lado del cliente

El servicio proporciona el total de registros, y el cliente puede utilizar parámetros como Limit y offset de acuerdo al total de registros. Todo el manejo de la paginación queda del lado del cliente, y el mismo puede recibir diferente cantidad de resultados de acuerdo a los parámetros que envía.

Importante:

Ordenamiento: En ambos casos, el servicio siempre debe tener un orden estable al devolver los resultados. El criterio de ordenamiento debe estar bien definido para que los resultados sean consistentes.

Resultados repetidos o perdidos: Los resultados podrían estar repetidos o no encontrados debido a la creación o eliminación de los mismos. El cliente debe estar preparado para estos casos.

Tamaño de página: El cliente puede solicitar cualquier tamaño de página siempre que sea menor al Tamaño máximo del servicio. El servicio debe proveer siempre la posibilidad de traer diferentes tamaños de páginas de acuerdo a los requerimientos de los clientes.

Total de registros: Se recomienda tener la cuenta del total de registros en otra tabla de resumen y no realizar consultas de “Count” a la base de datos. En dado caso que sea necesario que el cliente obtenga el total de los registros, se debe incluir el parámetro count=true, para que el servidor incluya este valor en el resultado.

```json
{
  "value":[
    { "id": "Item 1","price": 99.95,"sizes": null},
    { … },
    { … },
    { "id": "Item 99","price": 59.99,"sizes": null}
  ],
  "@offset": "1000",
  "@limit": 100,
  "@total": "10000"
}
```

## Manejo de errores

> En progreso

Todo servicio debe proveer una respuesta de error de acuerdo a los estados HTTP a partir de 4xx, indicando que la solicitud no puede ser cumplir. Si no existe un código de error apropiado de acuerdo a la solicitud, se debe devolver siempre un código 400 “Bad request”, y una respuesta donde el cliente tenga una guía de porque sucede el error. Un servicio debe incluir suficiente detalle para que un desarrollador y el cliente tengan conocimiento de que sucedió y cómo puede resolver el problema.

Ejemplos:

Error de validación

```json
{
  "code": "Bad Request",
  "message": "Previous passwords may not be reused",
  "error": {
    "errors": [
      {
        "code": "ErrorValue",
        "field": "Password",
        "message": "Password not meet the policy",
        "innererror": {
            "code": "PasswordDoesNotMeetPolicy",
            "minLength": "6",
            "maxLength": "64",
            "characterTypes": ["lowerCase","upperCase","number","symbol"],
            "minDistinctCharacterTypes": "2"
        }
      }
    ]
  }
}
```

Error de parámetros

```json
{
  "code": "BadRequest",
  "message": "Multiple errors in ContactInfo data",
  "error": {
    "errors": [
      {
        "code": "NullValue",
        "field": "PhoneNumber",
        "message": "Phone number must not be null"
      },
      {
        "code": "NullValue",
        "field": "LastName",
        "message": "Last name must not be null"
      },
      {
        "code": "MalformedValue",
        "field": "Address",
        "message": "Address is not valid"
      }
    ]
  }
}
```

Referencia de códigos HTTP: https://httpstatuses.com

## Seguridad

## Mejores prácticas

## Ejemplos


