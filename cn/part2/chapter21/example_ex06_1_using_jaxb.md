# Example ex06_1: Using JAXB


This example shows how easy it is to use JAXB and JAX-RS to exchange XML documents over HTTP. The **com.restfully.shop.domain.Customer** class is the first interesting piece of the example.


```Java:src/main/java/com/restfully/shop/domain/Customer.java
@XmlRootElement(name="customer")
public class Customer {
   private int id;
   private String firstName;
   private String lastName;
   private String street;
   private String city;
   private String state;
   private String zip;
   private String country;

   @XmlAttribute
   public int getId() {
      return id;
   }

   public void setId(int id) {
      this.id = id;
   }

   @XmlElement(name="first-name")
   public String getFirstName() {
      return firstName;
   }

   public void setFirstName(String firstName) {
      this.firstName = firstName;
   }

   @XmlElement(name="last-name")
   public String getLastName() {
      return lastName;
   }

   public void setLastName(String lastName) {
      this.lastName = lastName;
   }

   @XmlElement
   public String getStreet() {
      return street;
   }
...
}
```


The JAXB annotations provide a mapping between the **Customer** class and XML.


You don’t need to write a lot of code to implement the JAX-RS service because JAX-RS already knows how to handle JAXB annotated classes:


```Java:src/main/java/com/restfully/shop/services/CustomerResource.java
@Path("/customers")
public class CustomerResource {
   private Map<Integer, Customer> customerDB =
                      new ConcurrentHashMap<Integer, Customer>();
   private AtomicInteger idCounter = new AtomicInteger();

   public CustomerResource() {
   }

   @POST
   @Consumes("application/xml")
   public Response createCustomer(Customer customer) {
      customer.setId(idCounter.incrementAndGet());
      customerDB.put(customer.getId(), customer);
      System.out.println("Created customer " + customer.getId());
      return Response.created(URI.create("/customers/" +
                                  customer.getId())).build();

   }

   @GET
   @Path("{id}")
   @Produces("application/xml")
   public Customer getCustomer(@PathParam("id") int id) {
      Customer customer = customerDB.get(id);
      if (customer == null) {
        throw new WebApplicationException(Response.Status.NOT_FOUND);
      }
      return customer;
   }

   @PUT
   @Path("{id}")
   @Consumes("application/xml")
   public void updateCustomer(@PathParam("id") int id,
                                             Customer update) {
      Customer current = customerDB.get(id);
      if (current == null)
        throw new WebApplicationException(Response.Status.NOT_FOUND);

      current.setFirstName(update.getFirstName());
      current.setLastName(update.getLastName());
      current.setStreet(update.getStreet());
      current.setState(update.getState());
      current.setZip(update.getZip());
      current.setCountry(update.getCountry());
   }
}
```


If you compare this with the **CustomerResource** class in *ex03_1*, you’ll see that the code in this example is much more compact. There is no handcoded marshalling code, and our methods are dealing with **Customer** objects directly instead of raw strings.



### The Client Code



The client code can also now take advantage of automatic JAXB marshalling. All JAX-RS 2.0 client implementations must support JAXB as a mechanism to transmit XML on the client side. Let’s take a look at how the client code has changed from *ex03_1*:


```Java
   @Test
   public void testCustomerResource() throws Exception
   {
      System.out.println("*** Create a new Customer ***");
      Customer newCustomer = new Customer();
      newCustomer.setFirstName("Bill");
      newCustomer.setLastName("Burke");
      newCustomer.setStreet("256 Clarendon Street");
      newCustomer.setCity("Boston");
      newCustomer.setState("MA");
      newCustomer.setZip("02115");
      newCustomer.setCountry("USA");
```


We start off by allocating and initializing a **Customer** object with the values of the new customer we want to create.


```Java
      Response response =
               client.target("http://localhost:8080/services/customers")
                     .request().post(Entity.xml(newCustomer));
      if (response.getStatus() != 201)
           throw new RuntimeException("Failed to create");
```


The code for posting the new customer looks exactly the same as *ex03_1* except that instead of initializing a **javax.ws.rs.client.Entity** with an XML string, we are using the **Customer** object. The JAX-RS client will automatically marshal this object into an XML string and send it across the wire.



### Changes to pom.xml


JBoss RESTEasy is broken up into a bunch of smaller JARs so that you can pick and choose which features of RESTEasy to use. Because of this, the core RESTEasy JAR file does not have the JAXB content handlers. Therefore, we need to add a new dependency to our *pom.xml* file:


```xml
<dependency>
   <groupId>org.jboss.resteasy</groupId>
   <artifactId>resteasy-jaxb-provider</artifactId>
   <version>1.2</version>
</dependency>
```


Adding this dependency will add the JAXB provider to your project. It will also pull in any third-party dependency RESTEasy needs to process XML. If you are deploying on a Java EE application server like Wildfly or JBoss, you will not need this dependency; JAXB support is preinstalled.



### Build and Run the Example Program


Perform the following steps:

1. Open a command prompt or shell terminal and change to the *ex06_1* directory of the workbook example code. 
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md). 
3. Perform the build and run the example by typing **maven install**.