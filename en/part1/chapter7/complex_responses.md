# Complex Responses


Sometimes the web service you are writing can’t be implemented using the default request/response behavior inherent in JAX-RS. For the cases in which you need to explicitly control the response sent back to the client, your JAX-RS resource methods can return instances of **javax.ws.rs.core.Response**:


```Java
public abstract class Response {

   public abstract Object getEntity();
   public abstract int getStatus();
   public abstract MultivaluedMap<String, Object> getMetadata();
...
}
```


The **Response** class is an abstract class that contains three simple methods. The **getEntity()** method returns the Java object you want converted into an HTTP message body. The **getStatus()** method returns the HTTP response code. The **getMetadata()** method is a **MultivaluedMap** of response headers.


**Response** objects cannot be created directly; instead, they are created from **javax.ws.rs.core.Response.ResponseBuilder** instances returned by one of the static helper methods of **Response**:


```Java
public abstract class Response {
...
   public static ResponseBuilder status(Status status) {...}
   public static ResponseBuilder status(int status) {...}
   public static ResponseBuilder ok() {...}
   public static ResponseBuilder ok(Object entity) {...}
   public static ResponseBuilder ok(Object entity, MediaType type) {...}
   public static ResponseBuilder ok(Object entity, String type) {...}
   public static ResponseBuilder ok(Object entity, Variant var) {...}
   public static ResponseBuilder serverError() {...}
   public static ResponseBuilder created(URI location) {...}
   public static ResponseBuilder noContent() {...}
   public static ResponseBuilder notModified() {...}
   public static ResponseBuilder notModified(EntityTag tag) {...}
   public static ResponseBuilder notModified(String tag) {...}
   public static ResponseBuilder seeOther(URI location) {...}
   public static ResponseBuilder temporaryRedirect(URI location) {...}
   public static ResponseBuilder notAcceptable(List<Variant> variants) {...}
   public static ResponseBuilder fromResponse(Response response) {...}
...
}
```

If you want an explanation of each and every static helper method, the JAX-RS Javadocs are a great place to look. They generally center on the most common use cases for creating custom responses. For example:


```Java
public static ResponseBuilder ok(Object entity, MediaType type) {...}
```


The **ok()** method here takes the Java object you want converted into an HTTP response and the **Content-Type** of that response. It returns a preinitialized **ResponseBuilder** with a status code of 200, “OK.” The other helper methods work in a similar way, setting appropriate response codes and sometimes setting up response headers automatically.


The **ResponseBuilder** class is a factory that is used to create one individual **Response** instance. You store up state you want to use to create your response and when you’re finished, you have the builder instantiate the **Response**:


```Java
public static abstract class ResponseBuilder {

   public abstract Response build();
   public abstract ResponseBuilder clone();

   public abstract ResponseBuilder status(int status);
   public ResponseBuilder status(Status status) {...}

   public abstract ResponseBuilder entity(Object entity);
   public abstract ResponseBuilder type(MediaType type);
   public abstract ResponseBuilder type(String type);

   public abstract ResponseBuilder variant(Variant variant);
   public abstract ResponseBuilder variants(List<Variant> variants);

   public abstract ResponseBuilder language(String language);
   public abstract ResponseBuilder language(Locale language);

   public abstract ResponseBuilder location(URI location);
   public abstract ResponseBuilder contentLocation(URI location);

   public abstract ResponseBuilder tag(EntityTag tag);
   public abstract ResponseBuilder tag(String tag);

   public abstract ResponseBuilder lastModified(Date lastModified);
   public abstract ResponseBuilder cacheControl(CacheControl cacheControl);

   public abstract ResponseBuilder expires(Date expires);
   public abstract ResponseBuilder header(String name, Object value);

   public abstract ResponseBuilder cookie(NewCookie... cookies);
}
```


As you can see, **ResponseBuilder** has a lot of helper methods for initializing various response headers. I don’t want to bore you with all the details, so check out the JAX-RS Javadocs for an explanation of each one. I’ll be giving examples using many of them throughout the rest of this book.


Now that we have a rough idea about creating custom responses, let’s look at an example of a JAX-RS resource method setting some specific response headers:

```Java
@Path("/textbook")
public class TextBookService {

   @GET
   @Path("/restfuljava")
   @Produces("text/plain")
   public Response getBook() {

       String book = ...;
       ResponseBuilder builder = Response.ok(book);
       builder.language("fr")
               .header("Some-Header", "some value");

       return builder.build();
   }
}
```


Here, our **getBook()** method is returning a plain-text string that represents a book our client is interested in. We initialize the response body using the **Response.ok()** method. The status code of the **ResponseBuilder** is automatically initialized with 200. Using the **ResponseBuilder.language()** method, we then set the **Content-Language** header to French. We then use the **ResponseBuilder.header()** method to set a custom response header. Finally, we create and return the Response object using the **ResponseBuilder.build()** method.


One interesting thing to note about this code is that we never set the **Content-Type** of the response. Because we have already specified an **@Produces** annotation, the JAX-RS runtime will set the media type of the response for us.


