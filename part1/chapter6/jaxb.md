# JAXB


JAXB is an older Java specification and is not defined by JAX-RS. JAXB is an annotation framework that maps Java classes to XML and XML schema. It is extremely useful because instead of interacting with an abstract representation of an XML document, you can work with real Java objects that are closer to the domain you are modeling. JAX-RS has built-in support for JAXB, but before we review these handlers, let’s get a brief overview of the JAXB framework.


### Intro to JAXB


A whole book could be devoted to explaining the intricacies of JAXB, but I’m only going to focus here on the very basics of the framework. If you want to map an existing Java class to XML using JAXB, there are a few simple annotations you can use. Let’s look at an example:


```Java
@XmlRootElement(name="customer")
@XmlAccessorType(XmlAccessType.FIELD)
public class Customer {

    @XmlAttribute
    protected int id;

    @XmlElement
    protected String fullname;

    public Customer() {}

    public int getId() { return this.id; }
    public void setId(int id) { this.id = id; }

    public String getFullName() { return this.fullname; }
    public void setFullName(String name} { this.fullname = name; }
}
```


The **@javax.xml.bind.annotation.XmlRootElement** annotation is put on Java classes to denote that they are XML elements. The **name()** attribute of **@XmlRootElement** specifies the string to use for the name of the XML element. In our example, the annotation **@XmlRootElement** specifies that our **Customer** objects should be marshalled into an XML element named **&lt;customer&gt;**.


The **@javax.xml.bind.annotation.XmlAttribute** annotation was placed on the **id** field of our **Customer** class. This annotation tells JAXB to map the field to an **id** attribute on the main **&lt;customer&gt;** element of the XML document. The **@XmlAttribute** annotation also has a **name()** attribute that allows you to specify the exact name of the XML attribute within the XML document. By default, it is the same name as the annotated field.


In our example, the **@javax.xml.bind.annotation.XmlElement** annotation was placed on the **fullname** field of our **Customer** class. This annotation tells JAXB to map the field to a **&lt;fullname&gt;** element within the main **&lt;customer&gt;** element of the XML document. **@XmlElement** does have a **name()** attribute, so you can specify the exact string of the XML element. By default, it is the same name as the annotated field.


If we were to output an instance of our **Customer** class that had an **id** of 42 and a **name** of “Bill Burke,” the outputted XML would look like this:


```xml
<customer id="42">
   <fullname>Bill Burke</fullname>
</customer>
```


You can also use the **@XmlElement** annotation to embed other JAXB-annotated classes. For example, let’s say we wanted to add an **Address** class to our **Customer** class:


```Java
@XmlRootElement(name="address")
@XmlAccessorType(XmlAccessType.FIELD)
public class Address  {

   @XmlElement
   protected String street;

   @XmlElement
   protected String city;

   @XmlElement
   protected String state;

   @XmlElement
   protected String zip;

   // getters and setters

   ...
}
```


We would simply add a field to **Customer** that was of type **Address** as follows:


```Java
@XmlRootElement(name="customer")
@XmlAccessorType(XmlAccessType.FIELD)
public class Customer {

    @XmlAttribute
    protected int id;

    @XmlElement
    protected String name;

    @XmlElement
    protected Address address;

    public Customer() {}

    public int getId() { return this.id; }
    public void setId(int id) { this.id = id; }
...
}
```

If we were to output an instance of our new **Customer** class that had an **id** of 42, a **name** of “Bill Burke,” a **street** of “200 Marlborough Street,” a **city** of “Boston,” a **state** of “MA,” and a **zip** of “02115,” the outputted XML would look like this:


```xml
<customer id="42">
   <name>Bill Burke</name>
   <address>
      <street>200 Marlborough Street</street>
      <city>Boston</city>
      <state>MA</state>
      <zip>02115</zip>
   </address>
</customer>
```

There are a number of other annotations and settings that allow you to do some more complex Java-to-XML mappings. JAXB implementations are also required to have command-line tools that can automatically generate JAXB-annotated Java classes from XML schema documents. If you need to integrate with an existing XML schema, these autogeneration tools are the way to go.


To marshal Java classes to and from XML, you need to interact with the **javax.xml.bind.JAXBContext** class. **JAXBContext** instances introspect your classes to understand the structure of your annotated classes. They are used as factories for the **javax.xml.bind.Marshaller** and **javax.xml.bind.Unmarshaller** interfaces. **Marshaller** instances are used to take Java objects and output them as XML. **Unmarshaller** instances are used to take XML input and create Java objects out of it. Here’s an example of using JAXB to write an instance of the Customer class we defined earlier into XML and then to take that XML and re-create the **Customer** object:


```Java
Customer customer = new Customer();
customer.setId(42);
customer.setName("Bill Burke");

JAXBContext ctx = JAXBContext.newInstance(Customer.class);
StringWriter writer = new StringWriter();

ctx.createMarshaller().marshal(customer, writer);

String custString = writer.toString();

customer = (Customer)ctx.createUnmarshaller()
              .unmarshal(new StringReader(custString));
```


We first create an initialized instance of a **Customer** class. We then initialize a **JAXBContext** to understand how to deal with **Customer** classes. We use a **Marshaller** instance created by the method **JAXBContext.createMarshaller()** to write the **Customer** object into a Java string. Next we use the **Unmarshaller** created by the **JAXBContext.createUnmarshaller()** method to re-create the **Customer** object with the XML string we just created.


