#LAB 2 – API Analysis with OWASP ZAP (DAST)

In this lab we will see how to dynamically analyse APIs with the OWASP ZAP tool, applying the rules of [OWASP Top 10](https://owasp.org/www-project-top-ten/).

Requirements:

- [Docker](https://docs.docker.com/)
- [Git](https://git-scm.com/)

## Clone the project

First, we will make a clone of the project to be analysed. As you can see, it's the same project we've seen in [SAST lab 1](.. /Lab1%20-%20SAST/README.md)

``` Bash
git clone https://github.com/bertex/BikeLineViewer-AST

```

## Code Inspection

The project is an interactive map of Barcelona's bike lanes, but it also includes a functionality to register a user via REST API created with 'NodeJS + Express'. A basic 'CRUD' of users with the following endpoints:

```text
POST   /api/users
GET    /api/users
GET    /api/users/{id}
PUT    /api/users/{id}
DELETE /api/users/{id}
```

In addition to an endpoint to obtain OpenData data on bike lanes: 'GET /api/data/bikelanes'


## Running the app

We generate the 'Docker' image from the 'Dockerfile'

```bash
cd BikeLaneViewer_AST
docker build -t bikelanemap:latest .
```

We'll create and run a container based on the image we've generated earlier, mapping the application to port 3000.

```bash
docker run -d --name bikelanemap -p 3000:3000 bikelanemap:latest
```

If everything has worked well, we should find the app lifted on port '3000'. If we run a 'GET' on the '/users' endpoint, it should return an empty array.

```bash
curl --location 'http://localhost:3000/api/users'

[]
```

If you're on Windows and don't have the [cURL](https://curl.se/) tool, you can use the Swagger web interface (http://localhost:3000/api/doc/) to generate HTTP requests.

To be able to add a person, we will have to make a POST to the endpoint '/api/users' with this information:

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

This request should return the following message, with the data of the user created

```json
{
    "username":"Neo",
    "mail":"neo@matrix.net",
    "id":0
}
```


## Automatic scanning with OWASP ZAP

Now that we have the API up and running, we'll run the automatic scanning of OWASP ZAP using the definition of the API in OpenAPI (http://localhost:3000/api/doc/swagger.json).

```bash
docker run --rm -t ghcr.io/zaproxy/zaproxy:stable zap-api-scan.py -t http://host.docker.internal:3000/api/doc/swagger.json -f openapi
```

This command should return the following result

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

As you can see, the app isn't too bad, it generates 12 warnings.

Access customer data via 'Curl'

```bash
curl --location 'http://localhost:3000/api/users'
```
As you can see, all the tests that 'ZAP' has done to verify all the rules it has stored appear.

Next, we'll modify the app to generate more warnings. We will intentionally add a 500 error to the 'GET /api/users' endpoint and include 'stackTrace' information in the response.

We will only have to modify the file './controllers/usersController.js', and we will replace the response method 'GET' on the root endpoint '/' with this:

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

We stop the API service

```bash
docker rm -f bikelanemap
```

We generate the updated Docker image and run the container again

```bash
docker build -t bikelanemap:latest .
docker run -d --name bikelanemap -p 3000:3000 bikelanemap:latest
```

**If we now run the scanner again, we should see some additional errors... Which ones?**

```bash
docker run --rm -t ghcr.io/zaproxy/zaproxy:stable zap-api-scan.py -t http://host.docker.internal:3000/api/doc/swagger.json -f openapi
```
