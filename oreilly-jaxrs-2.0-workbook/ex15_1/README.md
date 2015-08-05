# @NameBinding and SecurityContext


<!-- toc -->



This project is a simple example that creates a custom one-time-password authentication protocol.  It uses **ContainerRequestFilter** and
**ClientRequestFilter** to implement this along with a **@NameBinding** to secure which parts you want to secure.  It also has
an authorization filter that allows you to specify how many times per day a user is allowed to invoke a method.


## System Requirements:

- Maven 3.0.4 or higher


## Building the project:

1. In root directory **mvn clean install**

This will build a WAR and run it with embedded Jetty.




## Source Code


### Hierarchy
```
ex15_1
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
    |   |               |   |-- AllowedPerDay.java
    |   |               |   |-- OTP.java
    |   |               |   |-- OTPAuthenticated.java
    |   |               |   |-- OneTimePasswordAuthenticator.java
    |   |               |   |-- OneTimePasswordGenerator.java
    |   |               |   `-- PerDayAuthorizer.java
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


*src/main/java/com/restfully/shop/features/AllowedPerDay.java*

!CODEFILE "./src/main/java/com/restfully/shop/features/AllowedPerDay.java"


*src/main/java/com/restfully/shop/features/OTP.java*

!CODEFILE "./src/main/java/com/restfully/shop/features/OTP.java"


*src/main/java/com/restfully/shop/features/OTPAuthenticated.java*

!CODEFILE "./src/main/java/com/restfully/shop/features/OTPAuthenticated.java"


*src/main/java/com/restfully/shop/features/OneTimePasswordAuthenticator.java*

!CODEFILE "./src/main/java/com/restfully/shop/features/OneTimePasswordAuthenticator.java"


*src/main/java/com/restfully/shop/features/OneTimePasswordGenerator.java*

!CODEFILE "./src/main/java/com/restfully/shop/features/OneTimePasswordGenerator.java"


*src/main/java/com/restfully/shop/features/PerDayAuthorizer.java*

!CODEFILE "./src/main/java/com/restfully/shop/features/PerDayAuthorizer.java"



*src/main/java/com/restfully/shop/services/CustomerResource.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/CustomerResource.java"


*src/main/java/com/restfully/shop/services/ShoppingApplication.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/ShoppingApplication.java"


*src/test/java/com/restfully/shop/test/CustomerResourceTest.java*

!CODEFILE "./src/test/java/com/restfully/shop/test/CustomerResourceTest.java"