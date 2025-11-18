# Curso de DevOps (Jenkins)

## Instalaciones

### Docker Desktop

- [WSL 2 feature on Windows](https://learn.microsoft.com/es-es/windows/wsl/install)
- [Docker Desktop](https://www.docker.com/get-started/)

### Configuración de puertos dinámicos en Windows

    netsh int ipv4 set dynamic tcp start=51000 num=14536

### Alternativas a Docker Desktop

- [Podman](https://podman.io/docs/installation)
- [Rancher Desktop](https://rancherdesktop.io/)

### Utilidades

- [Git](https://git-scm.com/)

## Entorno CI/CD

### Crear una red en Docker

    docker network create devops

### Desplegar contenedores

#### Servidor de correo electrónico

    docker run -d --name mailhog -p 1025:1025 -p 8025:8025 mailhog/mailhog

    docker network connect devops mailhog

#### Control de calidad

    docker run -d --name sonarQube --publish 9000:9000 --network devops sonarqube:latest

#### Docker in Docker

    docker run -d --name jenkins-docker-in-docker --privileged ^
        --network devops --network-alias docker --env DOCKER_TLS_CERTDIR=/certs ^
        --volume docker-in-docker-certs:/certs/client ^
        --volume jenkins-data:/var/jenkins_home ^
        --publish 2376:2376 --publish 50088:8080 docker:dind --storage-driver overlay2

#### Jenkins

    docker run --name jenkins-host --network devops --publish 50080:8080 --publish 50000:50000 --volume "%cd%":/externo --volume docker-in-docker-certs:/certs/client:ro --volume jenkins-data:/var/jenkins_home --env DOCKER_CERT_PATH=/certs/client --env DOCKER_HOST=tcp://docker:2376 --env DOCKER_TLS_VERIFY=1 --env JAVA_OPTS="-Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true" jenkins/jenkins:lts

#### Jenkins personalizado

- Descargar Dockerfile: [jenkins-with-docker/Dockerfile](./jenkins-with-docker/Dockerfile)
- Crear imagen

        docker build -t jenkins-with-docker .

- Crear contenedor

        docker run --name jenkins --network devops --publish 50080:8080 --publish 50000:50000 ^
            --volume .:/externo --volume jenkins-data:/var/jenkins_home ^
            --volume docker-in-docker-certs:/certs/client:ro --env DOCKER_CERT_PATH=/certs/client ^
            --env DOCKER_HOST=tcp://docker:2376 --env DOCKER_TLS_VERIFY=1 ^
            --env JAVA_OPTS="-Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true" ^
            jenkins-with-docker

        docker run --name jenkins --network devops --publish 50080:8080 --publish 50000:50000 ^
            --volume .:/externo --volume jenkins-data:/var/jenkins_home ^
            --volume docker-in-docker-certs:/certs/client:ro --env DOCKER_CERT_PATH=/certs/client ^
            --env DOCKER_HOST=tcp://docker:2376 --env DOCKER_TLS_VERIFY=1 ^
            --env JAVA_OPTS="-Dhudson.plugins.git.GitSCM.ALLOW_LOCAL_CHECKOUT=true" ^
            jamarton/jenkins-whit-docker

## Kata

- [GildedRose Kata: Requisitos (es)](https://github.com/emilybache/GildedRose-Refactoring-Kata/blob/main/GildedRoseRequirements_es.md)

## Repositorios

- <https://github.com/jenkins-docs/simple-java-maven-app>
- <https://github.com/jenkins-docs/simple-node-js-react-npm-app>
- <https://github.com/jenkins-docs/quickstart-tutorials>
- <https://github.com/jmagit/demos-devops>
- <https://github.com/jmagit/demos-frontend-angular>
- <https://github.com/jmagit/MOCKWebServer>
- <https://github.com/spring-projects/spring-petclinic>

## Comandos

    docker run --rm -it --name maven -v .:/local --workdir /local -v maven-repository:/root/.m2 maven:3.8.6-eclipse-temurin-8 bash

### Exportación e importación

#### Exportación

java -jar jenkins-cli.jar -s http://***usr:xxxxxxx@localhost***:50080/ get-job ***job-name **>** job-name.xml***

#### Importación

java -jar jenkins-cli.jar -s http://usr:xxxxxxx@localhost:50080/ create-job ***job-name **<** job-name.xml***
