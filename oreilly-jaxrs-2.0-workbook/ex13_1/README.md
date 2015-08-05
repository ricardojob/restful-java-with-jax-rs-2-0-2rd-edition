# Customer Chat


<!-- toc -->


This project is a simple example showing a customer chat app using JAX-RS 2.0 async client and server apis.


## System Requirements:

- Maven 3.0.4 or higher


## Running the project:

1. In root directory **mvn jetty:run**

This will start the web server

2. In a different window **mvn exec:java -Dexec.mainClass=ChatClient -Dexec.args="&lt;Your first name&gt;"**


You must specify your first name you will use in the chat.  When this comes up, enter your messages.  You can
also open up another window to have a conversation.






## Source Code


### Hierarchy
```
ex13_1
|-- pom.xml
`-- src
    `-- main
        |-- java
        |   |-- ChatClient.java
        |   `-- com
        |       `-- restfully
        |           `-- shop
        |               |-- domain
        |               |   `-- Customer.java
        |               `-- services
        |                   |-- CustomerChat.java
        |                   |-- CustomerResource.java
        |                   `-- ShoppingApplication.java
        `-- webapp
            `-- WEB-INF
                `-- web.xml
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


*src/main/java/com/restfully/shop/services/CustomerChat.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/CustomerChat.java"


*src/main/java/com/restfully/shop/services/ShoppingApplication.java*

!CODEFILE "./src/main/java/com/restfully/shop/services/ShoppingApplication.java"


*src/test/java/com/restfully/shop/test/CustomerResourceTest.java*

!CODEFILE "./src/test/java/com/restfully/shop/test/CustomerResourceTest.java"