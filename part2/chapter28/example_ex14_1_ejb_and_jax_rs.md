# Example ex14_1: EJB and JAX-RS


This example shows how you can use JAX-RS with EJB and JPA. It makes use of some of the integration code discussed in [Chapter 14](../../part1/chapter14/deployment_and_integration.md).



### Project Structure


To implement *ex14_1*, the Wildfly 8.0 Application Server is used to deploy the example. Wildfly is the community version of the JBoss application server. It is Java EE 7–compliant, so JAX-RS 2.0 is already built in. As a result, our Maven *pom.xml* file needs to change a little to support this example. First, let’s look at the dependency changes in this build file:


```xml:pom.xml
    <dependencies>
        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-jaxrs</artifactId>
            <version>3.0.5.Final</version>
            <scope>provided</scope>
        </dependency>
```


Because JAX-RS 2.0 is built in, we do not have to add all the RESTEasy third-party dependencies to our WAR file. The **provided** scope is used to tell Maven that the JAR dependencies are needed only for compilation and to not include them within the WAR.



Next, we need to include a Wildfly Maven plug-in:



```xml
        <plugins>
            <plugin>
                <groupId>org.jboss.as.plugins</groupId>
                <artifactId>jboss-as-maven-plugin</artifactId>
                <version>7.1.1.Final</version>
                <executions>
                    <execution>
                        <id>jboss-deploy</id>
                        <phase>pre-integration-test</phase>
                        <goals>
                            <goal>deploy</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>jboss-undeploy</id>
                        <phase>post-integration-test</phase>
                        <goals>
                            <goal>undeploy</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```



The **jboss-as-maven-plugin** is configured to run after the WAR is built, but before the unit tests are run. It uses the Wildfly remote deployment interface to automatically deploy our WAR to the Wildfly application server. We’ll see later how to start the example.



### The EJBs


The EJB code is very similar to *ex10_2* from [Chapter 24](../chapter24/examples_for_chapter_10.md), except the code has been expanded to save created order entries into a relational database instead of an in-memory map. Like all of our previous examples, the JAXB classes that define our XML data format live in the **com.restfully.shop.domain** package. A separate parallel Java package, **com.restfully.shop.persistence**, was created for the example’s JPA classes. These JPA classes are almost a carbon copy of the JAXB ones, except they are using JPA annotations to map to a relational database.



You could use JAXB and JPA annotations together within one class hierarchy, but this isn’t the best idea, as there are a few problems you might encounter. The first has to do with how JPA works. Objects like the **OrderEntity** have relationships to other classes like **LineItemEntity**, **ProductEntity**, and **CustomerEntity**. In JPA, it is common to lazy-load these objects as their object graphs are traversed. This can save on database access time. The problem where JAX-RS is concerned is that the JAX-RS runtime will usually turn the Java object into an XML document outside the scope of an EJB request. This might cause lazy-load exceptions when JAXB tries to traverse the entire object graph.



You can write your code so that it is careful not to introduce lazy-load exceptions, but there is one other major problem you may encounter. You will often want to support older clients that use older versions of the XML format. This can cause a divergence between your XML schema and your database schema. The best way to avoid this problem is to create two separate class hierarchies. That way, your XML and database mappings can evolve separately from one another. Yes, it’s a little more code for you to write, but it will save you headaches in the long run.



I’m going to skip a lot of the details of this example. You’ve already seen how JAXB classes work and this book isn’t an exercise on learning JPA, so I’ll focus on how JAX-RS interacts with EJB. Let’s take a look at one of the EJBs:



```Java:ejb/src/main/java/com/restfully/shop/services/CustomerResource.java
@Path("/customers")
public interface CustomerResource
{
   @POST
   @Consumes("application/xml")
   Response createCustomer(Customer customer,
                                          @Context UriInfo uriInfo);

   @GET
   @Produces("application/xml")
   @Formatted
   Customers getCustomers(@QueryParam("start") int start,
                   @QueryParam("size") @DefaultValue("2") int size,
                   @QueryParam("firstName") String firstName,
                   @QueryParam("lastName") String lastName,
                   @Context UriInfo uriInfo);

   @GET
   @Path("{id}")
   @Produces("application/xml)
   Customer getCustomer(@PathParam("id") int id);
}
```


For a non-JAX-RS-aware EJB container to work with JAX-RS, you need to define your JAX-RS annotations on the EJB’s business interface. The **CustomerResource** interface does just this.



