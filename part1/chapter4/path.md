# @Path


There’s more to the **@javax.ws.rs.Path** annotation than what we saw in our simple example in [Chapter 3](../chapter3/your_first_jax_rs_service.md). **@Path** can have complex matching expressions so that you can be more specific about what requests get bound to which incoming URIs. **@Path** can also be used on a Java method as sort of an object factory for subresources of your application. We’ll examine both in this section.


### Binding URIs


The **@javax.ws.rs.Path** annotation in JAX-RS is used to define a URI matching pattern for incoming HTTP requests. It can be placed upon a class or on one or more Java methods. For a Java class to be eligible to receive any HTTP requests, the class must be annotated with at least the **@Path("/")** expression. These types of classes are called JAX-RS *root resources*.


The value of the **@Path** annotation is an expression that denotes a relative URI to the context root of your JAX-RS application. For example, if you are deploying into a WAR archive of a servlet container, that WAR will have a base URI that browsers and remote clients use to access it. **@Path** expressions are relative to this URI.


To receive a request, a Java method must have at least an HTTP method annotation like **@javax.ws.rs.GET** applied to it. This method is not required to have an **@Path** annotation on it, though. For example:



```Java
@Path("/orders")
public class OrderResource {
   @GET
   public String getAllOrders() {
       ...
   }
}
```


An HTTP request of **GET /orders** would dispatch to the **getAllOrders()** method.


You can also apply **@Path** to your Java method. If you do this, the URI matching pattern is a concatenation of the class’s **@Path** expression and that of the method’s. For example:

```Java
@Path("/orders")
public class OrderResource {

   @GET
   @Path("unpaid")
   public String getUnpaidOrders() {
      ...
   }
}
```

So, the URI pattern for **getUnpaidOrders()** would be the relative URI **/orders/unpaid**.


### @Path Expressions


The value of the **@Path** annotation is usually a simple string, but you can also define more complex expressions to satisfy your URI matching needs.


#### Template parameters


In [Chapter 3](../chapter3/your_first_jax_rs_service.md), we wrote a customer access service that allowed us to query for a specific customer using a wildcard URI pattern:


```Java
@Path("/customers")
public class CustomerResource {

   @GET
   @Path("{id}")
   public String getCustomer(@PathParam("id") int id) {
      ...
   }
}
```

These template parameters can be embedded anywhere within an **@Path** declaration. For example:


```Java
@Path("/")
public class CustomerResource {

   @GET
   @Path("customers/{firstname}-{lastname}")
   public String getCustomer(@PathParam("firstname") String first,
                             @PathParam("lastname") String last) {
      ...
   }
}
```


In our example, the URI is constructed with a customer’s first name, followed by a hyphen, ending with the customer’s last name. So, the request **GET /customers/333** would no longer match to **getCustomer()**, but a **GET/customers/bill-burke** request would.


#### Regular expressions


**@Path** expressions are not limited to simple wildcard matching expressions. For example, our **getCustomer()** method takes an integer parameter. We can change our **@Path** value to match only digits:


```Java
@Path("/customers")
public class CustomerResource {

   @GET
   @Path("{id : \\d+}")
   public String getCustomer(@PathParam("id") int id) {
      ...
   }
}
```


Regular expressions are not limited in matching one segment of a URI. For example:


```Java
@Path("/customers")
public class CustomerResource {

   @GET
   @Path("{id : .+}")
   public String getCustomer(@PathParam("id") String id) {
      ...
   }

   @GET
   @Path("{id : .+}/address")
   public String getAddress(@PathParam("id") String id) {
      ...
   }

}
```


We’ve changed **getCustomer()**’s **@Path** expression to **{id : .+}**. The **.+** is a regular expression that will match any stream of characters after **/customers**. So, the **GET /customers/bill/burke** request would be routed to **getCustomer()**.












