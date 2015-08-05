# Injection Annotations (1)


<!-- toc -->


This project is a simple example showing usage of JAX-RS injection annotations.


## System Requirements:


- Maven 3.0.4 or higher


## Building the project:


1. In root directory **mvn clean install**


This will build a WAR and run it with embedded Jetty.


## Source Code


### Hierarchy
```
ex05_1
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
    |   |                   |-- CarResource.java
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
                            `-- InjectionTest.java
```


### Details


*pom.xml*

!CODEFILE "./pom.xml"


*src/main/webapp/WEB-INF/web.xml*

!CODEFILE "./src/main/webapp/WEB-INF/web.xml"


*src/main/java/com/restfully/shop/domain/Customer.java*

!CODEFILE "./src/main/java/com/restfully/shop/domain/Customer.java"


*src/main/java/com/restfully/shop/services/CustomerResource.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/CustomerResource.java"


*src/main/java/com/restfully/shop/services/ShoppingApplication.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/ShoppingApplication.java"


*src/main/java/com/restfully/shop/services/CarResource.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/CarResource.java"


*src/test/java/com/restfully/shop/test/CustomerResourceTest.java*

!CODEFILE "./src/test/java/com/restfully/shop/test/CustomerResourceTest.java"