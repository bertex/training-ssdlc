# LAB 3 - SCA

En este laboratorio, aprenderemos a usar la herramienta [Trivy](https://trivy.dev/) para listar y analizar dependencias de repositorios remotos.

Requisitos:

- [Docker](https://docs.docker.com/)
- [Git](https://git-scm.com/)

Esta herramienta de código abierto es uno de los escáneres de seguridad más populares.

- Objetivos (que puede analizar trivy):

  - Imágenes de contenedores
  - Sistemas de ficheros
  - Repositorios Git (remoto)
  - Imágenes de máquinas virtuales
  - Manifiestos Kubernetes
  - AWS...

- Escáneres (que puede encontrar trivy):
  - Paquetes del sistema operativo y dependencias de software en uso (SBOM)
  - Vulnerabilidades conocidas (CVEs)
  - Problemas de IaC y configuraciones incorrectas
  - Información sensible y secretos
  - Licencias de software

## Instalación Trivy

Trivy está disponible en los repositorios de paquetes más utilizados y solo está disponible para Linux. Dada esta limitación, y para evitar la instalación de paquetes, usaremos la [imagen Docker oficial](https://hub.docker.com/r/aquasec/trivy/) para ejecutar la herramienta.

```bash
docker pull aquasec/trivy:0.50.2
```

## BikeLaneMap

En esta práctica, utilizaremos el proyecto [`BikelaneMap`](https://github.com/TheMatrix97/BikeLaneViewer_AST), que hemos visto en los anteriores laboratorios. Se trata de una aplicación web escrita en `NodeJS` utilizando el framework [`Express`](https://www.npmjs.com/package/express). 

- Puedes clonar el proyecto en local, si no lo has hecho ya

```bash
git clone https://github.com/TheMatrix97/BikeLaneViewer_AST.git
```

Nuestro compañero nos ha avisado de que ya tiene lista una nueva funcionalidad de la aplicación. Estos cambios los ha dejado en una rama aparte llamada `new_feature`. Haremos un `checkout` de esa rama para descargar sus cambios en local

```bash
git fetch origin
```

```bash
git checkout -b new_feature origin/new_feature
```

Generaremos la imagen docker en local para verificar que el proceso de generación funciona bien

```bash
docker build -t bikelanemap:latest .
```


## SCA con Trivy

Queremos subir a producción esta aplicación, para incluir las nuevas funcionalidades.
Pero, antes que nada, tendremos que analizar la imagen Docker, revisando posibles vulnerabilidades de las dependencias o errores de configuración.

Para ello, crearemos un contenedor Docker temporal para ejecutar la herramienta. Previamente, generaremos un volumen Docker para que Trivy pueda crear una caché de la base de datos de vulnerabilidades, y acelerar posteriores ejecuciones.

```bash
docker volume create trivy_cache
```

```bash
docker run --rm -v //var/run/docker.sock:/var/run/docker.sock -v trivy_cache:/root/.cache/ aquasec/trivy:0.50.2 image bikelanemap:latest
```

**TODO:** Identifica las diferentes vulnerabilidades que tiene la imagen y dónde se encuentran.

<details>
<summary>Pista</summary>

- Secretos

- Vulnerabilidades NPM (package.json)
</details>


**TODO:** Intenta solucionar todas las vulnerabilidades que has encontrado. *(No hace falta tocar el código fuente .js ni .html!!)*, solamente la configuración del proyecto
