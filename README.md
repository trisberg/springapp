# springapp

POC of a CLI for creating Spring Boot Applications to run on Kubernetes

## Prerequisites:

* a [Kubernetes](https://kubernetes.io/) cluster and the [kubectl CLI](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* [curl](https://curl.haxx.se/) command
* [Java 11 JDK](https://adoptopenjdk.net/installation.html?variant=openjdk11#)
* [Skaffold](https://skaffold.dev/)
* [Docker](https://www.docker.com/)
* [pack CLI](https://buildpacks.io/docs/install-pack/) from the Cloud Native Buildpacks project
* [Mononoke](https://github.com/spring-cloud-incubator/mononoke) installed

> NOTE: This is only a POC with limited testing done in a Bash shell.

## Features

The `springapp` command can initialize Spring Boot application that will run on Kubernetes via Mononoke. 

Here is a `springapp` command example:

```
springapp init test --web
```

Available commands are listed via the help text:

```
$ springapp --help
springapp is for Spring Boot Applications on Kubernetes
version 0.0.1

Commands:
  init         Initialize a Spring Boot Application project
  build        Build a Spring Boot Application container
  run          Run a Spring Boot Application container on Kubernetes
  delete       Delete a Spring Boot Application from Kubernetes

All commands take the project name (which also is the relative path) 
as the first and only argument.

The 'init' command has the following options:

  --web                        Use the 'web' dependency (default)
  --webflux                    Use the 'webflux' dependency
  --jdbc-driver {driver-name}  Add 'jdbc' and the driver dependencies
  --service-type {type}        Expose the service using the type (default is ClusterIP)
```

## Installation

copy the CLI script to your PATH:

```
sudo curl https://raw.githubusercontent.com/trisberg/springapp/master/springapp \
 -o /usr/local/bin/springapp && \
sudo chmod +x /usr/local/bin/springapp
```

## Configure Skaffold

> The `default-repo` should match your Docker ID

Set the default repo for Skaffold:

```
skaffold config set default-repo $USER
```
