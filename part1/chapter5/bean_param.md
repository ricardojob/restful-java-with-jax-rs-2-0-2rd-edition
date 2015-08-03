# @BeanParam


The **@BeanParam** annotation is something new added in the JAX-RS 2.0 specification. It allows you to inject an application-specific class whose property methods or fields are annotated with any of the injection parameters discussed in this chapter. For example, take this class:


```Java
public class CustomerInput {

   @FormParam("first")
   String firstName;

   @FormParam("list")
   String lastName;

   @HeaderParam("Content-Type")
   String contentType;

   public String getFirstName() {...}
   ....
}
```


Here we have a simple POJO (Plain Old Java Object) that contains the first and last names of a created customer, as well as the content type of that customer. We can have JAX-RS create, initialize, and inject this class using the **@BeanParam** annotation:


```Java
@Path("/customers")
public class CustomerResource {

   @POST
   public void createCustomer(@BeanParam CustomerInput newCust) {
     ...
   }
}
```


The JAX-RS runtime will introspect the **@BeanParam** parameter’s type for injection annotations and then set them as appropriate. In this example, the **CustomerInput** class is interested in two form parameters and a header value. This is a great way to aggregate information instead of having a long list of method parameters.




