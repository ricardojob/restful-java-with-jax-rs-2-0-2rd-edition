# Server-Side Filters


On the server side there are two different types of filters: *request filters* and *response filters*. Request filters execute before a JAX-RS method is invoked. Response filters execute after the JAX-RS method is finished. By default they are executed for all HTTP requests, but can be bound to a specific JAX-RS method too. Internally, the algorithm for executing an HTTP on the server side looks something like this:



```Java
for (filter : preMatchFilters) {
   filter.filter(request);
}

jaxrs_method = match(request);

for (filter : postMatchFilters) {
   filter.filter(request);
}

response = jaxrs_method.invoke();

for (filter : responseFilters) {
   filter.filter(request, response);
}
```


For those of you familiar with the Servlet API, JAX-RS filters are quite different. JAX-RS breaks up its filters into separate request and response interfaces, while servlet filters wrap around servlet processing and are run in the same Java call stack. Because JAX-RS has an asynchronous API, JAX-RS filters cannot run in the same Java call stack. Each request filter runs to completion before the JAX-RS method is invoked. Each response filter runs to completion only after a response becomes available to send back to the client. In the asynchronous case, response filters run after **resume()**, **cancel()**, or a timeout happens. See [Chapter 13](../chapter13/async_invoker_client_api.md) for more details on the asynchronous API.



### Server Request Filters



Request filters are implementations of the **ContainerRequestFilter** interface:


```Java
package javax.ws.rs.container;

public interface ContainerRequestFilter {
    public void filter(ContainerRequestContext requestContext)
                      throws IOException;
}
```


**ContainerRequestFilters** come in two flavors: prematching and postmatching. Prematching **ContainerRequestFilters** are designated with the **@PreMatching** annotation and will execute before the JAX-RS resource method is matched with the incoming HTTP request. Prematching filters often are used to modify request attributes to change how they match to a specific resource. For example, some firewalls do not allow PUT and/or DELETE invocations. To circumvent this limitation, many applications tunnel the HTTP method through the HTTP header **X-Http-Method-Override**:



```Java
import javax.ws.rs.container.ContainerRequestFilter;
import javax.ws.rs.container.ContainerRequestContext;

@Provider
@PreMatching
public class HttpMethodOverride implements ContainerRequestFilter {
   public void filter(ContainerRequestContext ctx) throws IOException {
      String methodOverride = ctx.getHeaderString("X-Http-Method-Override");
      if (methodOverride != null) ctx.setMethod(methodOverride);
   }
}
```


This **HttpMethodOverride** filter will run before the HTTP request is matched to a specific JAX-RS method. The **ContainerRequestContext** parameter passed to the **filter()** method provides information about the request like headers, the URI, and so on. The **filter()** method uses the **ContainerRequestContext** parameter to check the value of the **X-Http-Method-Override** header. If the header is set in the request, the filter overrides the request’s HTTP method by calling **ContainerRequestFilter.setMethod()**. Filters can modify pretty much anything about the incoming request through methods on **ContainerRequestContext**, but once the request is matched to a JAX-RS method, a filter cannot modify the request URI or HTTP method.



Another great use case for request filters is implementing custom authentication protocols. For example, OAuth 2.0 has a token protocol that is transmitted through the **Authorization** HTTP header. Here’s what an implementation of that might look like:



```Java
import javax.ws.rs.container.ContainerRequestFilter;
import javax.ws.rs.container.ContainerRequestContext;
import javax.ws.rs.NotAuthorizedException;

@Provider
@PreMatching
public class BearerTokenFilter implements ContainerRequestFilter {
   public void filter(ContainerRequestContext ctx) throws IOException {
      String authHeader = request.getHeaderString(HttpHeaders.AUTHORIZATION);
      if (authHeader == null) throw new NotAuthorizedException("Bearer");
      String token = parseToken(authHeader);
      if (verifyToken(token) == false) {
         throw new NotAuthorizedException("Bearer error=\"invalid_token\"");
      }
   }

   private String parseToken(String header) {...}
   private boolean verifyToken(String token) {...}
}
```


In this example, if there is no **Authorization** header or it is invalid, the request is aborted with a **NotAuthorizedException**. The client receives a 401 response with a **WWW-Authenticate** header set to the value passed into the constructor of **NotAuthorizedException**. If you want to avoid exception mapping, then you can use the **ContainerRequestContext.abortWith()** method instead. Generally, however, I prefer to throw exceptions.



### Server Response Filters



Response filters are implementations of the **ContainerResponseFilter** interface:


```Java
package javax.ws.rs.container;

public interface ContainerResponseFilter {
    public void filter(ContainerRequestContext requestContext,
                       ContainerResponseContext responseContext)
            throws IOException;
}
```


Generally, you use these types of filters to decorate the response by adding or modifying response headers. One example is if you wanted to set a default **Cache-Control** header for each response to a GET request. Here’s what it might look like:


```Java
import javax.ws.rs.container.ContainerResponseFilter;
import javax.ws.rs.container.ContainerRequestContext;
import javax.ws.rs.container.ContainerResponseContext;
import javax.ws.rs.core.CacheControl;

@Provider
public class CacheControlFilter implements ContainerResponseFilter {
   public void filter(ContainerRequestContext req, ContainerResponseContext res)
        throws IOException
   {
      if (req.getMethod().equals("GET")) {
         CacheControl cc = new CacheControl();
         cc.setMaxAge(100);
         req.getHeaders().add("Cache-Control", cc);
      }
   }
}
```


The **ContainerResponseFilter.filter()** method has two parameters. The **ContainerRequestContext** parameter gives you access to information about the request. Here we’re checking to see if the request was a GET. The **ContainerResponseContext** parameter allows you to view, add, and modify the response before it is marshalled and sent back to the client. In the example, we use the **ContainerResponseContext** to set a **Cache-Control** response header.




