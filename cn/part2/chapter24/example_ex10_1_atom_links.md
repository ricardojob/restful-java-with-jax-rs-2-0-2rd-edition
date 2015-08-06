# Example ex10_1: Atom Links


This example is a slight modification of the *ex06_1* example introduced in [Chapter 21](../chapter21/examples_for_chapter_6.md). It expands the **CustomerResource** RESTful web service so that a client can fetch subsets of the customer database. If a client does a **GET /customers** request in our RESTful application, it will receive a subset list of customers in XML. Two Atom links are embedded in this document that allow you to view the next or previous sets of customer data. Example output would be:


```xml
<customers>
   <customer id="3">
     ...
   </customer>
   <customer id="4">
      ...
   </customer>
   <link rel="next"
         href="http://example.com/customers?start=5&size=2"
         type="application/xml"/>
   <link rel="previous"
         href="http://example.com/customers?start=1&size=2"
         type="application/xml"/>
</customers>
```


The **next** and **previous** links are URLs pointing to the same **/customers** URL, but they contain URI query parameters indexing into the customer database.



### The Server Code


The first bit of code is a JAXB class that maps to the **&lt;customers&gt;** element. It must be capable of holding an arbitrary number of **Customer** instances as well as the Atom links for our **next** and **previous** link relationships. We can use the **javax.ws.rs.core.Link** class with its JAXB adapter to represent these links:


```Java:src/main/java/com/restfully/shop/domain/Customers.java
import javax.ws.rs.core.Link;
...

@XmlRootElement(name = "customers")
public class Customers
{
   protected Collection<Customer> customers;
   protected List<Link> links;

   @XmlElementRef
   public Collection<Customer> getCustomers()
   {
      return customers;
   }

   public void setCustomers(Collection<Customer> customers)
   {
      this.customers = customers;
   }

   @XmlElement(name="link")
   @XmlJavaTypeAdapter(Link.JaxbAdapter.class)
   public List<Link> getLinks()
   {
      return links;
   }

   public void setLinks(List<Link> links)
   {
      this.links = links;
   }

   @XmlTransient
   public URI getNext()
   {
      if (links == null) return null;
      for (Link link : links)
      {
         if ("next".equals(link.getRel())) return link.getUri();
      }
      return null;
   }

   @XmlTransient
   public URI getPrevious()
   {
      if (links == null) return null;
      for (Link link : links)
      {
         if ("previous".equals(link.getRel())) return link.getUri();
      }
      return null;
   }

}
```


There is no nice way to define a map in JAXB, so all the Atom links are stuffed within a collection property in **Customers**. The convenience methods **getPrevious()** and **getNext()** iterate through this collection to find the **next** and **previous** Atom links embedded within the document if they exist.


The final difference from the *ex06_1* example is the implementation of **GET /customers** handling:


```Java:src/main/java/com/restfully/shop/services/CustomerResource.java
@Path("/customers")
public class CustomerResource
{
   @GET
   @Produces("application/xml")
   @Formatted
   public Customers getCustomers(@QueryParam("start") int start,
                   @QueryParam("size") @DefaultValue("2") int size,
                   @Context UriInfo uriInfo)
   {
```


The **@org.jboss.resteasy.annotations.providers.jaxb.Formatted** annotation is a RESTEasy-specific plug-in that formats the XML output returned to the client to include indentations and new lines so that the text is easier to read.


The query parameters for the **getCustomers()** method, **start** and **size**, are optional. They represent an index into the customer database and how many customers you want returned by the invocation. The **@DefaultValue** annotation is used to define a default page size of 2.


The **UriInfo** instance injected with **@Context** is used to build the URLs that define **next** and **previous** link relationships.


```Java
      UriBuilder builder = uriInfo.getAbsolutePathBuilder();
      builder.queryParam("start", "{start}");
      builder.queryParam("size", "{size}");
```


Here, the code defines a URI template by using the **UriBuilder** passed back from **UriInfo.getAbsolutePathBuilder()**. The **start** and **size** query parameters are added to the template. Their values are populated using template parameters later on when the actual links are built.


```Java
      ArrayList<Customer> list = new ArrayList<Customer>();
      ArrayList<Link> links = new ArrayList<Link>();
      synchronized (customerDB)
      {
         int i = 0;
         for (Customer customer : customerDB.values())
         {
            if (i >= start && i < start + size)
                                      list.add(customer);
            i++;
         }
```


The code then gathers up the **Customer** instances that will be returned to the client based on the **start** and **size** parameters. All this code is done within a **synchronized** block to protect against concurrent access on the **customerDB** map.



```Java
         // next link
         if (start + size < customerDB.size())
         {
            int next = start + size;
            URI nextUri = builder.clone().build(next, size);
            Link nextLink = Link.fromUri(nextUri)
                                .rel("next").type("application/xml").build();
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
```


If there are more possible customer instances left to be viewed, a **next** link relationship is calculated using the **UriBuilder** template defined earlier. A similar calculation is done to see if a **previous** link relationship needs to be added to the document.


```Java
      }
      Customers customers = new Customers();
      customers.setCustomers(list);
      customers.setLinks(links);
      return customers;
   }
```


Finally, a **Customers** instance is created and initialized with the **Customer** list and link relationships and returned to the client.



### The Client Code


The client initially gets the XML document from the **/customers** URL. It then loops using the **next** link relationship as the URL to print out all the customers in the database:


```Java
public class CustomerResourceTest
{
   @Test
   public void testQueryCustomers() throws Exception
   {
      URI uri = new URI("http://localhost:8080/services/customers");
      while (uri != null)
      {
         WebTarget target = client.target(uri);
         String output = target.request().get(String.class);
         System.out.println("** XML from " + uri.toString());
         System.out.println(output);

         Customers customers = target.request().get(Customers.class);
         uri = customers.getNext();
      }
   }
}
```


An interesting thing to note about this is that the server is guiding the client to make state transitions as it browses the customer database. Once the initial URL is invoked, further queries are solely driven by Atom links.



### Build and Run the Example Program


Perform the following steps:

1. Open a command prompt or shell terminal and change to the *ex10_1* directory of the workbook example code. 
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md). 
3. Perform the build and run the example by typing **maven install**.