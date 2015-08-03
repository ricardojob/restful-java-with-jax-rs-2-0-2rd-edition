# Defining the Data Format


One of the most important things we have to do when defining a RESTful interface is determine how our resources will be represented over the wire to our clients. XML is perhaps one of the most popular formats on the Web and can be processed by most modern languages, so let’s choose that. JSON is also a popular format, as it is more condensed and JavaScript can interpret it directly (great for Ajax applications), but let’s stick to XML for now.


Generally, you would define an XML schema for each representation you want to send across the wire. An XML schema defines the grammar of a data format. It defines the rules about how a document can be put together. I do find, though, that when explaining things within an article (or a book), providing examples rather than schema makes things much easier to read and understand.


### Read and Update Format


The XML format of our representations will look a tiny bit different when we read or update resources from the server as compared to when we create resources on the server. Let’s look at our read and update format first.


#### Common link element


Each format for **Order**, **Customer**, and **Product** will have a common XML element called link:

```xml
<link rel="self" href="http://example.com/..."/>
```


The **link**[^2] element tells any client that obtains an XML document describing one of the objects in our ecommerce system where on the network the client can interact with that particular resource. The **rel** attribute tells the client what relationship the link has with the resource the URI points to (contained within the **href** attribute). The self value just means it is pointing to itself. While not that interesting on its own, **link** becomes very useful when we aggregate or compose information into one larger XML document.

#### The details

So, with the common elements described, let’s start diving into the details by first looking at our **Customer** representation format:

```xml
<customer id="117">
   <link rel="self" href="http://example.com/customers/117"/>
   <first-name>Bill</first-name>
   <last-name>Burke</last-name>
   <street>555 Beacon St.<street>
   <city>Boston</city>
   <state>MA</state>
   <zip>02115</zip>
</customer>
```

Pretty straightforward. We just take the object model of **Customer** from *Figure 2-1* and expand its attributes as XML elements. **Product** looks much the same in terms of simplicity:

```xml
<product id="543">
   <link rel="self" href="http://example.com/products/543"/>
   <name>iPhone</name>
   <cost>$199.99</cost>
</product>
```


In a real system, we would, of course, have a lot more attributes for **Customer** and **Product**, but let’s keep our example simple so that it’s easier to illustrate these RESTful concepts:


```xml
<order id="233">
   <link rel="self" href="http://example.com/orders/233"/>
   <total>$199.02</total>
   <date>December 22, 2008 06:56</date>
   <customer id="117">
      <link rel="self" href="http://example.com/customers/117"/>
      <first-name>Bill</first-name>
      <last-name>Burke</last-name>
      <street>555 Beacon St.<street>
      <city>Boston</city>
      <state>MA</state>
      <zip>02115</zip>
   </customer>
   <line-items>
      <line-item id="144">
         <product id="543">
             <link rel="self" href="http://example.com/products/543"/>
             <name>iPhone</name>
             <cost>$199.99</cost>
         </product>
         <quantity>1</quantity>
      </line-item>
   </line-items>
</order>
```

The **Order** data format has the top-level elements of **total** and **date** that specify the total cost of the order and the date the **Order** was made. **Order** is a great example of data composition, as it includes **Customer** and **Product** information. This is where the **link** element becomes particularly useful. If the client is interested in interacting with a **Customer** or **Product** that makes up the **Order**, it has the URI needed to interact with one of these resources.



### Create Format


When we are creating new **Orders**, **Customers**, or **Products**, it doesn’t make a lot of sense to include an **id** attribute and **link** element with our XML document. The server will generate IDs when it inserts our new object into a database. We also don’t know the URI of a new object because the server also generates this. So, the XML for creating a new **Product** would look something like this:


```xml
<product>
   <name>iPhone</name>
   <cost>$199.99</cost>
</product>
```

**Orders** and **Customers** would follow the same pattern and leave out the **id** attribute and **link** element.

---
[^2]  I actually borrowed the link element from the Atom format. Atom is a syndication format that is used to aggregate and publish blogs and news feeds. You can find out more about Atom at http://www.w3.org/2005/Atom.