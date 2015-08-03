# The Basics


There are a lot of different things JAX-RS annotations can inject. Here is a list of those provided by the specification:


* @javax.ws.rs.PathParam

    This annotation allows you to extract values from URI template parameters.
    
* @javax.ws.rs.MatrixParam

    This annotation allows you to extract values from URI matrix parameters. 

* @javax.ws.rs.QueryParam 

    This annotation allows you to extract values from URI query parameters. 

* @javax.ws.rs.FormParam 

    This annotation allows you to extract values from posted form data. 

* @javax.ws.rs.HeaderParam

    This annotation allows you to extract values from HTTP request headers. 

* @javax.ws.rs.CookieParam 

    This annotation allows you to extract values from HTTP cookies set by the client. 

* @javax.ws.rs.core.Context 

    This class is the all-purpose injection annotation. It allows you to inject various helper and informational objects that are provided by the JAX-RS API. 


Usually, these annotations are used on the parameters of a JAX-RS resource method. When the JAX-RS provider receives an HTTP request, it finds a Java method that will service this request. If the Java method has parameters that are annotated with any of these injection annotations, it will extract information from the HTTP request and pass it as a parameter when it invokes the method.


For per-request resources, you may alternatively use these injection annotations on the fields, setter methods, and even constructor parameters of your JAX-RS resource class. Do not try to use these annotations on fields or setter methods if your component model does not follow per-request instantiation. Singletons process HTTP requests concurrently, so it is not possible to use these annotations on fields or setter methods, as concurrent requests will overrun and conflict with each other.