# Chapter 3. Your First JAX-RS Service


The first two chapters of this book focused on the theory of REST and designing the RESTful interface for a simple ecommerce order entry system. Now it’s time to implement a part of our system in the Java language.


Writing RESTful services in Java has been possible for years with the servlet API. If you have written a web application in Java, you are probably already very familiar with servlets. Servlets bring you very close to the HTTP protocol and require a lot of boilerplate code to move information to and from an HTTP request. In 2008, a new specification called JAX-RS was defined to simplify RESTful service implementation.


JAX-RS is a framework that focuses on applying Java annotations to plain Java objects. It has annotations to bind specific URI patterns and HTTP operations to individual methods of your Java class. It has parameter injection annotations so that you can easily pull in information from the HTTP request. It has message body readers and writers that allow you to decouple data format marshalling and unmarshalling from your Java data objects. It has exception mappers that can map an application-thrown exception to an HTTP response code and message. Finally, it has some nice facilities for HTTP content negotiation.


This chapter gives a brief introduction to writing a JAX-RS service. You’ll find that getting it up and running is fairly simple.

