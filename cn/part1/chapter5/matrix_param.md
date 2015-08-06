# @MatrixParam


Instead of injecting and processing **PathSegment** objects to obtain matrix parameter values, the JAX-RS specification allows you to inject matrix parameter values directly through the **@javax.ws.rs.MatrixParam** annotation. Let’s change our **CarResource** example from the previous section to reflect using this annotation:


```Java
@Path("/{make}")
public class CarResource {

   @GET
   @Path("/{model}/{year}")
   @Produces("image/jpeg")
   public Jpeg getPicture(@PathParam("make") String make,
                          @PathParam("model") String model,
                          @MatrixParam("color") String color) {
      ...
   }
```


Using the **@MatrixParam** annotation shrinks our code and provides a bit more readability. The only downside of **@MatrixParam** is that sometimes you might have a repeating matrix parameter that is applied to many different path segments in the URI. For example, what if **color** shows up multiple times in our car service example?


```
GET /mercedes/e55;color=black/2006/interior;color=tan
```


Here, the **color** attribute shows up twice: once with the model and once with the interior. Using **@MatrixParam("color")** in this case would be ambiguous and we would have to go back to processing **PathSegments** to obtain this matrix parameter.


