# @HeaderParam


The **@javax.ws.rs.HeaderParam** annotation is used to inject HTTP request header values. For example, what if your application was interested in the web page that referred to or linked to your web service? You could access the HTTP **Referer** header using the **@HeaderParam** annotation:


```Java
@Path("/myservice")
public class MyService {

   @GET
   @Produces("text/html")
   public String get(@HeaderParam("Referer") String referer) {
     ...
   }
}
```


The **@HeaderParam** annotation is pulling the **Referer** header directly from the HTTP request and injecting it into the referer method parameter.


#### Raw Headers


Sometimes you need programmatic access to view all headers within the incoming request. For instance, you may want to log them. The JAX-RS specification provides the **javax.ws.rs.core.HttpHeaders** interface for such scenarios.


```Java
public interface HttpHeaders {
   public List<String> getRequestHeader(String name);
   public MultivaluedMap<String, String> getRequestHeaders();
...
}
```


The **getRequestHeader()** method allows you to get access to one particular header, and **getRequestHeaders()** gives you a map that represents all headers.

As with **UriInfo**, you can use the **@Context** annotation to obtain an instance of **HttpHeaders**. Here’s an example:


```Java
@Path("/myservice")
public class MyService {

   @GET
   @Produces("text/html")
   public String get(@Context HttpHeaders headers) {
      String referer = headers.getRequestHeader("Referer").get(0);
      for (String header : headers.getRequestHeaders().keySet())
      {
         System.out.println("This header was set: " + header);
      }
     ...
   }
}
```








