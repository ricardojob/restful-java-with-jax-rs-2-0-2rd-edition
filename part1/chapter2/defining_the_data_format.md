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


The link[^2] element tells any client that obtains an XML document describing one of the objects in our ecommerce system where on the network the client can interact with that particular resource. The rel attribute tells the client what relationship the link has with the resource the URI points to (contained within the href attribute). The self value just means it is pointing to itself. While not that interesting on its own, link becomes very useful when we aggregate or compose information into one larger XML document.




[^2]  I actually borrowed the link element from the Atom format. Atom is a syndication format that is used to aggregate and publish blogs and news feeds. You can find out more about Atom at http://www.w3.org/2005/Atom.