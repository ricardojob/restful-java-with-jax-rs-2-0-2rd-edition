# RESTEasy Client Proxies


The RESTEasy Client Proxy Framework is a different way of writing RESTful Java clients. The idea of the framework is to reuse the JAX-RS annotations on the client side. When you write JAX-RS services, you are using the specification’s annotations to turn an HTTP invocation into a Java method call. The RESTEasy Client Proxy Framework flips this around to instead use the annotations to turn a method call into an HTTP request.


You start off by writing a Java interface with methods annotated with JAX-RS annotations. For example, let’s define a RESTful client interface to the customer service application we have talked about over and over again throughout this book:


```Java
@Path("/customers")
public interface CustomerResource {

   @GET
   @Produces("application/xml")
   @Path("{id}")
   public Customer getCustomer(@PathParam("id") int id);

   @POST
   @Consumes("application/xml")
   public Response createCustomer(Customer customer);

   @PUT
   @Consumes("application/xml")
   @Path("{id}")
   public void updateCustomer(@PathParam("id") int id, Customer cust);
}
```


This interface looks exactly like the interface a JAX-RS service might implement. Through RESTEasy, we can turn this interface into a Java object that can invoke HTTP requests. To do this, we use the **org.jboss.resteasy.client.jaxrs.ResteasyWebTarget** interface:


```Java
Client client = ClientFactory.newClient();
WebTarget target = client.target("http://example.com/base/uri");
ResteasyWebTarget target = (ResteasyWebTarget)target;

CustomerResource customerProxy = target.proxy(CustomerResource.class);
```


If you are using RESTEasy as your JAX-RS implementation, all you have to do is typecast an instance of **WebTarget** to **ResteasyWebTarget**. You can then invoke the **ResteasyWebTarget.proxy()** method. This method returns an instance of the **CustomerResource** interface that you can invoke on. Here’s the proxy in use:


```Java
// Create a customer
Customer newCust = new Customer();
newCust.setName("bill");
Response response = customerProxy.createCustomer(newCust);

// Get a customer
Customer cust = customerProxy.getCustomer(333);

// Update a customer
cust.setName("burke");
customerProxy.updateCustomer(333, cust);
```


When you invoke one of the methods of the returned **CustomerResource** proxy, it converts the Java method call into an HTTP request to the server using the metadata defined in the annotations applied to the **CustomerResource** interface. For example, the **getCustomer()** invocation in the example code knows that it must do a GET request on the *http://example.com/customers/333* URI, because it has introspected the values of the **@Path**, **@GET**, and **@PathParam** annotations on the method. It knows that it should be getting back XML from the **@Produces** annotation. It also knows that it should unmarshal it using a JAXB **MessageBodyReader**, because the **getCustomer()** method returns a JAXB annotated class.



### Advantages and Disadvantages


A nice side effect of writing Java clients with this proxy framework is that you can use the Java interface for Java clients and JAX-RS services. With one Java interface, you also have a nice, clear way of documenting how to interact with your RESTful Java service. As you can see from the example code, it also cuts down on a lot of boilerplate code. The disadvantage, of course, is that this framework, while open source, is proprietary.
