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


### CustomerResource: Our JAX-RS Service

Now that we have defined a domain object that will represent our customers at runtime, we need to implement our JAX-RS service so that remote clients can interact with our customer database. A JAX-RS service is a Java class that uses JAX-RS annotations to bind and map specific incoming HTTP requests to Java methods that can service these requests. While JAX-RS can integrate with popular component models like Enterprise JavaBeans (EJB), Web Beans, JBoss Seam, and Spring, it does define its own lightweight model.


In vanilla JAX-RS, services can either be singletons or per-request objects. A *singleton* means that one and only one Java object services HTTP requests. *Per-request* means that a Java object is created to process each incoming request and is thrown away at the end of that request. Per-request also implies statelessness, as no service state is held between requests.


For our example, we will write a **CustomerResource** class to implement our JAX-RS service and assume it will be a singleton. In this example, we need **CustomerResource** to be a singleton because it is going to hold state. It is going to keep a map of **Customer** objects in memory that our remote clients can access. In a real system, **CustomerResource** would probably interact with a database to retrieve and store customers and wouldn’t need to hold state between requests. In this database scenario, we could make **CustomerResource** per-request and thus stateless. Let’s start by looking at the first few lines of our class to see how to start writing a JAX-RS service:


```Java
package com.restfully.shop.services;

import ...;

@Path("/customers")
public class CustomerResource {

   private Map<Integer, Customer> customerDB =
                            new ConcurrentHashMap<Integer, Customer>();
   private AtomicInteger idCounter = new AtomicInteger();
```


As you can see, **CustomerResource** is a plain Java class and doesn’t implement any particular JAX-RS interface. The **@javax.ws.rs.Path** annotation placed on the **CustomerResource** class designates the class as a JAX-RS service. Java classes that you want to be recognized as JAX-RS services must have this annotation. Also notice that the **@Path** annotation has the value of **/customers**. This value represents the relative root URI of our customer service. If the absolute base URI of our server is *http://shop.restfully.com*, methods exposed by our **CustomerResource** class would be available under *http://shop.restfully.com/customers*.


In our class, we define a simple map in the **customerDB** field that will store created **Customer** objects in memory. We use a **java.util.concurrent.ConcurrentHashMap** for **customerDB** because **CustomerResource** is a singleton and will have concurrent requests accessing the map. Using a **java.util.HashMap** would trigger concurrent access exceptions in a multithreaded environment. Using a **java.util.Hashtable** creates a synchronization bottleneck. **ConcurrentHashMap** is our best bet. The **idCounter** field will be used to generate IDs for newly created **Customer** objects. For concurrency reasons, we use a **java.util.concurrent.atomic.AtomicInteger**, as we want to always have a unique number generated. Of course, these two lines of code have nothing to do with JAX-RS and are solely artifacts required by our simple example.


#### Creating customers

Let’s now take a look at how to create customers in our **CustomerResource** class:

```Java
   @POST
   @Consumes("application/xml")
   public Response createCustomer(InputStream is) {
      Customer customer = readCustomer(is);
      customer.setId(idCounter.incrementAndGet());
      customerDB.put(customer.getId(), customer);
      System.out.println("Created customer " + customer.getId());
      return Response.created(URI.create("/customers/"
                                   + customer.getId())).build();
   }
```

We will implement customer creation using the same model as that used in [Chapter 2](../chapter2/designing_restful_services.md). An HTTP POST request sends an XML document representing the customer we want to create. The **createCustomer()** method receives the request, parses the document, creates a **Customer** object from the document, and adds it to our **customerDB** map. The **createCustomer()** method returns a response code of 201, “Created,” along with a Location header pointing to the absolute URI of the customer we just created. So how does the **createCustomer()** method do all this? Let’s examine further.


To bind HTTP POST requests to the **createCustomer()** method, we annotate it with the **@javax.ws.rs.POST** annotation. The **@Path** annotation we put on the **CustomerResource** class, combined with this **@POST** annotation, binds all POST requests going to the relative URI **/customers** to the Java method **createCustomer()**.


The **@javax.ws.rs.Consumes** annotation applied to **createCustomer()** specifies which media type the method is expecting in the message body of the HTTP input request. If the client POSTs a media type other than XML, an error code is sent back to the client.


The **createCustomer()** method takes one **java.io.InputStream** parameter. In JAX-RS, any non-JAX-RS-annotated parameter is considered to be a representation of the HTTP input request’s message body. In this case, we want access to the method body in its most basic form, an **InputStream**.

> #### Warning
> Only one Java method parameter can represent the HTTP message body. This means any other parameters must be annotated with one of the JAX-RS annotations discussed in [Chapter 5](../chapter5/jax_rs_injection.md).


