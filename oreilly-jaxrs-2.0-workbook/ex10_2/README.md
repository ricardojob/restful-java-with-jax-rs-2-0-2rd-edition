# Link Headers and UriBuilder


<!-- toc -->


This project is an example of using UriBuilder to enable HATEOAS through Link headers


## System Requirements:

- Maven 3.0.4 or higher


## Building the project:


1. In root directoy **mvn clean install**

This will build a WAR and run it with embedded Jetty.




## Source Code


### Hierarchy
```
ex10_2
|-- pom.xml
`-- src
    |-- main
    |   |-- java
    |   |   `-- com
    |   |       `-- restfully
    |   |           `-- shop
    |   |               |-- domain
    |   |               |   |-- Customer.java
    |   |               |   |-- Customers.java
    |   |               |   |-- LineItem.java
    |   |               |   |-- Order.java
    |   |               |   `-- Orders.java
    |   |               `-- services
    |   |                   |-- CustomerResource.java
    |   |                   |-- OrderResource.java
    |   |                   |-- ShoppingApplication.java
    |   |                   `-- StoreResource.java
    |   `-- webapp
    |       `-- WEB-INF
    |           `-- web.xml
    `-- test
        `-- java
            `-- com
                `-- restfully
                    `-- shop
                        `-- test
                            `-- OrderResourceTest.java
```


### Details


*pom.xml*

!CODEFILE "./pom.xml"


*src/main/webapp/WEB-INF/web.xml*

!CODEFILE "./src/main/webapp/WEB-INF/web.xml"


*src/main/java/com/restfully/shop/domain/Customer.java*

!CODEFILE "./src/main/java/com/restfully/shop/domain/Customer.java"


*src/main/java/com/restfully/shop/domain/Customers.java*

!CODEFILE "./src/main/java/com/restfully/shop/domain/Customers.java"


*src/main/java/com/restfully/shop/domain/LineItem.java*

!CODEFILE "./src/main/java/com/restfully/shop/domain/LineItem.java"


*src/main/java/com/restfully/shop/domain/Order.java*

!CODEFILE "./src/main/java/com/restfully/shop/domain/Order.java"


*src/main/java/com/restfully/shop/domain/Orders.java*

!CODEFILE "./src/main/java/com/restfully/shop/domain/Orders.java"


*src/main/java/com/restfully/shop/services/CustomerResource.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/CustomerResource.java"


*src/main/java/com/restfully/shop/services/OrderResource.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/OrderResource.java"


*src/main/java/com/restfully/shop/services/StoreResource.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/StoreResource.java"


*src/main/java/com/restfully/shop/services/ShoppingApplication.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/ShoppingApplication.java"


*src/test/java/com/restfully/shop/test/CustomerResourceTest.java*

!CODEFILE "./src/test/java/com/restfully/shop/test/CustomerResourceTest.java"