# Exception Handling


Errors can be reported to a client either by creating and returning the appropriate **Response** object or by throwing an exception. Application code is allowed to throw any checked (classes extending **java.lang.Exception**) or unchecked (classes extending **java.lang.RuntimeException**) exceptions they want. Thrown exceptions are handled by the JAX-RS runtime if you have registered an exception mapper. Exception mappers can convert an exception to an HTTP response. If the thrown exception is not handled by a mapper, it is propagated and handled by the container (i.e., servlet) JAX-RS is running within. JAX-RS also provides the **javax.ws.rs.WebApplicationException**. This can be thrown by application code and automatically processed by JAX-RS without having to write an explicit mapper. Let’s look at how to use the **WebApplicationException** first. We’ll then examine how to write your own specific exception mappers.


### javax.ws.rs.WebApplicationException


JAX-RS has a built-in unchecked exception that applications can throw. This exception is preinitialized with either a **Response** or a particular status code:


```Java
public class WebApplicationException extends RuntimeException {

   public WebApplicationException() {...}
   public WebApplicationException(Response response) {...}
   public WebApplicationException(int status) {...}
   public WebApplicationException(Response.Status status) {...}
   public WebApplicationException(Throwable cause) {...}
   public WebApplicationException(Throwable cause,
                                       Response response) {...}
   public WebApplicationException(Throwable cause, int status) {...}
   public WebApplicationException(Throwable cause,
                                   Response.Status status) {...}

   public Response getResponse() {...]
}
```


When JAX-RS sees that a **WebApplicationException** has been thrown by application code, it catches the exception and calls its **getResponse()** method to obtain a **Response** to send back to the client. If the application has initialized the **WebApplicationException** with a status code or **Response** object, that code or Response will be used to create the actual HTTP response. Otherwise, the **WebApplicationException** will return a status code of 500, “Internal Server Error,” to the client.


For example, let’s say we have a web service that allows clients to query for customers represented in XML:


```Java
@Path("/customers")
public class CustomerResource {

   @GET
   @Path("{id}")
   @Produces("application/xml")
   public Customer getCustomer(@PathParam("id") int id) {

       Customer cust = findCustomer(id);
       if (cust == null) {
         throw new WebApplicationException(Response.Status.NOT_FOUND);
       }
       return cust;
   }
}
```


In this example, if we do not find a **Customer** instance with the given ID, we throw a **WebApplicationException** that causes a 404, “Not Found,” status code to be sent back to the client.



### Exception Mapping


Many applications have to deal with a multitude of exceptions thrown from application code and third-party frameworks. Relying on the underlying servlet container to handle the exception doesn’t give us much flexibility. Catching and then wrapping all these exceptions within **WebApplicationException** would become quite tedious. Alternatively, you can implement and register instances of **javax.ws.rs.ext.ExceptionMapper**. These objects know how to map a thrown application exception to a **Response** object:


```Java
public interface ExceptionMapper<E extends Throwable> {
{
   Response toResponse(E exception);
}
```


For example, one exception that is commonly thrown in Java Persistence API (JPA)–based database applications is **javax.persistence.EntityNotFoundException**. It is thrown when JPA cannot find a particular object in the database. Instead of writing code to handle this exception explicitly, you could write an **ExceptionMapper** to handle this exception for you. Let’s do that:


```Java
@Provider
public class EntityNotFoundMapper
     implements ExceptionMapper<EntityNotFoundException> {

   public Response toResponse(EntityNotFoundException e) {
      return Response.status(Response.Status.NOT_FOUND).build();
   }
}
```


Our **ExceptionMapper** implementation must be annotated with the **@Provider** annotation. This tells the JAX-RS runtime that it is a component. The class implementing the **ExceptionMapper** interface must provide the parameterized type of the **ExceptionMapper**. JAX-RS uses this generic type information to match up thrown exceptions to **ExceptionMappers**. Finally, the **toResponse()** method receives the thrown exception and creates a **Response** object that will be used to build the HTTP response.


JAX-RS supports exception inheritance as well. When an exception is thrown, JAX-RS will first try to find an **ExceptionMapper** for that exception’s type. If it cannot find one, it will look for a mapper that can handle the exception’s superclass. It will continue this process until there are no more superclasses to match against.


Finally, **ExceptionMappers** are registered with the JAX-RS runtime using the deployment APIs discussed in [Chapter 14](../chapter14/ejb_integration.md).


### Exception Hierarchy


JAX-RS 2.0 has added a nice exception hierarchy for various HTTP error conditions. So, instead of creating an instance of **WebApplicationException** and initializing it with a specific status code, you can use one of these exceptions instead. We can change our previous example to use **javax.ws.rs.NotFoundException**:


```Java
@Path("/customers")
public class CustomerResource {

   @GET
   @Path("{id}")
   @Produces("application/xml")
   public Customer getCustomer(@PathParam("id") int id) {

       Customer cust = findCustomer(id);
       if (cust == null) {
         throw new NotFoundException());
       }
       return cust;
   }
}
```


Like the other exceptions in the exception hierarchy, **NotFoundException** inherits from **WebApplicationException**. If you looked at the code, you’d see that in its constructor it is initializing the status code to be 404. Table 7-1 lists some other exceptions you can use for error conditions that are under the **javax.ws.rs package**.


Table 7-1. JAX-RS exception hierarchy

| Exception | Status code | Description |
| --------- | :-------------: | -----: |
| BadRequestException | 400 | Malformed message |



