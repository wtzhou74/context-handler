# Context Handler API

The Context Handler API is a RESTful web service component of Consent2Share (C2S). It is responsible for sending XACML response context that includes authorization decisions along with applied policy obligations to Policy Enforcement Point (PEP) API components. The Context Handler sends the XACML request context to the Policy Decision Point (PDP). The PDP evaluates the applicable policies from the Patient Consent Management (PCM) against the request context, and returns the response context that includes authorization decisions along with obligations of applied policies to the Context Handler. The Context Handler sends XACML response context back to PEP component. The PDP uses [HERAS-AF](https://bitbucket.org/herasaf/herasaf-xacml-core/overview), an open source XACML 2.0 implementation, for XACML evaluation and uses a PCM database as a local policy repository to retrieve XACML policies that are generated from patients’ consents.
   

## Build

### Prerequisites

+ [Oracle Java JDK 8 with Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
+ [Docker Engine](https://docs.docker.com/engine/installation/) (for building a Docker image from the project)
+ [Logback-Audit](http://audit.qos.ch/)

### Commands

This is a Maven project and requires [Apache Maven](https://maven.apache.org/) 3.3.3 or greater to build it. It is recommended to use the *Maven Wrapper* scripts provided with this project. *Maven Wrapper* requires an internet connection to download Maven and project dependencies for the very first build.

To build the project, navigate to the folder that contains `pom.xml` file using terminal/command line.

+ To build a JAR:
    + For Windows, run `mvnw.cmd clean install`
    + For *nix systems, run `mvnw clean install`
+ To build a Docker Image (this will create an image with `bhits/context-handler:latest` tag):
    + For Windows, run `mvnw.cmd clean package docker:build`
    + For *nix systems, run `mvnw clean package docker:build`

## Run

### Prerequisites

1. This API uses the PCM database as a local policy repository to retrieve XACML policies. Thus, it needs to have a database user configuration with read privileges to the PCM database. Please see [Configure](#configure) section for details of configuring the data source.
2. This API also needs to call Logback-Audit server, so please following the [Logback-Audit deployment instruction](https://github.com/bhits/logback-audit) to set it up
3. After Logback-Audit server is up, the hostname (currently is localhost) in the [default configuration](context-handler/src/main/resources/application.yml) need to be replaced with the real server name
Logback-Audit configuration section in the configuration file

```yml
...
    audit-service:
      host: localhost
      port: 9630
...
```

### Commands

This is a [Spring Boot](https://projects.spring.io/spring-boot/) project and serves the API via an embedded Tomcat instance. Therefore, there is no need for a separate application server to run this service.
+ Run as a JAR file: `java -jar context-handler-x.x.x-SNAPSHOT.jar <additional program arguments>`
+ Run as a Docker Container: `docker run -d bhits/context-handler:latest <additional program arguments>`

*NOTE: In order for this API to fully function as a microservice in the C2S application, it is required to setup the dependency microservices and support level infrastructure. Please refer to the [C2S Deployment Guide](https://github.com/bhits/consent2share/releases/download/2.0.0/c2s-deployment-guide.pdf) for instructions to setup the C2S infrastructure.*

## Configure

This API runs with some default configuration that is primarily targeted for development environment. However, [Spring Boot](https://projects.spring.io/spring-boot/) supports several methods to override the default configuration to configure the API for a certain deployment environment.

Please see the [default configuration](context-handler/src/main/resources/application.yml) for this API as a guidance and override the specific configuration per environment as needed. Also, please refer to [Spring Boot Externalized Configuration](http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html) documentation to see how Spring Boot applies the order to load the properties and [Spring Boot Common Properties](http://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html) documentation to see the common properties used by Spring Boot.

### Examples for Overriding a Configuration in Spring Boot

#### Override a Configuration Using Program Arguments While Running as a JAR:

+ `java -jar context-handler-x.x.x-SNAPSHOT.jar --server.port=80 `

#### Override a Configuration Using Program Arguments While Running as a Docker Container:

+ `docker run -d bhits/context-handler:latest --server.port=80 `

+ In a `docker-compose.yml`, this can be provided as:
```yml
version: '2'
services:
...
  context-handler.c2s.com:
    image: "bhits/context-handler:latest"
    command: ["--server.port=80"]
...
```
*NOTE: Please note that these additional arguments will be appended to the default `ENTRYPOINT` specified in the `Dockerfile` unless the `ENTRYPOINT` is overridden.*

### Enable SSL

For simplicity in development and testing environments, SSL is **NOT** enabled by default configuration. SSL can easily be enabled following the examples below:

#### Enable SSL While Running as a JAR

+ `java -jar context-handler-x.x.x-SNAPSHOT.jar --spring.profiles.active=ssl --server.ssl.key-store=/path/to/ssl_keystore.keystore --server.ssl.key-store-password=strongkeystorepassword`

#### Enable SSL While Running as a Docker Container

+ `docker run -d -v "/path/on/dockerhost/ssl_keystore.keystore:/path/to/ssl_keystore.keystore" bhits/context-handler:latest --spring.profiles.active=ssl --server.ssl.key-store=/path/to/ssl_keystore.keystore --server.ssl.key-store-password=strongkeystorepassword`
+ In a `docker-compose.yml`, this can be provided as:
```yml
version: '2'
services:
...
  context-handler.c2s.com:
    image: "bhits/context-handler:latest"
    command: ["--spring.profiles.active=ssl","--server.ssl.key-store=/path/to/ssl_keystore.keystore", "--server.ssl.key-store-password=strongkeystorepassword"]
    volumes:
      - /path/on/dockerhost/ssl_keystore.keystore:/path/to/ssl_keystore.keystore
...
```

*NOTE: As seen in the examples above, `/path/to/ssl_keystore.keystore` is made available to the container via a volume mounted from the Docker host running this container.*

### Override Java CA Certificates Store In Docker Environment

Java has a default CA Certificates Store that allows it to trust well-known certificate authorities. For development and testing purposes, one might want to trust additional self-signed certificates. In order to override the default Java CA Certificates Store in a Docker container, one can mount a custom `cacerts` file over the default one in the Docker image as `docker run -d -v "/path/on/dockerhost/to/custom/cacerts:/etc/ssl/certs/java/cacerts" bhits/context-handler:latest`

*NOTE: The `cacerts` references refered to in the volume mapping above are files, not directories.*

[//]: # (## API Documentation)

[//]: # (## Notes)

[//]: # (## Contribute)

## License
View [license](https://github.com/bhits/context-handler/blob/master/LICENSE) information for the software contained in this repository.

## Contact

If you have any questions, comments, or concerns please see [Consent2Share](https://bhits.github.io/consent2share/) project site.

## Report Issues

Please use [GitHub Issues](https://github.com/bhits/context-handler/issues) page to report issues.

[//]: # (License)
