# Example ex09_1: Conneg with JAX-RS



This example is a slight modification from *ex06_1* and shows two different concepts. First, the same JAX-RS resource method can process two different media types. [Chapter 9](../../part1/chapter9/http_content_negotiation.md) gives the example of a method that returns a JAXB annotated class instance that can be returned as either JSON or XML. We’ve implemented this in *ex09_1* by slightly changing the **CustomerResource.getCustomer()** method:


```Java:src/main/java/com/restfully/shop/services/CustomerResource.java
@Path("/customers")
public class CustomerResource {
...
   @GET
   @Path("{id}")
   @Produces({"application/xml", "application/json"})
   public Customer getCustomer(@PathParam("id") int id)
   {
      ...
   }
```


The JAXB provider that comes with RESTEasy can convert JAXB objects to JSON or XML. In this example, we have added the media type **application/json** to **getCustomer()**’s **@Produces** annotation. The JAX-RS runtime will process the **Accept** header and pick the appropriate media type of the response for **getCustomer()**. If the **Accept** header is **application/xml**, XML will be produced. If the **Accept** header is JSON, the **Customer** object will be outputted as JSON.


The second concept being highlighted here is that you can use the **@Produces** annotation to dispatch to different Java methods. To illustrate this, we’ve added the **getCustomerString()** method, which processes the same URL as **getCustomer()** but for a different media type:


```Java
   @GET
   @Path("{id}")
   @Produces("text/plain")
   public Customer getCustomerString(@PathParam("id") int id)
   {
      return getCustomer(id).toString();
   }
```


### The Client Code


The client code for this example executes various HTTP GET requests to retrieve different representations of a **Customer**. Each request sets the **Accept** header a little differently so that it can obtain a different representation. For example:


```Java:src/test/java/com/restfully/shop/test/CustomerResourceTest.java
public class CustomerResourceTest
{
   @Test
   public void testCustomerResource() throws Exception
   {
      ... initialization code ...

      System.out.println("*** GET XML Created Customer **");
      String xml = client.target(location).request()
                                .accept(MediaType.APPLICATION_XML_TYPE)
                                .get(String.class);
      System.out.println(xml);

      System.out.println("*** GET JSON Created Customer **");
      String json = client.target(location).request()
              .accept(MediaType.APPLICATION_JSON_TYPE)
              .get(String.class);
      System.out.println(json);
   }
}
```


The **SyncInvoker.accept()** method is used to initialize the **Accept** header. The client extracts a **String** from the HTTP response so it can show you the request XML or JSON.



### Build and Run the Example Program


Perform the following steps:

1. Open a command prompt or shell terminal and change to the *ex09_1* directory of the workbook example code. 
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../part2/chapter17/workbook_introduction.md).
3. Perform the build and run the example by typing **maven install**. 