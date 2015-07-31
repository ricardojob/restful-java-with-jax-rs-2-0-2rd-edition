# @PathParam


We looked at **@javax.ws.rs.PathParam** a little bit in Chapters [3](../chapter3/developing_a_jax_rs_restful_service.md) and [4](../chapter4/subresource_locators.md). **@PathParam** allows you to inject the value of named URI path parameters that were defined in **@Path** expressions. Let’s revisit the **CustomerResource** example that we defined in [Chapter 2](../chapter2/designing_restful_services.md) and implemented in [Chapter 3](../chapter3/developing_a_jax_rs_restful_service.md):


```Java
@Path("/customers")
public class CustomerResource {
   ...

   @Path("{id}")
   @GET
   @Produces("application/xml")
   public StreamingOutput getCustomer(@
PathParam("id") int id) {
     ...
   }
}
```


### More Than One Path Parameter


You can reference more than one URI path parameter in your Java methods. For instance, let’s say we are using first and last name to identify a customer in our **CustomerResource**:

```Java
@Path("/customers")
public class CustomerResource {
   ...

   @Path("{first}-{last}")
   @GET
   @Produces("application/xml")
   public StreamingOutput getCustomer(@PathParam("first") String firstName,
                                      @PathParam("last") String lastName) {
     ...
   }
}
```


### Scope of Path Parameters


Sometimes a named URI path parameter will be repeated by different **@Path** expressions that compose the full URI matching pattern of a resource method. The path parameter could be repeated by the class’s **@Path** expression or by a subresource locator. In these cases, the **@PathParam** annotation will always reference the final path parameter. For example:


```Java
@Path("/customers/{id}")
public class CustomerResource {

   @Path("/address/{id}")
   @Produces("text/plain")
   @GET
   public String getAddress(@PathParam("id") String addressId) {...}
}
```


If our HTTP request was **GET /customers/123/address/456**, the **addressId** parameter in the **getAddress()** method would have the **456** value injected.


### PathSegment and Matrix Parameters


**@PathParam** can not only inject the value of a path parameter, it can also inject instances of **javax.ws.rs.core.PathSegment**. The **PathSegment** class is an abstraction of a specific URI path segment:


```Java
package javax.ws.rs.core;

public interface PathSegment  {

   String getPath();
   MultivaluedMap<String, String> getMatrixParameters();

}
```


The **getPath()** method is the string value of the actual URI segment minus any matrix parameters. The more interesting method here is **getMatrixParameters()**. This returns a map of all of the matrix parameters applied to a particular URI segment. In combination with **@PathParam**, you can get access to the matrix parameters applied to your request’s URI. For example:


```Java
@Path("/cars/{make}")
public class CarResource {

   @GET
   @Path("/{model}/{year}")
   @Produces("image/jpeg")
   public Jpeg getPicture(@PathParam("make") String make,
                           @PathParam("model") PathSegment car,
                             @PathParam("year") String year) {
      String carColor = car.getMatrixParameters().getFirst("color");
      ...
   }
```


In this example, we have a **CarResource** that allows us to get pictures of cars in our database. The **getPicture()** method returns a JPEG image of cars that match the make, model, and year that we specify. The color of the vehicle is expressed as a matrix parameter of the model. For example:


```
GET /cars/mercedes/e55;color=black/2006
```


Here, our make is **mercedes**, the model is **e55** with a color attribute of **black**, and the year is **2006**. While the make, model, and year information can be injected into our **getPicture()** method directly, we need to do some processing to obtain information about the color of the vehicle.


Instead of injecting the model information as a Java string, we inject the path parameter as a **PathSegment** into the **car** parameter. We then use this **PathSegment** instance to obtain the **color** matrix parameter’s value.


#### Matching with multiple PathSegments


Sometimes a particular path parameter matches to more than one URI segment. In these cases, you can inject a list of **PathSegments**. For example, let’s say a model in our **CarResource** could be represented by more than one URI segment. Here’s how the **getPicture()** method might change:


```Java
@Path("/cars/{make}")
public class CarResource {

   @GET
   @Path("/{model : .+}/year/{year}")
   @Produces("image/jpeg")
   public Jpeg getPicture(@PathParam("make") String make,
                           @PathParam("model") List<PathSegment> car,
                            @PathParam("year") String year) {
   }
}
```


In this example, if our request was **GET /cars/mercedes/e55/amg/year/2006**, the car parameter would have a list of two **PathSegments** injected into it, one representing the **e55** segment and the other representing the **amg** segment. We could then query and pull in matrix parameters as needed from these segments.















