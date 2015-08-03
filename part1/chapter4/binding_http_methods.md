# Binding HTTP Methods


JAX-RS defines five annotations that map to specific HTTP operations:

* **@javax.ws.rs.GET**
* **@javax.ws.rs.PUT**
* **@javax.ws.rs.POST**
* **@javax.ws.rs.DELETE**
* **@javax.ws.rs.HEAD**


In [Chapter 3](../chapter3/your_first_jax_rs_service.md), we used these annotations to bind HTTP GET requests to a specific Java method. For example:

```Java
@Path("/customers")
public class CustomerService {

   @GET
   @Produces("application/xml")
   public String getAllCustomers() {
   }
}
```


Here we have a simple method, **getAllCustomers()**. The **@GET** annotation instructs the JAX-RS runtime that this Java method will process HTTP GET requests to the URI **/customers**. You would use one of the other five annotations described earlier to bind to different HTTP operations. One thing to note, though, is that you may only apply one HTTP method annotation per Java method. A deployment error occurs if you apply more than one.


Beyond simple binding, there are some interesting things to note about the implementation of these types of annotations. Let’s take a look at **@GET**, for instance:

```Java
package javax.ws.rs;

import ...;

@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@HttpMethod(HttpMethod.GET)
public @interface GET {
}
```

**@GET**, by itself, does not mean anything special to the JAX-RS provider. In other words, JAX-RS is not hardcoded to look for this annotation when deciding whether or not to dispatch an HTTP GET request. What makes the **@GET** annotation meaningful to a JAX-RS provider is the meta-annotation **@javax.ws.rs.HttpMethod**. Meta-annotations are simply annotations that annotate other annotations. When the JAX-RS provider examines a Java method, it looks for any method annotations that use the meta-annotation **@HttpMethod**. The value of this meta-annotation is the actual HTTP operation that you want your Java method to bind to.



### HTTP Method Extensions

What are the implications of this? This means that you can create new annotations that bind to HTTP methods other than GET, POST, DELETE, HEAD, and PUT. While HTTP is a ubiquitous, stable protocol, it is still constantly evolving. For example, consider the WebDAV standard.[^3] The WebDAV protocol makes the Web an interactive readable and writable medium. It allows users to create, change, and move documents on web servers. It does this by adding a bunch of new methods to HTTP like MOVE, COPY, MKCOL, LOCK, and UNLOCK.


Although JAX-RS does not define any WebDAV-specific annotations, we could create them ourselves using the **@HttpMethod** annotation:


```Java
package org.rest.webdav;

import ...;

@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@HttpMethod("LOCK")
public @interface LOCK {
}
```


Here, we have defined a new **@org.rest.LOCK** annotation using **@HttpMethod** to specify the HTTP operation it binds to. We can then use it on JAX-RS resource methods:


```Java
@Path("/customers")
public class CustomerResource {

   @Path("{id}")
   @LOCK
   public void lockIt(@PathParam("id") String id) {
      ...
   }
}
```


Now WebDAV clients can invoke **LOCK** operations on our web server and they will be dispatched to the **lockIt()** method.



> #### Warning
> Do not use **@HttpMethod** to define your own application-specific HTTP methods. **@HttpMethod** exists to hook into new methods defined by standards bodies like the W3C. The purpose of the uniform interface is to define a set of well-known behaviors across companies and organizations on the Web. Defining your own methods breaks this architectural principle.

---
[^3] For more information on WebDAV, see http://www.webdav.org.