Our EJB business logic is defined within the **CustomerResourceBean** class:



```Java:ejb/src/main/java/com/restfully/shop/services/CustomerResourceBean.java
@Stateless
public class CustomerResourceBean implements CustomerResource
{
   @PersistenceContext
   private EntityManager em;
```


Our EJB class is annotated with the **@javax.ejb.Stateless** annotation to mark it as a stateless session EJB. The **CustomerResourceBean** class implements the **CustomerResource** interface.


There is a **javax.persistence.EntityManager** field named **em**. The annotation **@javax.persistence.PersistenceContext** injects an instance of the **EntityManager** into that field. The **EntityManager** persists Java objects into a relational database. These are all facilities of EJB and JPA.


```Java
   public Response createCustomer(Customer customer, UriInfo uriInfo)
   {
      CustomerEntity entity = new CustomerEntity();
      domain2entity(entity, customer);
      em.persist(entity);
      em.flush();

      System.out.println("Created customer " + entity.getId());
      UriBuilder builder = uriInfo.getAbsolutePathBuilder();
      builder.path(Integer.toString(entity.getId()));
      return Response.created(builder.build()).build();

   }
```


The **createCustomer()** method implements the RESTful creation of a **Customer** in the database. The **Customer** object is the unmarshalled representation of the XML document posted through HTTP. The code allocates an instance of **com.restfully.shop.persistence.CustomerEntity** and copies the data from **Customer** to this instance. The **EntityManager** then persists the **CustomerEntity** instance into the database. Finally, the method uses **UriInfo.getAbsolutePathBuilder()** to create a URL that will populate the value of the **Location** header that is sent back with the HTTP response.


```Java
   public Customer getCustomer(int id)
   {
      CustomerEntity customer = em.getReference(CustomerEntity.class,
                                                  id);
      return entity2domain(customer);
   }
```


The **getCustomer()** method services **GET /customers/&lt;id &gt;** requests and retrieves **CustomerEntity** objects from the database using the **EntityManager**. The **entity2domain()** method call converts the **CustomerEntity** instance found in the database into an instance of the JAXB class **Customer**. This **Customer** instance is what is returned to the JAX-RS runtime.


```Java
   public static void domain2entity(CustomerEntity entity,
                                     Customer customer)
   {
      entity.setId(customer.getId());
      entity.setFirstName(customer.getFirstName());
      entity.setLastName(customer.getLastName());
      entity.setStreet(customer.getStreet());
      entity.setCity(customer.getCity());
      entity.setState(customer.getState());
      entity.setZip(customer.getZip());
      entity.setCountry(customer.getCountry());
   }

   public static Customer entity2domain(CustomerEntity entity)
   {
      Customer cust = new Customer();
      cust.setId(entity.getId());
      cust.setFirstName(entity.getFirstName());
      cust.setLastName(entity.getLastName());
      cust.setStreet(entity.getStreet());
      cust.setCity(entity.getCity());
      cust.setState(entity.getState());
      cust.setZip(entity.getZip());
      cust.setCountry(entity.getCountry());
      return cust;
   }
```


The **domain2entity()** and **entity2domain()** methods simply convert to and from the JAXB and JPA class hierarchies.


```Java
   public Customers getCustomers(int start,
                                 int size,
                                 String firstName,
                                 String lastName,
                                 UriInfo uriInfo)
   {
      UriBuilder builder = uriInfo.getAbsolutePathBuilder();
      builder.queryParam("start", "{start}");
      builder.queryParam("size", "{size}");

      ArrayList<Customer> list = new ArrayList<Customer>();
      ArrayList<Link> links = new ArrayList<Link>();
```


The **getCustomers()** method is expanded as compared to previous examples in this book. The **firstName** and **lastName** query parameters are added. This allows clients to search for customers in the database with a specific first and last name.


```Java
      Query query = null;
      if (firstName != null && lastName != null)
      {
         query = em.createQuery(
                "select c from Customer c where c.firstName=:first
                     and c.lastName=:last");
         query.setParameter("first", firstName);
         query.setParameter("last", lastName);

      }
      else if (lastName != null)
      {
         query = em.createQuery(
                  "select c from Customer c where c.lastName=:last");
         query.setParameter("last", lastName);
      }
      else
      {
         query = em.createQuery("select c from Customer c");
      }
```


