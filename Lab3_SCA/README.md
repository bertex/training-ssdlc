#LAB 3 – SCA

In this lab, we'll learn how to use the [Trivy](https://trivy.dev/) tool to list and analyse dependencies on remote repositories.

Requirements:

- [Docker](https://docs.docker.com/)
- [Git](https://git-scm.com/)

This open-source tool is one of the most popular security scanners.

- Objectives (which can be analysed by trivy):

  - Container images
  - File systems
  - Git repositories (remote)
  - Virtual machine images
  -Kubernetes manifests
  - AWS...

- Scanners (which you can find trivy):
  - Operating system packages and software-in-use dependencies (SBOMs)
  - Known vulnerabilities (CVEs)
  - IaC issues and misconfigurations
  - Sensitive information and secrets
  - Software licenses

## Trivy Installation

Trivy is available in the most commonly used package repositories and is only available for Linux. Given this limitation, and to avoid installing packages, we'll use the [official Docker image](https://hub.docker.com/r/aquasec/trivy/) to run the tool.

```bash
docker pull aquasec/trivy:0.50.2
```

## BikeLaneMap

In this practice, we will use the ['BikelaneMap'](https://github.com/TheMatrix97/BikeLaneViewer_AST) project, which we have seen in the previous labs. It is a web application written in 'NodeJS' using the ['Express'](https://www.npmjs.com/package/express) framework. 

- You can clone the project locally, if you haven't already done so

```bash
git clone https://github.com/bertex/BikeLineViewer-AST.git
```

Our colleague has told us that he has a new functionality of the application ready. These changes have been left in a separate branch called 'new_feature'. We will check out that branch to download its changes locally

```bash
git fetch origin
```

```bash
git checkout -b new_feature origin/new_feature
```

We will generate the docker image locally to verify that the generation process works well

```bash
docker build -t bikelanemap:latest .
```


## SCA with Trivy

We want to put this application into production, to include the new functionalities.
But, first of all, we will have to analyse the Docker image, reviewing possible vulnerabilities of dependencies or configuration errors.

To do this, we'll create a temporary Docker container to run the tool. Previously, we will generate a Docker volume so that Trivy can create a cache of the vulnerability database, and speed up subsequent executions.

```bash
docker volume create trivy_cache
```

```bash
docker run --rm -v //var/run/docker.sock:/var/run/docker.sock -v trivy_cache:/root/.cache/ aquasec/trivy:0.50.2 image bikelanemap:latest
```

**ALL:** Identify the different vulnerabilities in the image and where they are located.

<details>
<summary>Hint</summary>

- Secrets

- NPM Vulnerabilities (package.json)
</details>


**ALL:** Try to fix all the vulnerabilities you've found. *(No need to touch the source code .js or .html!!) *, Project configuration only
