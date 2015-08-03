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
| --------- | :-------------: | :-----: |
| BadRequestException | 400 | Malformed message |
| NotAuthorizedException | 401 | Authentication failure |
| ForbiddenException | 403 | Not permitted to access |
| NotFoundException | 404 | Couldn’t find resource |
| NotAllowedException | 405 | HTTP method not supported |
| NotAcceptableException | 406 | Client media type requested not supported |
| NotSupportedException | 415 | Client posted media type not supported |
| InternalServerErrorException | 500 | General server error |
| ServiceUnavailableException | 503 | Server is temporarily unavailable or busy |


**BadRequestException** is used when the client sends something to the server that the server cannot interpret. The JAX-RS runtime will actually throw this exception in certain scenarios. The most obvious is when a PUT or POST request has submitted malformed XML or JSON that the **MessageBodyReader** fails to parse. JAX-RS will also throw this exception if it fails to convert a header or cookie value to the desired type. For example:


```Java
@HeaderParam("Custom-Header") int header;
@CookieParam("myCookie") int cookie;
```


If the HTTP request’s **Custom-Header** value or the **myCookie** value cannot be parsed into an integer, **BadRequestException** is thrown.


**NotAuthorizedException** is used when you want to write your own authentication protocols. The 401 HTTP response code this exception represents requires you to send back a challenge header called **WWW-Authenticate**. This header is used to tell the client how it should authenticate with the server. **NotAuthorizedException** has a few convenience constructors that make it easier to build this header automatically:


```Java
    public NotAuthorizedException(Object challenge, Object... moreChallenges) {}
```


For example, if I wanted to tell the client that OAuth Bearer tokens are required for authentication, I would throw this exception:


```Java
    throw new NotAuthorizedException("Bearer");
```


The client would receive this HTTP response:

```
HTTP/1.1 401 Not Authorized
WWW-Authenticate: Bearer
```


**ForbiddenException** is generally used when the client making the invocation does not have permission to access the resource it is invoking on. In Java EE land, this is usually because the authenticated client does not have the specific role mapping required.


**NotFoundException** is used when you want to tell the client that the resource it is requesting does not exist. There are also some error conditions where the JAX-RS runtime will throw this exception automatically. If the JAX-RS runtime fails to inject into an **@PathParam**, **@QueryParam**, or **@MatrixParam**, it will throw this exception. Like in the conditions discussed for **BadRequestException**, this can happen if you are trying to convert to a type the parameter value isn’t meant for.


**NotAllowedException** is used when the HTTP method the client is trying to invoke isn’t supported by the resource the client is accessing. The JAX-RS runtime will automatically throw this exception if there isn’t a JAX-RS method that matches the invoked HTTP method.


**NotAcceptableException** is used when the client is requesting a specific format through the **Accept** header. The JAX-RS runtime will automatically throw this exception if there is not a JAX-RS method with an **@Produces** annotation that is compatible with the client’s **Accept** header.


**NotSupportedException** is used when a client is posting a representation that the server does not understand. The JAX-RS runtime will automatically throw this exception if there is no JAX-RS method with an @Consumes annotation that matches the **Content-Type** of the posted entity.


**InternalServerErrorException** is a general-purpose error that is thrown by the server. For applications, you would throw this exception if you’ve reached an error condition that doesn’t really fit with the other HTTP error codes. The JAX-RS runtime throws this exception if a **MessageBodyWriter** fails or if there is an exception thrown from an **ExceptionMapper**.


**ServiceUnavailableException** is used when the server is temporarily unavailable or busy. In most cases, it is OK for the client to retry the request at a later time. The HTTP 503 status code is often sent with a **Retry-After** header. This header is a suggestion to the client when it might be OK to retry the request. Its value is in seconds or a formatted date string. 


**ServiceUnavailableException** has a few convenience constructors to help with initializing this header:

```Java
    public ServiceUnavailableException(Long retryAfter) {}
    public ServiceUnavailableException(Date retryAfter) {}
```


#### Mapping default exceptions


What’s interesting about the default error handling for JAX-RS is that you can write an **ExceptionMapper** for these scenarios. For example, if you want to send back a different response to the client when JAX-RS cannot find an **@Produces** match for an **Accept** header, you can write an **ExceptionMapper** for **NotAcceptableException**. This gives you complete control on how errors are handled by your application.


