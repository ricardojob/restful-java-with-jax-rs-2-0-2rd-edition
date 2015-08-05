# Exception Mappers


<!-- toc -->


This project is a simple example of using JAX-RS **ExceptionHandlers**.


## System Requirements:

- Maven 3.0.4 or higher


## Building the project:

1. In root directory **mvn clean install**


This will build a WAR and run it with embedded Jetty.



## Source Code


### Hierarchy
```
ex07_1
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
    |   |                   |-- CustomerNotFoundException.java
    |   |                   |-- CustomerNotFoundExceptionMapper.java
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


*src/main/java/com/restfully/shop/services/CustomerNotFoundException.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/CustomerNotFoundException.java"


*src/main/java/com/restfully/shop/services/CustomerNotFoundExceptionMapper.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/CustomerNotFoundExceptionMapper.java"


*src/test/java/com/restfully/shop/test/CustomerResourceTest.java*

!CODEFILE "./src/test/java/com/restfully/shop/test/CustomerResourceTest.java"