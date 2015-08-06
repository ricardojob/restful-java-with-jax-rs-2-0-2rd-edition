# Example ex07_1: ExceptionMapper


This example is a slight modification from *ex06_1* to show you how you can use **ExceptionMappers**. Let’s take a look at the CustomerResource class to see what is different:


```Java:src/main/java/com/restfully/shop/services/CustomerResource.java
@Path("/customers")
public class CustomerResource {
...
   @GET
   @Path("{id}")
   @Produces("application/xml")
   public Customer getCustomer(@PathParam("id") int id)
   {
      Customer customer = customerDB.get(id);
      if (customer == null)
      {
         throw new CustomerNotFoundException("Could not find customer "
                                      + id);
      }
      return customer;
   }

   @PUT
   @Path("{id}")
   @Consumes("application/xml")
   public void updateCustomer(@PathParam("id") int id,
                              Customer update)
   {
      Customer current = customerDB.get(id);
      if (current == null)
        throw new CustomerNotFoundException("Could not find customer " + id);

      current.setFirstName(update.getFirstName());
      current.setLastName(update.getLastName());
      current.setStreet(update.getStreet());
      current.setState(update.getState());
      current.setZip(update.getZip());
      current.setCountry(update.getCountry());
    }
}
```


In *ex06_1*, our **getCustomer()** and **updateCustomer()** methods threw a **javax.ws.rs.WebApplicationException**. We’ve replaced this exception with our own custom class, **CustomerNotFoundException**:



```Java:src/main/java/com/restfully/shop/services/CustomerNotFoundException.java
public class CustomerNotFoundException extends RuntimeException
{
   public NotFoundException(String s)
   {
      super(s);
   }
}
```


There’s nothing really special about this exception class other than it inherits from **java.lang.RuntimeException**. What we are going to do, though, is map this thrown exception to a **Response** object using an **ExceptionMapper**:


```Java:src/main/java/com/restfully/shop/services/CustomerNotFoundExceptionMapper.java
@Provider
public class NotFoundExceptionMapper
                      implements ExceptionMapper<CustomerNotFoundException>
{
   public Response toResponse(NotFoundException exception)
   {
      return Response.status(Response.Status.NOT_FOUND)
                     .entity(exception.getMessage())
                     .type("text/plain").build();
   }
}
```

When a client makes a GET request to a customer URL that does not exist, the **CustomerResource.getCustomer()** method throws a **CustomerNotFoundException**. This exception is caught by the JAX-RS runtime, and the **NotFoundExceptionMapper.toResponse()** method is called. This method creates a **Response** object that returns a 404 status code and a plain-text error message.



The last thing we have to do is modify our **Application** class to register the **ExceptionMapper**:


```Java:src/main/java/com/restfully/shop/services/ShoppingApplication.java
public class ShoppingApplication extends Application {
   private Set<Object> singletons = new HashSet<Object>();
   private Set<Class<?>> classes = new HashSet<Class<?>>();

   public ShoppingApplication()
   {
      singletons.add(new CustomerResource());
      classes.add(CustomerNotFoundExceptionMapper.class);
   }

   @Override
   public Set<Class<?>> getClasses()
   {
      return classes;
   }

   @Override
   public Set<Object> getSingletons()
   {
      return singletons;
   }
}
```


### The Client Code


The client code for this example is very simple. We make a GET request to a customer resource that doesn’t exist:


```Java:src/test/java/com/restfully/shop/test/CustomerResourceTest.java
package com.restfully.shop.test;

import javax.ws.rs.NotFoundException;

...

   @Test
   public void testCustomerResource() throws Exception
   {
      try
      {
         Customer customer = client.target
                                   ("http://localhost:8080/services/customers/1")
                                   .request().get(Customer.class);
         System.out.println("Should never get here!");
      }
      catch (NotFoundException e)
      {
         System.out.println("Caught error!");
         String error = e.getResponse().readEntity(String.class);
         System.out.println(error);
      }
   }
```


When this client code runs, the server will throw a **CustomerNotFoundException**, which is converted into a 404 response back to the client. The client code handles the error as discussed in Exception Handling and throws a **javax.ws.rs.NotFoundException**, which is handled in the catch block. The error message is extracted from the HTTP error response and displayed to the console.



### Build and Run the Example Program


Perform the following steps:

1. Open a command prompt or shell terminal and change to the *ex07_1* directory of the workbook example code. 
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md). 
3. Perform the build and run the example by typing **maven install**.