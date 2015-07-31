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










