### Returning Cookies


JAX-RS also provides a simple class to represent new cookie values. This class is **javax.ws.rs.core.NewCookie**:


```Java
public class NewCookie extends Cookie {

   public static final int DEFAULT_MAX_AGE = −1;

   public NewCookie(String name, String value) {}

   public NewCookie(String name, String value, String path,
                     String domain, String comment,
                       int maxAge, boolean secure) {}

   public NewCookie(String name, String value, String path,
                     String domain, int version, String comment,
                      int maxAge, boolean secure) {}

   public NewCookie(Cookie cookie) {}

   public NewCookie(Cookie cookie, String comment,
                     int maxAge, boolean secure) {}

   public static NewCookie valueOf(String value)
                     throws IllegalArgumentException {}

   public String getComment() {}
   public int getMaxAge() {}
   public boolean isSecure() {}
   public Cookie toCookie() {}
}
```

The **NewCookie** class extends the **Cookie** class discussed in [Chapter 5](../chapter5/common_functionality.md). To set response cookies, create instances of **NewCookie** and pass them to the method **ResponseBuilder.cookie()**. For example:


```Java
@Path("/myservice")
public class MyService {

   @GET
   public Response get() {

       NewCookie cookie = new NewCookie("key", "value");
       ResponseBuilder builder = Response.ok("hello", "text/plain");
       return builder.cookie(cookie).build();
   }
```

Here, we’re just setting a cookie named key to the value value.


### The Status Enum


Generally, developers like to have constant variables represent raw strings or numeric values within. For instance, instead of using a numeric constant to set a **Response** status code, you may want a static final variable to represent a specific code. The JAX-RS specification provides a Java enum called **javax.ws.rs.core.Response.Status** for this very purpose:


```Java
public enum Status {
   OK(200, "OK"),
   CREATED(201, "Created"),
   ACCEPTED(202, "Accepted"),
   NO_CONTENT(204, "No Content"),
   MOVED_PERMANENTLY(301, "Moved Permanently"),
   SEE_OTHER(303, "See Other"),
   NOT_MODIFIED(304, "Not Modified"),
   TEMPORARY_REDIRECT(307, "Temporary Redirect"),
   BAD_REQUEST(400, "Bad Request"),
   UNAUTHORIZED(401, "Unauthorized"),
   FORBIDDEN(403, "Forbidden"),
   NOT_FOUND(404, "Not Found"),
   NOT_ACCEPTABLE(406, "Not Acceptable"),
   CONFLICT(409, "Conflict"),
   GONE(410, "Gone"),
   PRECONDITION_FAILED(412, "Precondition Failed"),
   UNSUPPORTED_MEDIA_TYPE(415, "Unsupported Media Type"),
   INTERNAL_SERVER_ERROR(500, "Internal Server Error"),
   SERVICE_UNAVAILABLE(503, "Service Unavailable");

   public enum Family {
         INFORMATIONAL, SUCCESSFUL, REDIRECTION,
         CLIENT_ERROR, SERVER_ERROR, OTHER
   }

   public Family getFamily()

   public int getStatusCode()

   public static Status fromStatusCode(final int statusCode)
}
```


Each **Status** enum value is associated with a specific family of HTTP response codes. These families are identified by the **Status.Family** Java enum. Codes in the 100 range are considered *informational*. Codes in the 200 range are considered *successful*. Codes in the 300 range are success codes, but fall under the *redirection* category. Error codes are in the 400 to 500 ranges. The 400s are *client errors* and 500s are *server errors*.


Both the **Response.status()** and **ResponseBuilder.status()** methods can accept a **Status** enum value. For example:


```Java
@DELETE
Response delete() {
   ...

   return Response.status(Status.GONE).build();
}
```


Here, we’re telling the client that the thing we want to delete is already gone (410).


### javax.ws.rs.core.GenericEntity


When we’re dealing with returning **Response** objects, we do have a problem with **MessageBodyWriters** that are sensitive to generic types. For example, what if our built-in JAXB **MessageBodyWriter** can handle lists of JAXB objects? The **isWriteable()** method of our JAXB handler needs to extract parameterized type information of the generic type of the response entity. Unfortunately, there is no easy way in Java to obtain generic type information at runtime. To solve this problem, JAX-RS provides a helper class called **javax.ws.rs.core.GenericEntity**. This is best explained with an example:


```Java
@GET
@Produces("application/xml")
public Response getCustomerList() {
    List<Customer> list = new ArrayList<Customer>();
    list.add(new Customer(...));

    GenericEntity entity = new GenericEntity<List<Customer>>(list){};
    return Response.ok(entity).build();
}
```


The **GenericEntity** class is a Java generic template. What you do here is create an anonymous class that extends **GenericEntity**, initializing the **GenericEntity**’s template with the generic type you’re using. If this looks a bit magical, it is. The creators of Java generics made things a bit difficult, so we’re stuck with this solution.