The **getCustomers()** method builds a JPA query based on the values of **firstName** and **lastName**. If these are both set, it searches in the database for all customers with that first and last name. If only **lastName** is set, it searches only for customers with that last name. Otherwise, it just queries for all customers in the database.


```Java
      List customerEntities = query.setFirstResult(start)
              .setMaxResults(size)
              .getResultList();
```


Next, the code executes the query. You can see that doing paging is a little bit easier with JPA than the in-memory database we used in [Chapter 24](../chapter24/examples_for_chapter_10.md). The **setMaxResults()** and **query.setFirstResult()** methods set the index and size of the dataset you want returned.


```Java
      for (Object obj : customerEntities)
      {
         CustomerEntity entity = (CustomerEntity) obj;
         list.add(entity2domain(entity));
      }
```


Next, the code iterates through all the **CustomerEntity** objects returned by the executed query and creates **Customer** JAXB object instances.


```Java
      // next link
      // If the size returned is equal then assume there is a next
      if (customerEntities.size() == size)
      {
         int next = start + size;
         URI nextUri = builder.clone().build(next, size);
         Link nextLink = Link.fromUri(nextUri)
                             .rel("next")
                             .type("application/xml").build();
         links.add(nextLink);
      }
      // previous link
      if (start > 0)
      {
         int previous = start - size;
         if (previous < 0) previous = 0;
         URI previousUri = builder.clone().build(previous, size);
         Link previousLink = Link.fromUri(previousUri)
                                 .rel("previous")
                                 .type("application/xml").build();
         links.add(previousLink);
      }
      Customers customers = new Customers();
      customers.setCustomers(list);
      customers.setLinks(links);
      return customers;
   }

}
```


Finally, the method calculates whether the **next** and **previous** Atom links should be added to the **Customers** JAXB object returned. This code is very similar to the examples described in [Chapter 24](../chapter24/examples_for_chapter_10.md).


The other EJB classes defined in the example are pretty much extrapolated from the *ex10_2* example and modified to work with JPA. I don’t want to rehash old code, so I won’t get into detail on how these work.




### The Remaining Server Code


There’s a few more server-side classes we need to go over.


#### The ExceptionMappers


The **EntityManager.getReference()** method is used by various EJBs in this example to locate objects within the database. When this method cannot find an object within the database, it throws a **javax.persistence.EntityNotFoundException**. If we deployed this code as is, JAX-RS would end up eating this exception and returning a 500, “Internal Server Error,” to our clients if they tried to access an unknown object in the database. The 404, “Not Found,” error response code makes a lot more sense to return in this scenario. To facilitate this, a JAX-RS **ExceptionMapper** is used. Let’s take a look:


```Java:ejb/src/main/java/com/restfully/shop/services/EntityNotFoundExceptionMapper.java
@Provider
public class EntityNotFoundExceptionMapper
               implements ExceptionMapper<EntityNotFoundException>
{
   public Response toResponse(EntityNotFoundException exception)
   {
      return Response.status(Response.Status.NOT_FOUND).build();
   }
}
```


This class catches **EntityNotFoundExceptions** and generates a 404 response.



#### Changes to Application class


The **ShoppingApplication** class has been simplified a bit. Because all of our code is implemented as EJBs, there’s no special registration we need to do in our **Application** class. Here’s what it looks like now:


```Java:war/src/main/java/com/restfully/shop/services/ShoppingApplication.java
@ApplicationPath("/services")
public class ShoppingApplication extends Application
{
}
```


The Wildfly application server will scan the WAR for any annotated JAX-RS classes and automatically deploy them. In this deployment, all of our JAX-RS services are EJBs and contained in the *WEB-INF/classes* folder of our WAR.



### The Client Code


Let’s take a look at the client code:


```Java:ear/src/test/java/com/restfully/shop/test/ShoppingTest.java
   protected void populateDB() throws Exception
   {
      Response response =
                client.target("http://localhost:8080/ex14_1/services/shop")
                      .request().head();
      Link products = response.getLink("products");
      response.close();

      System.out.println("** Populate Products");

      Product product = new Product();
      product.setName("iPhone");
      product.setCost(199.99);
      response = client.target(products).request().post(Entity.xml(product));
      Assert.assertEquals(201, response.getStatus());
      response.close();

      product = new Product();
      product.setName("MacBook Pro");
      product.setCost(3299.99);
      response = client.target(products).request().post(Entity.xml(product));
      Assert.assertEquals(201, response.getStatus());
      response.close();

      product = new Product();
      product.setName("iPod");
      product.setCost(49.99);
      response = client.target(products).request().post(Entity.xml(product));
      Assert.assertEquals(201, response.getStatus());
      response.close();
   }
```


