# LAB 2.1 - Análisis de APIs con OWASP ZAP (DAST)

En este laboratorio veremos cómo analizar dinámicamente APIs con la herramienta OWASP ZAP, aplicando las reglas del [OWASP Top 10](https://owasp.org/www-project-top-ten/).

Requisitos:

- [Docker](https://docs.docker.com/)
- [Git](https://git-scm.com/)

## Clonar el proyecto

Primero, haremos un clon del proyecto a analizar. Como puedes ver, es el mismo proyecto que hemos visto en el [laboratorio 1 de SAST](../Lab1%20-%20SAST/README.md)

```bash
git clone https://github.com/TheMatrix97/BikeLaneViewer_AST
```

## Inspección del código

El proyecto es un Mapa interactivo de los carriles bici de Barcelona, pero además, incluye una funcionalidad para dar de alta un usuario vía API REST creada con `NodeJS + Express`. Un `CRUD` básico de usuarios con los siguientes endpoints:

```text
POST   /api/users
GET    /api/users
GET    /api/users/{id}
PUT    /api/users/{id}
DELETE /api/users/{id}
```

Además de un endpoint para obtener los datos de OpenData sobre los carriles bici: `GET /api/data/bikelanes`


## Ejecución de la aplicación

Generamos la imagen `Docker` a partir del `Dockerfile`

```bash
cd BikeLaneViewer_AST
docker build -t bikelanemap:latest .
```

Crearemos y ejecutaremos un contenedor basado en la imagen que hemos generado anteriormente, mapeando la aplicación al puerto 3000.

```bash
docker run -d --name bikelanemap -p 3000:3000 bikelanemap:latest
```

Si todo ha funcionado bien, deberíamos encontrar la aplicación levantada en el puerto `3000`. Si ejecutamos un `GET` en el endpoint `/users`, debería devolvernos un array vacío.

```bash
curl --location 'http://localhost:3000/api/users'

[]
```

Si estás en Windows y no dispones de la herramienta [cURL](https://curl.se/), puedes usar la interfaz web de Swagger (http://localhost:3000/api/doc/) para generar las peticiones HTTP.

Para poder añadir una persona, tendremos que hacer un POST al endpoint `/api/users` con esta información:

```json
{
  "username": "Neo",
  "mail": "neo@matrix.net"
}
```

```bash
curl --location 'http://localhost:3000/api/users' \
--header 'Content-Type: application/json' \
--data-raw '{
  "username": "Neo",
  "mail": "neo@matrix.net"
}'
````

Esta petición debería devolvernos el siguiente mensaje, con los datos del usuario creado

```json
{
    "username":"Neo",
    "mail":"neo@matrix.net",
    "id":0
}
```


## Escaneo automático con OWASP ZAP

Ahora que tenemos la API en marcha, ejecutaremos el escaneo automático de OWASP ZAP utilizando la definición de la misma en OpenAPI (http://localhost:3000/api/doc/swagger.json).

```bash
docker run --rm -t ghcr.io/zaproxy/zaproxy:stable zap-api-scan.py -t http://host.docker.internal:3000/api/doc/swagger.json -f openapi
```

Este comando debería devolvernos el siguiente resultado

```text
....
PASS: Loosely Scoped Cookie [90033]
PASS: Server Side Template Injection [90035]
PASS: Server Side Template Injection (Blind) [90036]
WARN-NEW: Unexpected Content-Type was returned [100001] x 251 
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/data/search?term=term (200 OK)
        http://host.docker.internal:3000/7797490179450435410 (200 OK)
WARN-NEW: Cookie No HttpOnly Flag [10010] x 8 
        http://host.docker.internal:3000/api/users/ (200 OK)
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/users/ (201 Created)
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/users/id (404 Not Found)
WARN-NEW: Missing Anti-clickjacking Header [10020] x 1 
        http://host.docker.internal:3000/api/data/search?term=term (200 OK)
WARN-NEW: X-Content-Type-Options Header Missing [10021] x 5 
        http://host.docker.internal:3000/api/users/ (201 Created)
        http://host.docker.internal:3000/api/users/ (200 OK)
        http://host.docker.internal:3000/api/doc/swagger.json (200 OK)
        http://host.docker.internal:3000/api/data/search?term=term (200 OK)
        http://host.docker.internal:3000/api/data/bikelanes (200 OK)
WARN-NEW: Information Disclosure - Debug Error Messages [10023] x 1 
        http://host.docker.internal:3000/api/doc/swagger.json (200 OK)
WARN-NEW: Server Leaks Information via "X-Powered-By" HTTP Response Header Field(s) [10037] x 8 
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/users/ (200 OK)
        http://host.docker.internal:3000/api/users/ (201 Created)
WARN-NEW: Content Security Policy (CSP) Header Not Set [10038] x 4 
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/data/search?term=term (200 OK)
WARN-NEW: Cookie without SameSite Attribute [10054] x 8 
        http://host.docker.internal:3000/api/users/ (200 OK)
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/users/ (201 Created)
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/users/id (404 Not Found)
WARN-NEW: Permissions Policy Header Not Set [10063] x 4 
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/users/id (404 Not Found)
        http://host.docker.internal:3000/api/data/search?term=term (200 OK)
WARN-NEW: Insufficient Site Isolation Against Spectre Vulnerability [90004] x 1 
        http://host.docker.internal:3000/api/users/ (201 Created)
WARN-NEW: Application Error Disclosure [90022] x 1 
        http://host.docker.internal:3000/api/doc/swagger.json (200 OK)
WARN-NEW: Cloud Metadata Potentially Exposed [90034] x 1 
        http://host.docker.internal:3000/latest/meta-data/ (200 OK)
FAIL-NEW: 0     FAIL-INPROG: 0  WARN-NEW: 12    WARN-INPROG: 0  INFO: 0 IGNORE: 0       PASS: 100
```

Como puedes ver, la aplicación no está tan mal, genera 12 advertencias.

Accede a los datos de los clientes via `Curl`

```bash
curl --location 'http://localhost:3000/api/users'
```
Como puedes observar, aparecen todas las pruebas que ha hecho `ZAP` para verificar todas las reglas que tiene almacenadas.

A continuación, modificaremos la aplicación para generar más advertencias. Intencionadamente, añadiremos un error 500 en el endpoint `GET /api/users` e incluiremos información del `stackTrace` en la respuesta.

Solo tendremos que modificar el archivo `./controllers/usersController.js`, y sustituiremos el método de respuesta `GET` sobre el endpoint raiz `/` por este:

```js
// Route to get all users
router.get('/', (req, res) => {
    //Lanza Error
    throw new Error("¡Ups, algo ha salido mal!");
    res.json(users);
    /*
    #swagger.responses[200] = {
        description: 'Array of users',
        content: {
            'application/json': {
                schema: {
                    type: 'array',
                    items: {$ref: '#/components/schemas/userSchema'}
                }
            }
        }
    }
    */
});
```

Detenemos el servicio de la API

```bash
docker rm -f bikelanemap
```

Generamos la imagen Docker actualizada y ejecutamos el contenedor de nuevo

```bash
docker build -t bikelanemap:latest .
docker run -d --name bikelanemap -p 3000:3000 bikelanemap:latest
```

**Si ahora volvemos a ejecutar el escáner, deberíamos ver algunos errores adicionales... ¿Cuáles?**

```bash
docker run --rm -t ghcr.io/zaproxy/zaproxy:stable zap-api-scan.py -t http://host.docker.internal:3000/api/doc/swagger.json -f openapi
```
