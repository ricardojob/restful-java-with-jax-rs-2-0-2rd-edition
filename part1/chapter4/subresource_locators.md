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


Besides the added constructor, another difference in the **CustomerResource** class from previous examples is that it is no longer annotated with **@Path**. It is no longer a root resource in our system; it is a subresource and must not be registered with the JAX-RS runtime within an **Application** class.


### Full Dynamic Dispatching


While our previous example does illustrate the concept of subresource locators, it does not show their full dynamic nature. The **CustomerDatabaseResource.getDatabase()** method can return any instance of any class. At runtime, the JAX-RS provider will introspect this instance’s class for resource methods that can handle the request.


Let’s say that in our example, we have two customer databases with different kinds of identifiers. One database uses a numeric key, as we talked about before. The other uses first and last name as a composite key. We would need to have two different classes to extract the appropriate information from the URI. Let’s change our example:


```Java
@Path("/customers")
public class CustomerDatabaseResource {

   protected CustomerResource europe = new CustomerResource();
   protected FirstLastCustomerResource northamerica =
                              new FirstLastCustomerResource();

   @Path("{database}-db")
   public Object getDatabase(@PathParam("database") String db) {
      if (db.equals("europe")) {
          return europe;
      }
      else if (db.equals("northamerica")) {
          return northamerica;
      }
      else return null;
   }
}
```


Instead of our **getDatabase()** method returning a **CustomerResource**, it will return any **java.lang.Object**. JAX-RS will introspect the instance returned to figure out how to dispatch the request. For this example, if our database is **europe**, we will use our original **CustomerResource** class to service the remainder of the request. If our database is **northamerica**, we will use a new subresource class **FirstLastCustomerResource**:


```Java
public class FirstLastCustomerResource {
   private Map<String, Customer> customerDB =
                          new ConcurrentHashMap<String, Customer>();

      @GET
   @Path("{first}-{last}")
   @Produces("application/xml")
   public StreamingOutput getCustomer(@PathParam("first") String firstName,
                                      @PathParam("last") String lastName) {
     ...
   }

   @PUT
   @Path("{first}-{last}")
   @Consumes("application/xml")
   public void updateCustomer(@PathParam("first") String firstName,
                              @PathParam("last") String lastName,
                              InputStream is) {
      ...
   }
}
```


Customer lookup requests routed to **europe** would match the **/customers/{database}-db/{id}** URI pattern defined in **CustomerResource**. Requests routed to **northamerica** would match the **/customers/{database}-db/{first}-{last}** URI pattern defined in **FirstLastCustomerResource**. This type of pattern gives you a lot of freedom to dispatch your own requests.









