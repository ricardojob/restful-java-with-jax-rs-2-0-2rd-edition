# Leveraging Content Negotiation


Most of the examples so far in this chapter have used conneg simply to differentiate between well-known media types like XML and JSON. While this is very useful to help service different types of clients, it’s not the main purpose of conneg. Your web services will evolve over time. New features will be added. Expanded datasets will be offered. Data formats will change and evolve. How do you manage these changes? How can you manage older clients that can only work with older versions of your services? Modeling your application design around conneg can address a lot of these issues. Let’s discuss some of the design decisions you must make to leverage conneg when designing and building your applications.


### Creating New Media Types


An important principle of REST is that the complexities of your resources are encapsulated within the data formats you are exchanging. While location information (URIs) and protocol methods remain fixed, data formats can evolve. This is a very important thing to remember and consider when you are planning how your web services are going to handle versioning.


Since complexity is confined to your data formats, clients can use media types to ask for different format versions. A common way to address this is to design your applications to define their own new media types. The convention is to combine a **vnd** prefix, the name of your new format, and a concrete media type suffix delimited by the “+” character. For example, let’s say the company Red Hat had a specific XML format for its customer database. The media type name might look like this:


```
application/vnd.rht.customers+xml
```

The **vnd** prefix stands for vendor. The **rht** string in this example represents Red Hat and, of course, the **customers** string represents our customer database format. We end it with **+xml** to let users know that the format is XML based. We could do the same with JSON as well:


```
application/vnd.rht.customers+json
```


Now that we have a base media type name for the Red Hat format, we can append versioning information to it so that older clients can still ask for older versions of the format:


```
application/vnd.rht.customers+xml;version=1.0
```


Here, we’ve kept the subtype name intact and used media type properties to specify version information. Specifying a version property within a custom media type is a common pattern to denote versioning information. As this customer data format evolves over time, we can bump the version number to support newer clients without breaking older ones.


### Flexible Schemas


Using media types to version your web services and applications is a great way to mitigate and manage change as your web services and applications evolve over time. While embedding version information within the media type is extremely useful, it shouldn’t be the primary way you manage change. When defining the initial and newer versions of your data formats, you should pay special attention to backward compatibility.


Take, for instance, your initial schema should allow for extended or custom elements and attributes within each and every schema type in your data format definition. Here’s the initial definition of a customer data XML schema:


```xml
<schema targetNamespace="http://www.example.org/customer"
           xmlns="http://www.w3.org/2001/XMLSchema">
   <element name="customer" type="customerType"/>
   <complexType name="customerType">
      <attribute name="id" use="required" type="string"/>
      <anyAttribute/>
      <element name="first" type="string" minOccurs="1"/>
      <element name="last" type="string" minOccurs="1"/>
      <any/>
   </complexType>
</schema>
```


In this example, the schema allows for adding any arbitrary attribute to the **customer** element. It also allows documents to contain any XML element in addition to the **first** and **last** elements. If new versions of the customer XML data format retain the initial data structure, clients that use the older version of the schema can still validate and process newer versions of the format as they receive them.


As the schema evolves, new attributes and elements can be added, but they should be made optional. For example:


```xml
<schema targetNamespace="http://www.example.org/customer"
           xmlns="http://www.w3.org/2001/XMLSchema">
   <element name="customer" type="customerType"/>
   <complexType name="customerType">
      <attribute name="id" use="required" type="string"/>
      <anyAttribute/>
      <element name="first" type="string" minOccurs="1"/>
      <element name="last" type="string" minOccurs="1"/>
      <element name="street" type="string" minOccurs="0"/>
      <element name="city" type="string" minOccurs="0"/>
      <element name="state" type="string" minOccurs="0"/>
      <element name="zip" type="string" minOccurs="0"/>
      <any/>
   </complexType>
</schema>
```


Here, we have added the street, city, state, and zip elements to our schema, but have made them optional. This allows older clients to still PUT and POST older, yet valid, versions of the data format.


If you combine flexible, backward-compatible schemas with media type versions, you truly have an evolvable system of data formats. Clients that are version-aware can use the media type version scheme to request specific versions of your data formats. Clients that are not version-aware can still request and send the version of the format they understand.

