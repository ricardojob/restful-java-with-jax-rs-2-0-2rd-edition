# Gotchas in Request Matching


There are some fine print details about the URI request matching algorithm that I must go over, as there may be cases where you’d expect a request to match and it doesn’t. First of all, the specification requires that potential JAX-RS class matches are filtered first based on the root **@Path** annotation. Consider the following two classes:


```Java
@Path("/a")
public class Resource1 {
   @GET
   @Path("/b")
   public Response get() {}
}

@Path("/{any : .*}")
public class Resource2 {

   @GET
   public Response get() {}

   @OPTIONS
   public Response options() {}
}
```


If we have an HTTP request **GET /a/b**, the matching algorithm will first find the best class that matches before finishing the full dispatch. In this case, class **Resource1** is chosen because its **@Path("/a")** annotation best matches the initial part of the request URI. The matching algorithm then tries to match the remainder of the URI based on expressions contained in the **Resource1** class.


Here’s where the weirdness comes in. Let’s say you have the HTTP request **OPTIONS /a/b**. If you expect that the **Resource2.options()** method would be invoked, you would be wrong! You would actually get a 405, “Method Not Allowed,” error response from the server. This is because the initial part of the request path, **/a**, matches the **Resource1** class best, so **Resource1** is used to resolve the rest of the HTTP request. If we change **Resource2** as follows, the request would be processed by the **options()** method:


```Java
@Path("/a")
public class Resource2 {


   @OPTIONS
   @Path("b")
   public Response options() {}
}
```

If the **@Path** expressions are the same between two different JAX-RS classes, then they both are used for request matching.


There are also similar ambiguities in subresource locator matching. Take these classes, for example:


```Java
@Path("/a")
public class Foo {
   @GET
   @Path("b")
   public String get() {...}

   @Path("{id}")
   public Locator locator() { return new Locator(); }
}

public class Locator{
   @PUT
   public void put() {...}
}
```


If we did a **PUT /a/b** request, you would also get a 405 error response. The specification algorithm states that if there is at least one other resource method whose **@Path** expression matches, then no subresource locator will be traversed to match the request.


In most applications, you will not encounter these maching issues, but it’s good to know about them just in case you do. I tried to get these problems fixed in the JAX-RS 2.0 spec, but a few JSR members thought that this would break backward compatibility.