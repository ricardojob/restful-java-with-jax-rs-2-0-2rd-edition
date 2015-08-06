# Response Filter with DynamicFeature


<!-- toc -->


Implements a **@MaxAge** annotation that binds a response filter via a **DynamicFeature**.  Models example given in Chapter 12.


## System Requirements:

- Maven 3.0.4 or higher


## Building the project:

1. In root directory **mvn clean install**

This will build a WAR and run it with embedded Jetty.





## Source Code


### Hierarchy
```
ex12_1
|-- pom.xml
`-- src
    |-- main
    |   |-- java
    |   |   `-- com
    |   |       `-- restfully
    |   |           `-- shop
    |   |               |-- domain
    |   |               |   `-- Customer.java
    |   |               |-- features
    |   |               |   |-- CacheControlFilter.java
    |   |               |   |-- MaxAge.java
    |   |               |   `-- MaxAgeFeature.java
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


### Details


*pom.xml*

!CODEFILE "./pom.xml"


*src/main/webapp/WEB-INF/web.xml*

!CODEFILE "./src/main/webapp/WEB-INF/web.xml"


*src/main/java/com/restfully/shop/domain/Customer.java*

!CODEFILE "./src/main/java/com/restfully/shop/domain/Customer.java"


*src/main/java/com/restfully/shop/features/CacheControlFilter.java*

!CODEFILE "./src/main/java/com/restfully/shop/features/CacheControlFilter.java"


*src/main/java/com/restfully/shop/features/MaxAge.java*

!CODEFILE "./src/main/java/com/restfully/shop/features/MaxAge.java"


*src/main/java/com/restfully/shop/features/MaxAgeFeature.java*

!CODEFILE "./src/main/java/com/restfully/shop/features/MaxAgeFeature.java"


*src/main/java/com/restfully/shop/services/CustomerResource.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/CustomerResource.java"


*src/main/java/com/restfully/shop/services/ShoppingApplication.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/ShoppingApplication.java"


*src/test/java/com/restfully/shop/test/CustomerResourceTest.java*

!CODEFILE "./src/test/java/com/restfully/shop/test/CustomerResourceTest.java"