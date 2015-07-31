# @QueryParam


The **@javax.ws.rs.QueryParam** annotation allows you to inject individual URI query parameters into your Java parameters. For example, let’s say we wanted to query a customer database and retrieve a subset of all customers in the database. Our URI might look like this:

```
GET /customers?start=0&size=10
```


The **start** query parameter represents the customer index we want to start with and the **size** query parameter represents how many customers we want returned. The JAX-RS service that implemented this might look like this:


```Java
@Path("/customers")
public class CustomerResource {

   @GET
   @Produces("application/xml")
   public String getCustomers(@QueryParam("start") int start,
                               @QueryParam("size") int size) {
     ...
   }
}
```

Here, we use the **@QueryParam** annotation to inject the URI query parameters **"start"** and **"size"** into the Java parameters **start** and **size**. As with other annotation injection, JAX-RS automatically converts the query parameter’s string into an integer.



#### Programmatic Query Parameter Information


You may have the need to iterate through all query parameters defined on the request URI. The **javax.ws.rs.core.UriInfo** interface has a **getQueryParameters()** method that gives you a map containing all query parameters:


```Java
public interface UriInfo {
...
   public MultivaluedMap<String, String> getQueryParameters();
   public MultivaluedMap<String, String> getQueryParameters(boolean decode);
...
}
```


You can inject instances of **UriInfo** using the **@javax.ws.rs.core.Context** annotation. Here’s an example of injecting this class and using it to obtain the value of a few query parameters:


```Java
@Path("/customers")
public class CustomerResource {

   @GET
   @Produces("application/xml")
   public String getCustomers(@Context UriInfo info) {
      String start = info.getQueryParameters().getFirst("start");
      String size = info.getQueryParameters().getFirst("size");
     ...
   }
}
```





