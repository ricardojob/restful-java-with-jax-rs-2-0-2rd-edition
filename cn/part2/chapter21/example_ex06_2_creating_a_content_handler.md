# Example ex06_2: Creating a Content Handler


For this example, we’re going to create something entirely new. The [Chapter 6](../../part1/chapter6/jax_rs_content_handlers.md) example of a content handler is a reimplementation of JAXB support. It is suitable for that chapter because it illustrates both the writing of a **MessageBodyReader** and a **MessageBodyWriter** and demonstrates how the **ContextResolver** is used. For *ex06_2*, though, we’re going to keep things simple.


In *ex06_2*, we’re going to rewrite *ex06_1* to exchange Java objects between the client and server instead of XML. Java objects, you ask? Isn’t this REST? Well, there’s no reason a Java object can’t be a valid representation of a resource! If you’re exchanging Java objects, you can still realize a lot of the advantages of REST and HTTP. You still can do content negotiation (described in [Chapter 9](../../part1/chapter9/http_content_negotiation.md)) and HTTP caching (described in [Chapter 11](../../part1/chapter11/scaling_jax_rs_applications.md)).



### The Content Handler Code


For our Java object content handler, we’re going to write one class that is both a **MessageBodyReader** and a **MessageBodyWriter**:


```Java:src/main/java/com/restfully/shop/services/JavaMarshaller.java
@Provider
@Produces("application/example-java")
@Consumes("application/x-java-serialized-object")
public class JavaMarshaller
                     implements MessageBodyReader, MessageBodyWriter
{
```


The **JavaMarshaller** class is annotated with **@Provider**, **@Produces**, and **@Consumes**, as required by the specification. The media type used by the example to represent a Java object is **application/example-java**:[^22]


```Java
   public boolean isReadable(Class type, Type genericType,
   Annotation[] annotations, MediaType mediaType)
   {
      return Serializable.class.isAssignableFrom(type);
   }

   public boolean isWriteable(Class type, Type genericType,
                      Annotation[] annotations, MediaType mediaType)
   {
      return Serializable.class.isAssignableFrom(type);
   }
```


For the **isReadable()** and **isWriteable()** methods, we just check to see if our Java type implements the **java.io.Serializable** interface:



```Java
   public Object readFrom(Class type, Type genericType,
                       Annotation[] annotations, MediaType mediaType,
                          MultivaluedMap httpHeaders,
                            InputStream is)
                     throws IOException, WebApplicationException
   {
      ObjectInputStream ois = new ObjectInputStream(is);
      try
      {
         return ois.readObject();
      }
      catch (ClassNotFoundException e)
      {
         throw new RuntimeException(e);
      }
   }
```


The **readFrom()** method uses basic Java serialization to read a Java object from the HTTP input stream:



```Java
   public long getSize(Object o, Class type,
                     Type genericType, Annotation[] annotations,
                      MediaType mediaType)
   {
      return −1;
   }
```



The **getSize()** method returns –1. It is impossible to figure out the exact length of our marshalled Java object without serializing it into a byte buffer and counting the number of bytes. We’re better off letting the servlet container figure this out for us. The **getSize()** method has actually been deprecated in JAX-RS 2.0.


```Java
   public void writeTo(Object o, Class type,
                        Type genericType, Annotation[] annotations,
                         MediaType mediaType,
                          MultivaluedMap httpHeaders, OutputStream os)
                          throws IOException, WebApplicationException
   {
      ObjectOutputStream oos = new ObjectOutputStream(os);
      oos.writeObject(o);
   }
```


Like the **readFrom()** method, basic Java serialization is used to marshal our Java object into the HTTP response body.


### The Resource Class


The **CustomerResource** class doesn’t change much from *ex06_2*:


```Java:src/main/java/com/restfully/shop/services/CustomerResource.java
@Path("/customers")
public class CustomerResource
{
...

@POST
   @Consumes("application/example-java
")
   public Response createCustomer(Customer customer)
   {
      customer.setId(idCounter.incrementAndGet());
      customerDB.put(customer.getId(), customer);
      System.out.println("Created customer " + customer.getId());
      return Response.created(URI.create("/customers/"
                                + customer.getId())).build();

   }
...
}
```


The code is actually exactly the same as that used in *ex06_1*, except that the **@Produces** and **@Consumes** annotations use the **application/example-java** media type.



### The Application Class


The **ShoppingApplication** class needs to change a tiny bit from the previous examples:


```Java:src/main/java/com/restfully/shop/services/ShoppingApplication.java
public class ShoppingApplication extends Application {
   private Set<Object> singletons = new HashSet<Object>();
   private Set<Class<?>> classes = new HashSet<Class<?>>();

   public ShoppingApplication() {
      singletons.add(new CustomerResource());
      classes.add(JavaMarshaller.class);
   }

   @Override
   public Set<Class<?>> getClasses() {
      return classes;
   }

   @Override
   public Set<Object> getSingletons() {
      return singletons;
   }
}
```


For our **Application** class, we need to register the **JavaMarshaller** class. If we don’t, the JAX-RS runtime won’t know how to handle the **application/example-java** media type.



### The Client Code



The client code isn’t much different from *ex06_1*. We just need to modify the **javax.ws.rs.client.Entity** construction to use the **application/example-java** media type. For example, here’s what customer creation looks like:


```Java:src/test/java/com/restfully/shop/test/CustomerResourceTest.java
public class CustomerResourceTest
{
   @Test
   public void testCustomerResource() throws Exception
   {
...
      Response response = client.target
              ("http://localhost:8080/services/customers")
              .request().post(Entity.entity
                             (newCustomer, "application/example-java"));
```


The static **Entity.entity()** method is called, passing in the plain **Customer** Java object along with our custom media type.


### Build and Run the Example Program


Perform the following steps:

1. Open a command prompt or shell terminal and change to the *ex06_2* directory of the workbook example code. 
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md). 
3. Perform the build and run the example by typing **maven install**.

---
[^22] The media type **application/x-java-serialized-object** should actually be used, but as of the second revision of this book, RESTEasy now has this type built in.