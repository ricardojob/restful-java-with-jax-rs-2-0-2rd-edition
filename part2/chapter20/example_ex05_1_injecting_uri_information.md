# Example ex05_1: Injecting URI Information


This example illustrates the injection annotations that are focused on pulling in information from the incoming request URI. Specifically, it shows how to use **@PathParam**, **@MatrixParam**, and **@QueryParam**. Parallel examples are also shown using **javax.ws.rs.core.UriInfo** to obtain the same data.


### The Server Code


The first thing you should look at on the server side is **CarResource**. This class pulls the various examples in [Chapter 5](../../part1/chapter5/jax_rs_injection.md) together to illustrate using **@MatrixParam** and **@PathParam** with the **javax.ws.rs.core.PathSegment** class:


```Java:src/main/java/com/restfully/shop/services/CarResource.java
@Path("/cars")
public class CarResource
{
   public static enum Color
   {
      red,
      white,
      blue,
      black
   }

   @GET
   @Path("/matrix/{make}/{model}/{year}")
   @Produces("text/plain")
   public String getFromMatrixParam(
                             @PathParam("make") String make,
                             @PathParam("model") PathSegment car,
                             @MatrixParam("color") Color color,
                             @PathParam("year") String year)
   {
      return "A " + color + " " + year + " "
                  + make + " " + car.getPath();
   }
```


The **getFromMatrixParam()** method uses the **@MatrixParam** annotation to inject the matrix parameter **color**. An example of a URI it could process is **/cars/matrix/mercedes/e55;color=black/2006**. Notice that it automatically converts the matrix parameter into the Java enum **Color**:


```Java
   @GET
   @Path("/segment/{make}/{model}/{year}")
   @Produces("text/plain")
   public String getFromPathSegment(@PathParam("make") String make,
                                @PathParam("model") PathSegment car,
                               @PathParam("year") String year)
   {
      String carColor = car.getMatrixParameters().getFirst("color");
      return "A " + carColor + " " + year + " "
                             + make + " " + car.getPath();
   }
```


The **getFromPathSegment()** method also illustrates how to extract matrix parameter information. Instead of using **@MatrixParam**, it uses an injected **PathSegment** instance representing the **model** path parameter to obtain the matrix parameter information:


```Java
   @GET
   @Path("/segments/{make}/{model : .+}/year/{year}")
   @Produces("text/plain")
   public String getFromMultipleSegments(
                             @PathParam("make") String make,
                           @PathParam("model") List<PathSegment> car,
                         @PathParam("year") String year)
   {
      String output = "A " + year + " " + make;
      for (PathSegment segment : car)
      {
         output += " " + segment.getPath();
      }
      return output;
   }
```


The **getFromMultipleSegments()** method illustrates how a path parameter can match multiple segments of a URI. An example of a URI that it could process is **/cars/segments/mercedes/e55/amg/year/2006**. In this case, **e55/amg** would match the **model** path parameter. The example injects the **model** parameter into a list of **PathSegment** instances:


```Java
   @GET
   @Path("/uriinfo/{make}/{model}/{year}")
   @Produces("text/plain")
   public String getFromUriInfo(@Context UriInfo info)
   {
      String make = info.getPathParameters().getFirst("make");
      String year = info.getPathParameters().getFirst("year");
      PathSegment model = info.getPathSegments().get(3);
      String color = model.getMatrixParameters().getFirst("color");

      return "A " + color + " " + year + " "
                  + make + " " + model.getPath();
   }
```


The final method, **getFromUriInfo()**, shows how you can obtain the same information using the **UriInfo** interface. As you can see, the matrix parameter information is extracted from **PathSegment** instances.


The next piece of code you should look at on the server is **CustomerResource**. This class shows how **@QueryParam** and **@DefaultValue** can work together to obtain information about the request URI’s query parameters. An example using **UriInfo** is also shown so that you can see how this can be done without injection annotations:



```Java:src/main/java/com/restfully/shop/services/CustomerResource.java
@Path("/customers")
public class CustomerResource {
...

   @GET
   @Produces("application/xml")
   public StreamingOutput getCustomers(
               final @QueryParam("start") int start,
               final @QueryParam("size") @DefaultValue("2") int size)
   {
     ...
   }
```


The **getCustomers()** method returns a set of customers from the customer database. The **start** parameter defines the start index and the **size** parameter specifies how many customers you want returned. The **@DefaultValue** annotation is used for the case in which a client does not use the query parameters to index into the customer list.



The next implementation of **getCustomers()** uses **UriInfo** instead of injection parameters:


```Java
   @GET
   @Produces("application/xml")
   @Path("uriinfo")
   public StreamingOutput getCustomers(@Context UriInfo info)
   {
      int start = 0;
      int size = 2;
      if (info.getQueryParameters().containsKey("start"))
      {
         start = Integer.valueOf(
                        info.getQueryParameters().getFirst("start"));
      }
      if (info.getQueryParameters().containsKey("size"))
      {
         size = Integer.valueOf(
                        info.getQueryParameters().getFirst("size"));
      }
      return getCustomers(start, size);
   }
```


As you can see, the code to access query parameter data programmatically is a bit more verbose than using injection annotations.


### The Client Code


The client code for this example lives in the file *src/test/java/com/restfully/shop/test/InjectionTest.java*. The code is quite boring, so I won’t get into too much detail.


The **testCarResource()** method invokes these requests on the server to test the **CarResource** class:


```
GET http://localhost:8080/services/cars/matrix/mercedes/e55;color=black/2006
GET http://localhost:8080/services/cars/segment/mercedes/e55;color=black/2006
GET http://localhost:8080/services/cars/segments/mercedes/e55/amg/year/2006
GET http://localhost:8080/services/cars/uriinfo/mercedes/e55;color=black/2006
```


The **testCustomerResource()** method invokes these requests on the server to test the **CustomerResource** class:


```
GET http://localhost:8080/services/customers
GET http://localhost:8080/services/customers?start=1&size=3
GET http://localhost:8080/services/customers/uriinfo?start=2&size=2
```


The request without query parameters shows **@DefaultValue** in action. It is worth noting how query parameters are handled in the client code. Let’s look at **testCustomerResource()** a little bit:


```Java
      list = client.target("http://localhost:8080/services/customers/uriinfo")
                   .queryParam("start", "2")
                   .queryParam("size", "2")
                   .request().get(String.class);
```


The **javax.ws.rs.client.WebTarget.queryParam()** method is used to fill out the query parameters for the invocation. This is a nice convenience method to use, especially if your values might have characters that need to be encoded.



### Build and Run the Example Program


Perform the following steps:

1. Open a command prompt or shell terminal and change to the *ex05_1* directory of the workbook example code.
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md).
3. Perform the build and run the example by typing **maven install**. 

