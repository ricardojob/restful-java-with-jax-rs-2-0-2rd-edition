# Injection Annotations (2)


<!-- toc -->


This project is a simple example showing usage of JAX-RS injection annotations.


## System Requirements:


- Maven 2.0.9 or higher


## Building the project:


1. In root directoy **mvn jetty:run**

This will build a WAR and run it with embedded Jetty


Then open browser and go to: http://localhost:8080


Submit form and follow links.



## Source Code


### Hierarchy
```
ex05_2
|-- pom.xml
`-- src
    `-- main
        |-- java
        |   `-- com
        |       `-- restfully
        |           `-- shop
        |               |-- domain
        |               |   `-- Customer.java
        |               `-- services
        |                   |-- CustomerResource.java
        |                   `-- ShoppingApplication.java
        `-- webapp
            |-- WEB-INF
            |   `-- web.xml
            `-- index.html
```


### Details


*pom.xml*

!CODEFILE "./pom.xml"


*src/main/webapp/WEB-INF/web.xml*

!CODEFILE "./src/main/webapp/WEB-INF/web.xml"


*src/main/webapp/index.html*

!CODEFILE "./src/main/webapp/index.html"


*src/main/java/com/restfully/shop/domain/Customer.java*

!CODEFILE "./src/main/java/com/restfully/shop/domain/Customer.java"


*src/main/java/com/restfully/shop/services/CustomerResource.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/CustomerResource.java"


*src/main/java/com/restfully/shop/services/ShoppingApplication.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/ShoppingApplication.java"