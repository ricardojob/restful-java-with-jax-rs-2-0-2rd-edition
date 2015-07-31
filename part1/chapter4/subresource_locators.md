# Subresource Locators


So far, I’ve shown you the JAX-RS capability to statically bind URI patterns expressed through the **@Path** annotation to a specific Java method. JAX-RS also allows you to dynamically dispatch requests yourself through subresource locators. Subresource locators are Java methods annotated with **@Path**, but with no HTTP method annotation, like **@GET**, applied to them. This type of method returns an object that is, itself, a JAX-RS annotated service that knows how to dispatch the remainder of the request. This is best described using an example.


Let’s continue by expanding our customer database JAX-RS service. This example will be a bit contrived, so please bear with me. Let’s say our customer database is partitioned into different databases based on geographic regions. We want to add this information to our URI scheme, but we want to decouple finding a database server from querying and formatting customer information. We will now add the database partition information to the URI pattern **/customers/{database}-db/{customerId}**. We can define a **CustomerDatabaseResource** class and have it delegate to our original **CustomerResource** class. Here’s the example:


```Java
@Path("/customers")
public class CustomerDatabaseResource {

   @Path("{database}-db")
   public CustomerResource getDatabase(@PathParam("database") String db) {
      // find the instance based on the db parameter
      CustomerResource resource = locateCustomerResource(db);
      return resource;
   }

   protected CustomerResource locateCustomerResource(String db) {
     ...
   }
}
```


The **CustomerDatabaseResource** class is our root resource. It does not service any HTTP requests directly. It processes the database identifier part of the URI and locates the identified customer database. Once it does this, it allocates a **CustomerResource** instance, passing in a reference to the database. The JAX-RS provider uses this **CustomerResource** instance to service the remainder of the request:


```Java
public class CustomerResource {
   private Map<Integer, Customer> customerDB =
                          new ConcurrentHashMap<Integer, Customer>();
   private AtomicInteger idCounter = new AtomicInteger();

   public CustomerResource(Map<Integer, Customer> customerDB)
   {
      this.customerDB = customerDB;
   }

   @POST
   @Consumes("application/xml")
   public Response createCustomer(InputStream is) {
     ...
   }

   @GET
   @Path("{id}")
   @Produces("application/xml")
   public StreamingOutput getCustomer(@PathParam("id") int id) {
     ...
   }

   @PUT
   @Path("{id}")
   @Consumes("application/xml")
   public void updateCustomer(@PathParam("id") int id, InputStream is) {
      ...
   }
}
```


So, if a client sends **GET /customers/northamerica-db/333**, the JAX-RS provider will first match the expression on the method **CustomerDatabaseResource.getDatabase()**. It will then match and process the remaining part of the request with the method **CustomerResource.getCustomer()**.



