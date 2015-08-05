Your first JAX_RS Client and Server
========================

<!-- toc -->


This project is a simple example showing usage of **@Path**, **@GET**, **@PUT**, **@POST**, and **@PathParam**.  It uses pure streaming output as well. 


System Requirements:
-------------------------
- Maven 3.0.4 or higher



Building the project:
-------------------------

1. In root directory **mvn clean install**


This will build a WAR and run it with embedded Jetty



Source Code Hierarchy:
-------------------------

```
ex03_1
|-- pom.xml
`-- src
    |-- main
    |   |-- java
    |   |   `-- com
    |   |       `-- restfully
    |   |           `-- shop
    |   |               |-- domain
    |   |               |   `-- Customer.java
    |   |               `-- services
    |   |                   |-- CustomerResource.java
    |   |                   `-- ShoppingApplication.java
    |   `-- webapp
    |       `-- WEB-INF
    |           `-- web.xml
    `-- test
        `-- java
            `-- com
                `-- restfully
                    `-- shop
                        `-- test
                            `-- CustomerResourceTest.java
```

***pom.xml***

!CODEFILE "./pom.xml"
