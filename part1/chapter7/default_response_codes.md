# Default Response Codes


The default response codes that JAX-RS uses are pretty straightforward. There is pretty much a one-to-one relationship to the behavior described in the HTTP 1.1 Method Definition specification[^7]. Let’s examine what the response codes would be for both success and error conditions for the following JAX-RS resource class:


```Java
@Path("/customers")
public class CustomerResource {

   @Path("{id}")
   @GET
   @Produces("application/xml")
   public Customer getCustomer(@PathParam("id") int id) {...}

   @POST
   @Produces("application/xml")
   @Consumes("application/xml")
   public Customer create(Customer newCust) {...}

   @PUT
   @Path("{id}")
   @Consumes("application/xml")
   public void update(@PathParam("id") int id, Customer cust) {...}

   @Path("{id}")
   @DELETE
   public void delete(@PathParam("id") int id) {...}
}
```


### Successful Responses


Successful HTTP response code numbers range from 200 to 399. For the **create()** and **getCustomer()** methods of our **CustomerResource** class, they will return a response code of 200, “OK,” if the **Customer** object they are returning is not null. If the return value is null, a successful response code of 204, “No Content,” is returned. The 204 response is not an error condition. It just tells the client that everything went OK, but that there is no message body to look for in the response. If the JAX-RS resource method’s return type is void, a response code of 204, “No Content,” is returned. This is the case with our **update()** and **delete()** methods.


The HTTP specification is pretty consistent for the PUT, POST, GET, and DELETE methods. If a successful HTTP response contains a message body, 200, “OK,” is the response code. If the response doesn’t contain a message body, 204, “No Content,” must be returned.



### Error Responses


In our **CustomerResource** example, error responses are mostly driven by application code throwing an exception. We will discuss this exception handling later in this chapter. There are some default error conditions that we can talk about right now, though.


Standard HTTP error response code numbers range from 400 to 599. In our example, if a client mistypes the request URI, for example, to **customers**, it will result in the server not finding a JAX-RS resource method that can service the request. In this case, a 404, “Not Found,” response code will be sent back to the client.


For our **getCustomer()** and **create()** methods, if the client requests a text/html response, the JAX-RS implementation will automatically return a 406, “Not Acceptable,” response code with no response body. This means that JAX-RS has a relative URI path that matches the request, but doesn’t have a JAX-RS resource method that can produce the client’s desired response media type. ([Chapter 9](../chapter9/http_content_negotiation.md) talks in detail about how clients can request certain formats from the server.)


If the client invokes an HTTP method on a valid URI to which no JAX-RS resource method is bound, the JAX-RS runtime will send an error code of 405, “Method Not Allowed.” So, in our example, if our client does a PUT, GET, or DELETE on the **/customers** URI, it will get a 405 response because POST is the only supported method for that URI. The JAX-RS implementation will also return an **Allow** response header back to the client that contains a list of HTTP methods the URI supports. So, if our client did a **GET /customers** in our example, the server would send this response back:


```
HTTP/1.1 405, Method Not Allowed
Allow: POST
```

The exception to this rule is the HTTP HEAD and OPTIONS methods. If a JAX-RS resource method isn’t available that can service HEAD requests for that particular URI, but there does exist a method that can handle GET, JAX-RS will invoke the JAX-RS resource method that handles GET and return the response from that minus the request body. If there is no existing method that can handle OPTIONS, the JAX-RS implementation is required to send back some meaningful, automatically generated response along with the **Allow** header set.


---
[^7] For more information, see http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html