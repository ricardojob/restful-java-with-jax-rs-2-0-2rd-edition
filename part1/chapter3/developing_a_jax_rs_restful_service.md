# Developing a JAX-RS RESTful Service


Let’s start by implementing one of the resources of the order entry system we defined in [Chapter 2](../chapter2/designing_restful_services.md). Specifically, we’ll define a JAX-RS service that allows us to read, create, and update **Customers**. To do this, we will need to implement two Java classes. One class will be used to represent actual **Customers**. The other will be our JAX-RS service.


### Customer: The Data Class

First, we will need a Java class to represent customers in our system. We will name this class **Customer**. **Customer** is a simple Java class that defines eight properties: **id**, **firstName**, **lastName**, **street**, **city**, **state**, **zip**, and **country**. *Properties* are attributes that can be accessed via the class’s fields or through public set and get methods. A Java class that follows this pattern is also called a *Java bean*:

```Java
package com.restfully.shop.domain;

public class Customer {
   private int id;
   private String firstName;
   private String lastName;
   private String street;
   private String city;
   private String state;
   private String zip;
   private String country;

   public int getId() { return id; }
   public void setId(int id) { this.id = id; }

   public String getFirstName() { return firstName; }
   public void setFirstName(String firstName) {
                  this.firstName = firstName; }

   public String getLastName() { return lastName; }
   public void setLastName(String lastName) {
                        this.lastName = lastName; }

   public String getStreet() { return street; }
   public void setStreet(String street) { this.street = street; }

   public String getCity() { return city; }
   public void setCity(String city) { this.city = city;  }

   public String getState() { return state; }
   public void setState(String state) { this.state = state; }

   public String getZip() { return zip; }
   public void setZip(String zip) { this.zip = zip; }

   public String getCountry() { return country; }
   public void setCountry(String country) { this.country = country; }
}
```


In an Enterprise Java application, the **Customer** class would usually be a Java Persistence API (JPA) Entity bean and would be used to interact with a relational database. It could also be annotated with JAXB annotations that allow you to map a Java class directly to XML. To keep our example simple, **Customer** will be just a plain Java object and stored in memory. In [Chapter 6](../chapter6/jax_rs_content_handlers.md), I’ll show how you can use JAXB with JAX-RS to make translating between your customer’s data format (XML) and your Customer objects easier. [Chapter 14](../chapter14/deployment_and_integration.md) will show you how JAX-RS works in the context of a Java EE (Enterprise Edition) application and things like JPA.