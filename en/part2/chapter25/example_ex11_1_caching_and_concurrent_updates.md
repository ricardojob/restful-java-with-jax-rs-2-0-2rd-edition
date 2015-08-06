# Example ex11_1: Caching and Concurrent Updates


The example in this chapter expands on the **CustomerResource** example repeated throughout this book to support caching, conditional GETs, and conditional PUTs.



### The Server Code


The first thing is to add a **hashCode()** method to the **Customer** class:


```Java:src/main/java/com/restfully/shop/domain/Customer.java
@XmlRootElement(name = "customer")
public class Customer
{
...
   @Override
   public int hashCode()
   {
      int result = id;
      result = 31 * result + (firstName != null
                                  ? firstName.hashCode() : 0);
      result = 31 * result + (lastName != null
                                  ? lastName.hashCode() : 0);
      result = 31 * result + (street != null
                                  ? street.hashCode() : 0);
      result = 31 * result + (city != null ? city.hashCode() : 0);
      result = 31 * result + (state != null ? state.hashCode() : 0);
      result = 31 * result + (zip != null ? zip.hashCode() : 0);
      result = 31 * result + (country != null
                                    ? country.hashCode() : 0);
      return result;
   }
}
```


This method is used in the **CustomerResource** class to generate semi-unique **ETag** header values. While a hash code calculated in this manner isn’t guaranteed to be unique, there is a high probability that it will be. A database application might use an incremented version column to calculate the **ETag** value.


The **CustomerResource** class is expanded to support conditional GETs and PUTs. Let’s take a look at the relevant pieces of code:


```Java:src/main/java/com/restfully/shop/services/CustomerResource.java
@Path("/customers")
public class CustomerResource
{
...

   @GET
   @Path("{id}")
   @Produces("application/xml")
   public Response getCustomer(@PathParam("id") int id,
                                @Context Request request) {
      Customer cust = customerDB.get(id);
      if (cust == null)
      {
         throw new WebApplicationException(Response.Status.NOT_FOUND);
      }

      if (sent == null) System.out.println("No ETag sent by client");

      EntityTag tag = new EntityTag(Integer.toString(cust.hashCode()));

      CacheControl cc = new CacheControl();
      cc.setMaxAge(5);
```


The **getCustomer()** method first starts out by retrieving the current **Customer** object identified by the **id** parameter. A current **ETag** value is created from the hash code of the **Customer** object. A new **Cache-Control** header is instantiated as well.


```Java
      Response.ResponseBuilder builder =
                   request.evaluatePreconditions(tag);
      if (builder != null) {
         System.out.println(
                   "** revalidation on the server was successful");
         builder.cacheControl(cc);
         return builder.build();
      }
```


Next, **Request.evaluatePreconditions()** is called to perform a conditional GET. If the client has sent an **If-None-Match** header that matches the calculated current **ETag**, the method returns immediately with an empty response body. In this case, a new **Cache-Control** header is sent back to refresh the **max-age** the client will use.


```Java
      // Preconditions not met!

      cust.setLastViewed(new Date().toString());
      builder = Response.ok(cust, "application/xml");
      builder.cacheControl(cc);
      builder.tag(tag);
      return builder.build();
   }
}
```


If no **If-None-Match** header was sent or the preconditions were not met, the **Customer** is sent back to the client with an updated **Cache-Control** header.



```Java
   @Path("{id}")
   @PUT
   @Consumes("application/xml")
   public Response updateCustomer(@PathParam("id") int id,
                                   @Context Request request,
                                    Customer update ) {
      Customer cust = customerDB.get(id);
      if (cust == null)
          throw new WebApplicationException(Response.Status.NOT_FOUND);
      EntityTag tag = new EntityTag(Integer.toString(cust.hashCode()));
```


The **updateCustomer()** method is responsible for updating a customer. It first starts off by finding the current **Customer** with the given **id**. From this queried customer, it generates the up-to-date value of the **ETag** header.


```Java
      Response.ResponseBuilder builder =
                        request.evaluatePreconditions(tag);

      if (builder != null) {
         // Preconditions not met!
         return builder.build();
      }
```


The current **ETag** header is compared against any **If-Match** header sent by the client. If it does match, the update can be performed:


```Java
      // Preconditions met, perform update

      cust.setFirstName(update.getFirstName());
      cust.setLastName(update.getLastName());
      cust.setStreet(update.getStreet());
      cust.setState(update.getState());
      cust.setZip(update.getZip());
      cust.setCountry(update.getCountry());


      builder = Response.noContent();
      return builder.build();
   }
}
```


Finally, the update is performed.



### The Client Code


The client code first performs a conditional GET. It then tries to do a conditional PUT using a bad **ETag** value.


```Java
public class CustomerResourceTest
{
   @Test
   public void testCustomerResource() throws Exception
   {
       WebTarget customerTarget =
             client.target("http://localhost:8080/services/customers/1");
       Response response = customerTarget.request().get();
       Assert.assertEquals(200, response.getStatus());
       Customer cust = response.readEntity(Customer.class);

       EntityTag etag = response.getEntityTag();
       response.close();
```


The **testCustomerResource()** method starts off by fetching a preinitialized **Customer** object. It does this so that it can obtain the current **ETag** of the **Customer** representation.


```Java
      System.out.println("Doing a conditional GET with ETag: "
                                                + etag.toString());
      response = customerTarget.request()
                               .header("If-None-Match", etag).get();
      Assert.assertEquals(304, response.getStatus());
      response.close();
```


This code is performing a conditional GET. We set the **If-None-Match** header using the previously fetched **ETag** value. The client is expecting that the server will return a 304, “Not Modified,” response.


```Java
      // Update and send a bad etag with conditional PUT
      cust.setCity("Bedford");
      response = customerTarget.request()
              .header("If-Match", "JUNK")
              .put(Entity.xml(cust));
      Assert.assertEquals(412, response.getStatus());
      response.close();
   }
}
```


Finally, the code does a conditional PUT with a bad **ETag** value sent with the **If-Match** header. The client is expecting this operation to fail with a 412, “Precondition Failed,” response.



### Build and Run the Example Program


Perform the following steps:

1. Open a command prompt or shell terminal and change to the *ex11_1* directory of the workbook example code. 
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md). 
3. Perform the build and run the example by typing **maven install**. 



Another interesting thing you might want to try is to start up and leave the application running by using **maven jetty:run**. Open your browser to http://localhost:8080/customers/1. Continually refresh this URL. You will be able to see if your browser performs a conditional GET request or not by viewing the **&lt;last-viewed&gt;** element of the returned XML. I found that Firefox 3.5.2 does a conditional GET, while Safari 4.0.1 does not.