Now that we have a general idea of how JAXB works, let’s look at how JAX-RS integrates with it.


### JAXB JAX-RS Handlers


The JAX-RS specification requires implementations to automatically support the marshalling and unmarshalling of classes that are annotated with **@XmlRootElement** or **@XmlType** as well as objects wrapped inside **javax.xml.bind.JAXBElement** instances. Here’s an example that interacts using the **Customer** class defined earlier:


```Java
@Path("/customers")
public class CustomerResource {

   @GET
   @Path("{id}")
   @Produces("application/xml")
   public Customer getCustomer(@PathParam("id") int id) {

      Customer cust = findCustomer(id);
      return cust;
   }

   @POST
   @Consumes("application/xml")
   public void createCustomer(Customer cust) {
      ...
   }
}
```


As you can see, once you’ve applied JAXB annotations to your Java classes, it is very easy to exchange XML documents between your client and web services. The built-in JAXB handlers will handle any JAXB-annotated class for the **application/xml**, **text/xml**, or **application/\*+xml** media types. By default, they will also manage the creation and initialization of JAXBContext instances. Because the creation of JAXBContext instances can be expensive, JAX-RS implementations usually cache them after they are first initialized.


#### Managing your own JAXBContexts with ContextResolvers


If you are already familiar with JAXB, you’ll know that many times you need to configure your **JAXBContext** instances a certain way to get the output you desire. The JAX-RS built-in JAXB provider allows you to plug in your own **JAXBContext** instances. The way it works is that you have to implement a factory-like interface called **javax.ws.rs.ext.ContextResolver** to override the default **JAXBContext** creation:


```Java
public interface ContextResolver<T> {

   T getContext(Class<?> type);
}
```


**ContextResolvers** are pluggable factories that create objects of a specific type, for a certain Java type, and for a specific media type. To plug in your own **JAXBContext**, you will have to implement this interface. Here’s an example of creating a specific **JAXBContext** for our **Customer** class:



```Java
@Provider
@Produces("application/xml")
public class CustomerResolver
                      implements ContextResolver<JAXBContext> {
   private JAXBContext ctx;

   public CustomerResolver() {
     this.ctx = ...; // initialize it the way you want
   }


   public JAXBContext getContext(Class<?> type) {
      if (type.equals(Customer.class)) {
         return ctx;
      } else {
         return null;
      }
   }
}
```

Your resolver class must implement **ContextResolver** with the parameterized type of **JAXBContext**. The class must also be annotated with the **@javax.ws.rs.ext.Provider** annotation to identify it as a JAX-RS component. In our example, the **CustomerResolver** constructor initializes a **JAXBContext** specific to our **Customer** class.


You register your **ContextResolver** using the **javax.ws.rs.core.Application** API discussed in Chapters [3](../chapter3/your_first_jax_rs_service.md) and [14](../../part1/chapter14/ejb_integration.md). The built-in JAXB handler will see if there are any registered **ContextResolvers** that can create **JAXBContext** instances. It will iterate through them, calling the **getContext()** method passing in the Java type it wants a **JAXBContext** created for. If the **getContext()** method returns **null**, it will go on to the next **ContextResolver** in the list. If the **getContext()** method returns an instance, it will use that **JAXBContext** to handle the request. If there are no **ContextResolvers** found, it will create and manage its own **JAXBContext**. In our example, the **CustomerResolver.getContext()** method checks to see if the type is a **Customer** class. If it is, it returns the **JAXBContext** we initialized in the constructor; otherwise, it returns **null**.


The **@Produces** annotation on your **CustomerResolver** implementation is optional. It allows you to specialize a **ContextResolver** for a specific media type. You’ll see in the next section that you can use JAXB to output to formats other than XML. This is a way to create **JAXBContext** instances for each individual media type in your system.


### JAXB and JSON


JAXB is flexible enough to support formats other than XML. The Jettison[^5] open source project has written a JAXB adapter that can input and output the JSON format. JSON is a text-based format that can be directly interpreted by JavaScript. It is the preferred exchange format for Ajax applications. Although not required by the JAX-RS specification, many JAX-RS implementations use Jettison to support marshalling JAXB annotated classes to and from JSON.


JSON is a much simpler format than XML. Objects are enclosed in curly brackets, “{}”, and contain key/value pairs. Values can be quoted strings, Booleans (true or false), numeric values, or arrays of these simple types. Here’s an example:


```
{
  "id" : 42,
  "name" : "Bill Burke",
  "married" : true,
  "kids" : [ "Molly", "Abby" ]
}
```


Key and value pairs are separated by a colon character and delimited by commas. Arrays are enclosed in brackets, “[].” Here, our object has four properties—**id**, **name**, **married**, and **kids**—with varying values.


#### XML to JSON using BadgerFish


As you can see, JSON is a much simpler format than XML. While XML has elements, attributes, and namespaces, JSON only has name/value pairs. There has been some work in the JSON community to produce a mapping between XML and JSON so that one XML schema can output documents of both formats. The de facto standard, BadgerFish, is a widely used XML-to-JSON mapping and is available in most JAX-RS implementations that have JAXB/JSON support. Let’s go over this mapping:

1. XML element names become JSON object properties and the text values of these elements are contained within a nested object that has a property named “$.” So, if you had the XML **&lt;customer&gt;Bill Burke&lt;/customer&gt;**, it would map to { "customer" : { "$" : "Bill Burke" }}. 


































---
[^5] For more information, see http://jettison.codehaus.org.