The implementation of the method reads and transforms the POSTed XML into a **Customer** object and stores it in the **customerDB** map. The method returns a complex response to the client using the **javax.ws.rs.core.Response** class. The static **Response.created()** method creates a **Response** object that contains an HTTP status code of 201, “Created.” It also adds a **Location** header to the HTTP response with the value of something like *http://shop.restfully.com/customers/333*, depending on the base URI of the server and the generated ID of the **Customer** object (333 in this example).


#### Retrieving customers


```Java
   @GET
   @Path("{id}")
   @Produces("application/xml")
   public StreamingOutput getCustomer(@PathParam("id") int id) {
      final Customer customer = customerDB.get(id);
      if (customer == null) {
         throw new WebApplicationException(Response.Status.NOT_FOUND);
      }
      return new StreamingOutput() {
         public void write(OutputStream outputStream)
                       throws IOException, WebApplicationException {
            outputCustomer(outputStream, customer);
         }
      };
   }
```

We annotate the **getCustomer()** method with the **@javax.ws.rs.GET** annotation to bind HTTP GET operations to this Java method.


We also annotate **getCustomer()** with the **@javax.ws.rs.Produces** annotation. This annotation tells JAX-RS which HTTP **Content-Type** the GET response will be. In this case, it is **application/xml**.


In the implementation of the method, we use the **id** parameter to query for a **Customer** object in the customerDB map. If this customer does not exist, we throw the **javax.ws.rs.WebApplicationException**. This exception will set the HTTP response code to 404, “Not Found,” meaning that the customer resource does not exist. We’ll discuss more about exception handling in [Chapter 7](../chapter7/server_responses_and_exception_handling.md), so I won’t go into more detail about the **WebApplicationException** here.


We will write the response manually to the client through a **java.io.OutputStream**. In JAX-RS, when you want to do streaming manually, you must implement and return an instance of the **javax.ws.rs.core.StreamingOutput** interface from your JAX-RS method. **StreamingOutput** is a callback interface with one callback method, **write()**:

```Java
package javax.ws.rs.core;

public interface StreamingOutput {
   public void write(OutputStream os) throws IOException,
                                             WebApplicationException;
}
```


In the last line of our **getCustomer()** method, we implement and return an inner class implementation of **StreamingOutput**. Within the **write()** method of this inner class, we delegate back to a utility method called **outputCustomer()** that exists in our **CustomerResource** class. When the JAX-RS provider is ready to send an HTTP response body back over the network to the client, it will call back to the **write()** method we implemented to output the XML representation of our **Customer** object.


In general, you will not use the **StreamingOutput** interface to output responses. In [Chapter 6](../chapter6/jax_rs_content_handlers.md), you will see that JAX-RS has a bunch of nice content handlers that can automatically convert Java objects straight into the data format you are sending across the wire. I didn’t want to introduce too many new concepts in the first introductory chapter, so the example only does simple streaming.


#### Updating a customer


The last RESTful operation we have to implement is updating customers. In [Chapter 2](../chapter2/designing_restful_services.md), we used **PUT /customers/{id}**, while passing along an updated XML representation of the customer. This is implemented in the **updateCustomer()** method of our **CustomerResource** class:


```Java
   @PUT
   @Path("{id}")
   @Consumes("application/xml")
   public void updateCustomer(@PathParam("id") int id,
                               InputStream is) {
      Customer update = readCustomer(is);
      Customer current = customerDB.get(id);
      if (current == null)
        throw new WebApplicationException(Response.Status.NOT_FOUND);

      current.setFirstName(update.getFirstName());
      current.setLastName(update.getLastName());
      current.setStreet(update.getStreet());
      current.setState(update.getState());
      current.setZip(update.getZip());
      current.setCountry(update.getCountry());
   }
```


We annotate the **updateCustomer()** method with **@javax.ws.rs.PUT** to bind HTTP PUT requests to this method. Like our **getCustomer()** method, **updateCustomer()** is annotated with an additional **@Path** annotation so that we can match **/customers/{id}** URIs.


The **updateCustomer()** method takes two parameters. The first is an **id** parameter that represents the **Customer** object we are updating. Like **getCustomer()**, we use the **@PathParam** annotation to extract the ID from the incoming request URI. The second parameter is an **InputStream** that will allow us to read in the XML document that was sent with the PUT request. Like **createCustomer()**, a parameter that is not annotated with a JAX-RS annotation is considered a representation of the body of the incoming message.


In the first part of the method implementation, we read in the XML document and create a **Customer** object out of it. The method then tries to find an existing **Customer** object in the **customerDB** map. If it doesn’t exist, we throw a **WebApplicationException** that will send a 404, “Not Found,” response code back to the client. If the **Customer** object does exist, we update our existing **Customer** object with new updated values.










