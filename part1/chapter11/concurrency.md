# Concurrency


Now that we have a good idea of how to boost the performance of our JAX-RS services using HTTP caching, we need to look at how to scale applications that update resources on our server. The way RESTful updates work is that the client fetches a representation of a resource through a GET request. It then modifies the representation locally and PUTs or POSTs the modified representation back to the server. This is all fine and dandy if there is only one client at a time modifying the resource, but what if the resource is being modified concurrently? Because the client is working with a snapshot, this data could become stale if another client modifies the resource while the snapshot is being processed.



The HTTP specification has a solution to this problem through the use of conditional PUTs or POSTs. This technique is very similar to how cache revalidation and conditional GETs work. The client first starts out by fetching the resource. For example, let’s say our client wants to update a customer in a RESTful customer directory. It would first start off by submitting **GET /customers/123** to pull down the current representation of the specific customer it wants to update. The response might look something like this:


```xml
HTTP/1.1 200 OK
Content-Type: application/xml
Cache-Control: max-age=1000
ETag: "3141271342554322343200"
Last-Modified: Tue, 15 May 2013 09:56 EST

<customer id="123">...</customer>
```


In order to do a conditional update, we need either an **ETag** or **Last-Modified** header. This information tells the server which snapshot version we have modified when we perform our update. It is sent along within the **If-Match** or **If-Unmodified-Since** header when we do our PUT or POST request. The **If-Match header** is initialized with the **ETag** value of the snapshot. The **If-Unmodified-Since** header is initialized with the value of **Last-Modified** header. So, our update request might look like this:



```xml
PUT /customers/123 HTTP/1.1
If-Match: "3141271342554322343200"
If-Unmodified-Since: Tue, 15 May 2013 09:56 EST
Content-Type: application/xml

<customer id="123">...</customer>
```


You are not required to send both the **If-Match** and **If-Unmodified-Since** headers. One or the other is sufficient to perform a conditional PUT or POST. When the server receives this request, it checks to see if the current **ETag** of the resource matches the value of the **If-Match** header and also to see if the timestamp on the resource matches the **If-Unmodified-Since** header. If these conditions are not met, the server will return an error response code of 412, “Precondition Failed.” This tells the client that the representation it is updating was modified concurrently and that it should retry. If the conditions are met, the service performs the update and sends a success response code back to the client.


### JAX-RS and Conditional Updates


To do conditional updates with JAX-RS, you use the **Request.evaluatePreconditions()** method again. Let’s look at how we can implement it within Java code:



```Java
@Path("/customers")
public class CustomerResource {

   @Path("{id}")
   @PUT
   @Consumes("application/xml")
   public Response updateCustomer(@PathParam("id") int id,
                                   @Context Request request,
                                    Customer update ) {
      Customer cust = findCustomer(id);
      EntityTag tag = new EntityTag(
                                   Integer.toString(cust.hashCode()));
      Date timestamp = ...; // get the timestamp

      ResponseBuilder builder =
                        request.evaluatePreconditions(timestamp, tag);

      if (builder != null) {
         // Preconditions not met!
         return builder.build();
      }

      ... perform the update ...

      builder = Response.noContent();
      return builder.build();
   }
```


The **updateCustomer()** method obtains a customer ID and an instance of **javax.ws.rs.core.Request** from the injected parameters. It then locates an instance of a **Customer** object in some application-specific way (for example, from a database). From this current instance of **Customer**, it creates an **EntityTag** from the hash code of the object. It also finds the current timestamp of the **Customer** instance in some application-specific way. The **Request.evaluatePreconditions()** method is then called with **timestamp** and **tag** variables. If these values do not match the values within the **If-Match** and **If-Unmodified-Since** headers sent with the request, **evaluatePreconditions()** sends back an instance of a **ResponseBuilder** initialized with the error code 412, “Precondition Failed.” A **Response** object is built and sent back to the client. If the preconditions are met, the service performs the update and sends back a success code of 204, “No Content.”



With this code in place, we can now worry less about concurrent updates of our resources. One interesting thought is that we did not have to come up with this scheme ourselves. It is already defined within the HTTP specification. This is one of the beauties of REST, in that it fully leverages the HTTP protocol.