The **populateDB()** method makes HTTP calls on the **ProductResource** JAX-RS service to create a few products in the database.



```Java
   @Test
   public void testCreateOrder() throws Exception
   {
      populateDB();

      Response response = client.target
        ("http://localhost:8080/ex14_1/services/shop").request().head();
      Link customers = response.getLink("customers");
      Link products = response.getLink("products");
      Link orders = response.getLink("orders");
      response.close();
```

The test starts off by initializing the server’s database by calling **populateDB()**. Like *ex10_2*, the client interacts with the **StoreResource** JAX-RS service to obtain links to all the services in the system.


```Java
      System.out.println("** Buy an iPhone for Bill Burke");
      System.out.println();
      System.out.println("** First see if Bill Burke exists as a customer");
      Customers custs = client.target(customers)
                                  .queryParam("firstName", "Bill")
                                  .queryParam("lastName", "Burke")
                                  .request().get(Customers.class);
      Customer customer = null;
      if (custs.getCustomers().size() > 0)
      {
         System.out.println("- Found a Bill Burke in the database, using that");
         customer = custs.getCustomers().iterator().next();
      }
      else
      {
         System.out.println("- Cound not find a Bill Burke in the database,
                            creating one.");
         customer = new Customer();
         customer.setFirstName("Bill");
         customer.setLastName("Burke");
         customer.setStreet("222 Dartmouth Street");
         customer.setCity("Boston");
         customer.setState("MA");
         customer.setZip("02115");
         customer.setCountry("USA");
         response = client.target(customers)
                          .request()
                          .post(Entity.xml(customer));
         Assert.assertEquals(201, response.getStatus());
         URI uri = response.getLocation();
         response.close();

         customer = client.target(uri).request().get(Customer.class);
      }
```


The client code checks to see if the customer “Bill Burke” already exists. If that customer doesn’t exist, it is created within the customer database.


```Java
      System.out.println();
      System.out.println("Search for iPhone in the Product database");
      Products prods = client.target(products)
                             .queryParam("name", "iPhone")
                             .request()
                             .get(Products.class);
      Product product = null;
      if (prods.getProducts().size() > 0)
      {
         System.out.println("- Found iPhone in the database.");
         product = prods.getProducts().iterator().next();
      }
      else
      {
         throw new RuntimeException("Failed to find an iPhone in the database!");
      }
```


The customer wants to buy a product called **iPhone**, so the client searches the product database for it.


```Java
      System.out.println();
      System.out.println("** Create Order for iPhone");
      LineItem item = new LineItem();
      item.setProduct(product);
      item.setQuantity(1);
      Order order = new Order();
      order.setTotal(product.getCost());
      order.setCustomer(customer);
      order.setDate(new Date().toString());
      order.getLineItems().add(item);
      response = client.target(orders).request().post(Entity.xml(order));
      Assert.assertEquals(201, response.getStatus());
      response.close();

      System.out.println();
      System.out.println("** Show all orders.");
      String xml = client.target(orders).request().get(String.class);
      System.out.println(xml);
   }
}
```


Finally, an order is created within the database.



### Build and Run the Example Program


Perform the following steps:

1. Download the latest Wildfly 8.0 Application Server from http://www.wildfly.org/download/.
2. Unzip Wildfly 8.0 into any directory you want.
3. Open a command prompt or shell terminal, and then change to the *wildfly-8.0.0.Final/bin* directory.
4. You must start Wildfly manually before you can run the example. To do this, execute *standalone.sh* or *standalone.bat*, depending on whether you are using a Unix- or Windows-based system.
5. Open another command prompt or shell terminal and change to the *ex14_1* directory of the workbook example code.
6. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md).
7. Perform the build and run the example by typing **maven install**.


As described before, the *pom.xml* file within the project is configured to use a special JBoss plug-in so that it can deploy the WAR file from the example to the application server. After the WAR is deployed, the client test code will be executed. Following the execution of the test, the WAR will be undeployed from JBoss by Maven.
