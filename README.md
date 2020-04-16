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
* [Helm v3](https://helm.sh/docs/intro/install/) for installing MySQL

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
version 0.0.2

Commands:
  init         Initialize a Spring Boot Application project
  bind         Bind a service to a Spring Boot Application
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

The 'bind' command has the following options:

  --name                       The name of the service to bind
  --user                       Username for login (defaults to database type default)
  --database                   Database name (defaults to database type default)
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

## EXAMPLE: A database backed app - step-by-step

### Initialize a new project

```
springapp init mytest --jdbc-driver hsql,mysql --service-type LoadBalancer
```

We specify `hsql` and `mysql` as the JDBC drivers and we prefer to get an external IP address so we can access the app.

### Write the app code

Open the controller class using:

```
code mytest mytest/src/main/java/com/example/mytest/MytestController.java
```

Then, modify the code to be:

```
package com.example.mytest;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.support.JdbcUtils;
import org.springframework.jdbc.support.MetaDataAccessException;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.PostConstruct;

@RestController
public class MytestController {

    @Autowired
    JdbcTemplate jdbcTemplate;

    String databaseInfo = "database";
    String databaseUser = "unknown";

    @PostConstruct
    public void init() {
        try {
            databaseInfo = JdbcUtils.extractDatabaseMetaData(
                    jdbcTemplate.getDataSource(), "getDatabaseProductName");
        } catch (MetaDataAccessException e) {}
        if (databaseInfo.toLowerCase().contains("mysql")) {
            databaseUser = jdbcTemplate.queryForObject("select user()", String.class);
        }
        if (databaseInfo.toLowerCase().contains("hsql")) {
            databaseUser = jdbcTemplate.queryForObject("call current_user", String.class);
        }
    }

    @RequestMapping("/")
    public String index() {
        return "Hello " + databaseInfo + " user " + databaseUser;
    }
}
```

On startup we query the database for product name and user to be displayed in the `index` method.

If you run this, the app will use an embedded HSQLDB database and the index method will return:

```
Hello HSQL Database Engine user SA
```

### Bind a MySQL database and deploy the app

We can bind a MySQL database that we create using a Bitnami Helm chart.

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install mydb --namespace default bitnami/mysql
```

Once the database is running we can bind it and deploy the app.

```
springapp bind mytest --name mydb-mysql
springapp run mytest
```

When the app is up and runnning we can query the service for the IP address and curl the index endpoint using:

```
mytest_ip=$(kubectl get --namespace default service/mytest -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
curl ${mytest_ip}/ && echo
```

We should now see a message similar to this:

```
Hello MySQL user root@10.48.0.32
```

### Cleanup

```
springapp delete mytest
kubectl delete secret mydb-mysql-binding
helm delete --namespace default mydb
```

