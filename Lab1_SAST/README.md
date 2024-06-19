# LAB 1 - SAST

In this lab, we will learn how to use the [SonarQube](https://www.sonarsource.com/products/sonarqube/) tool and the functionalities it incorporates in terms of static code analysis.
We'll also look at how to use the standalone CLI [Semgrep](https://github.com/returntocorp/semgrep) tool.

Requirements:

- [Docker](https://docs.docker.com/)
- [Git](https://git-scm.com/)
- [Postman](https://www.postman.com/) (Optional)

## Project clone

First, we will make a fork and a clone of the project that we will analyse in this practice.
(<https://github.com/bertex/BikeLineViewer-AST>)

![fork repo](./fig/fork_dotnet_repo.png)



```bash
git clone https://github.com/<Usuario>/BikeLaneViewer_AST
````


## SonarQube

```bash
docker network create -d bridge sonar_network
docker volume create sonar_data
docker run -d --name sonarqube -p 9000:9000 -v sonar_data:/opt/sonarqube/data --network sonar_network sonarqube:10.4-community
```

Next, we will open a browser and access 'http://localhost:9000'. Where we should see the SonarQube login screen

![alt text](./fig/login_sonarq.png)

We will access with the username 'admin', password 'admin'. It will ask us to change the password for a different one and we should already see the main dashboard of the application

![Dashboard sonarqube](./fig/ini_sonarq.png)

![](./fig/ini_sonarq.PNG)

### Project creation

First of all, we'll create a new project manually, which will reference our API. By clicking on the 'Create a local project' option

In this example, we'll point you to the project name and key 'bikelane_map_app', defining the 'master' branch as the main branch of the project


![Create project sonarqube](./fig/create_project.png)

Next, we'll instruct you to detect any changes as new code, with the option 'Use the global setting'

If everything went well, we should now see the project view empty

![Empty project sonarqube](./fig/empty_project_sonarq.PNG)

### API Token Generation

To be able to run the 'sonar-scanner' we must generate an analysis token, to do so, we must access the security section and generate a global token, which will allow us to analyse any project

![Generate token sonarq](./fig/generate_token.png)

**Write down the Token in a Notepad!** It will not be displayed again...

### Code analysis

At this point, we can already run the first code scan. We must bear in mind that SonarQube has different specialized scanners, depending on the tool we use to compile the project:
- Gradle
- NET
- Maven
- Ant

and a generic 'SonarScanner'. Depending on the project, we may be interested in using the specific scanner, as it will provide us with more information than the generic one.

For more information you can consult the SonarQube documentation regarding [Scanners](https://docs.sonarsource.com/sonarqube/9.9/analyzing-source-code/scanners/sonarscanner/).

In our case, as the application is programmed in JavaScript with 'NodeJS', we will use the default image of [sonnar-scanner-cli]((https://hub.docker.com/r/sonarsource/sonar-scanner-cli))

We will only have to replace the following parameters:

- **$HOST**: `http://sonarqube:9000` -> (Host local)
- **$TOKEN**: `sqa_*` -> (Authentication token that we obtained in the previous step)
- **$KEY**: `bikelane_map_app` -> (Project ID)
  
```bash
cd BikeLaneViewer_AST
docker run --rm \
    -e SONAR_HOST_URL="$HOST" \
    -e SONAR_SCANNER_OPTS="-Dsonar.projectKey=$KEY" \
    -e SONAR_TOKEN="$TOKEN" \
    -v "${PWD}:/usr/src" --network sonar_network \
    sonarsource/sonar-scanner-cli
```

If everything has gone well, we should see the following output:

```txt
....
INFO: ANALYSIS SUCCESSFUL, you can find the results at: http://sonarqube:9000/dashboard?id=bikelane_map_app
INFO: Note that you will be able to access the updated dashboard once the server has processed the submitted analysis report
INFO: More about the report processing at http://sonarqube:9000/api/ce/task?id=5d2ef6ea-1dbb-47a1-920b-e464e4f1210d
INFO: Analysis total time: 37.405 s
INFO: ------------------------------------------------------------------------
INFO: EXECUTION SUCCESS
INFO: ------------------------------------------------------------------------
INFO: Total time: 43.180s
INFO: Final Memory: 22M/80M
INFO: ------------------------------------------------------------------------
```

As you can see in the dashboard, we have 5 Security Hostspots that we should check to see if they may be security issues.
What problems does this code have?

![Security review](./fig/security_review.png)

## Semgrep

Semgrep is distributed as a CLI tool through PyPI, in our case, we will use the [official Docker image](), thus saving the installation of the tool.

```bash
docker pull returntocorp/semgrep
```

If we run the help command, we should see the 'usage' of the CLI tool

```bash
docker run --rm returntocorp/semgrep semgrep --help
```
```txt
Usage: semgrep [OPTIONS] COMMAND [ARGS]...
```

We run the scan tool in [offline mode](https://semgrep.dev/docs/getting-started/), mapping the contents of the project to the path '/src'

```bash
docker run --rm -v ${PWD}:/src returntocorp/semgrep semgrep scan --config=auto
```

**What vulnerabilities have you found? More or less than SonarQube?**


# Cleaning

Clean up the resources deployed in this workshop:

- Sonarqube

```bash
docker stop sonarqube
docker rm sonarqube
docker network rm sonar_network
docker rmi sonarqube:10.4-community
```

- Semgrep

```bash
docker rmi returntocorp/semgrep
```
