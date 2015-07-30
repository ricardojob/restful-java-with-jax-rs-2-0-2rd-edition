# @Path


There’s more to the **@javax.ws.rs.Path** annotation than what we saw in our simple example in [Chapter 3](../chapter3/your_first_jax_rs_service.md). **@Path** can have complex matching expressions so that you can be more specific about what requests get bound to which incoming URIs. **@Path** can also be used on a Java method as sort of an object factory for subresources of your application. We’ll examine both in this section.


### Binding URIs


The **@javax.ws.rs.Path** annotation in JAX-RS is used to define a URI matching pattern for incoming HTTP requests. It can be placed upon a class or on one or more Java methods. For a Java class to be eligible to receive any HTTP requests, the class must be annotated with at least the **@Path("/")** expression. These types of classes are called JAX-RS *root resources*.


The value of the **@Path** annotation is an expression that denotes a relative URI to the context root of your JAX-RS application. For example, if you are deploying into a WAR archive of a servlet container, that WAR will have a base URI that browsers and remote clients use to access it. **@Path** expressions are relative to this URI.


To receive a request, a Java method must have at least an HTTP method annotation like **@javax.ws.rs.GET** applied to it. This method is not required to have an **@Path** annotation on it, though. For example:






