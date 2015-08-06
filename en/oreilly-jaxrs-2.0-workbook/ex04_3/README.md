# Locators and Subresources


<!-- toc -->


Shows an example of **Locators** and **Subresources**.


## System Requirements:


- Maven 3.0.4 or higher



## Building the project:


1. In root directory **mvn clean install**


This will build a WAR and run it with embedded Jetty.



## Source Code


### Hierarchy
```
ex04_3
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
    |   |                   |-- CustomerDatabaseResource.java
    |   |                   |-- CustomerResource.java
    |   |                   |-- FirstLastCustomerResource.java
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


*src/main/java/com/restfully/shop/services/CustomerDatabaseResource.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/CustomerDatabaseResource.java"


*src/main/java/com/restfully/shop/services/CustomerResource.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/CustomerResource.java"


*src/main/java/com/restfully/shop/services/FirstLastCustomerResource.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/FirstLastCustomerResource.java"



*src/main/java/com/restfully/shop/services/ShoppingApplication.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/ShoppingApplication.java"


*src/test/java/com/restfully/shop/test/CustomerResourceTest.java*

!CODEFILE "./src/test/java/com/restfully/shop/test/CustomerResourceTest.java"                            