# Developing a JAX-RS RESTful Service


Let’s start by implementing one of the resources of the order entry system we defined in [Chapter 2](../chapter2/designing_restful_services.md). Specifically, we’ll define a JAX-RS service that allows us to read, create, and update **Customers**. To do this, we will need to implement two Java classes. One class will be used to represent actual **Customers**. The other will be our JAX-RS service.


### Customer: The Data Class

First, we will need a Java class to represent customers in our system. We will name this class **Customer**. **Customer** is a simple Java class that defines eight properties: **id**, **firstName**, **lastName**, **street**, **city**, **state**, **zip**, and **country**. *Properties* are attributes that can be accessed via the class’s fields or through public set and get methods. A Java class that follows this pattern is also called a *Java bean*: