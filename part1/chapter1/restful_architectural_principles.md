# RESTful Architectural Principles


Roy Fielding’s PhD thesis describing REST was really an explanation of why the human-readable Web had become so pervasive in the past 18 years. As time went on, though, programmers started to realize that they could use the concepts of REST to build distributed services and model service-oriented architectures (SOAs).


The idea of SOA is that application developers design their systems as a set of reusable, decoupled, distributed services. Since these services are published on the network, conceptually, it should be easier to compose larger and more complex systems. SOA has been around for a long time. Developers have used technologies like DCE, CORBA, and Java RMI to build them in the past. Nowadays, though, when you think of SOA, you think of SOAP-based web services.


While REST has many similarities to the more traditional ways of writing SOA applications, it is very different in many important ways. You would think that a background in distributed computing would be an asset to understanding this new way of creating web services, but unfortunately this is not always the case. The reason is that some of the concepts of REST are hard to swallow, especially if you have written successful SOAP or CORBA applications. If your career has a foundation in one of these older technologies, there’s a bit of emotional baggage you will have to overcome. For me, it took a few months of reading, researching, and intense arguing with REST evangelists (aka RESTafarians). For you, it may be easier. Others will never pick REST over something like SOAP and WS-*.


Let’s examine each of the architectural principles of REST in detail and why they are important when you are writing a web service.


### Addressability


Addressability is the idea that every object and resource in your system is reachable through a unique identifier. This seems like a no-brainer, but if you think about it, standardized object identity isn’t available in many environments. If you have tried to implement a portable J2EE application, you probably know what I mean. In J2EE, distributed and even local references to services are not standardized, which makes portability really difficult. This isn’t such a big deal for one application, but with the new popularity of SOA, we’re heading to a world where disparate applications must integrate and interact. Not having something as simple as standardized service addressability adds a whole complex dimension to integration efforts.


In the REST world, addressability is managed through the use of URIs. When you make a request for information in your browser, you are typing in a URI. Each HTTP request must contain the URI of the object you are requesting information from or posting information to. The format of a URI is standardized as follows:

```
scheme://host:port/path?queryString#fragment
```


The **scheme** is the protocol you are using to communicate with. For RESTful web services, it is usually **http** or **https**. The **host** is a DNS name or IP address. It is followed by an optional **port**, which is numeric. The **host** and **port** represent the location of your resource on the network. Following **host** and **port** is a **path** expression. This **path** expression is a set of text segments delimited by the “/” character. Think of the **path** expression as a directory list of a file on your machine. Following the path expression is an optional query string. The “?” character separates the path from the query string. The query string is a list of parameters represented as name/value pairs. Each pair is delimited with the “&” character. Here’s an example query string within a URI:

```
http://example.com/customers?lastName=Burke&zipcode=02115
```


A specific parameter name can be repeated in the query string. In this case, there are multiple values for the same parameter.


The last part of the URI is the fragment. It is delimited by a “#” character. The fragment is usually used to point to a certain place in the document you are querying.


Not all characters are allowed within a URI string. Some characters must be encoded using the following rules. The characters a–z, A–Z, 0–9, ., -, *, and _ remain the same. The space character is converted to +. The other characters are first converted into a sequence of bytes using a specific encoding scheme. Next, a two-digit hexadecimal number prefixed by % represents each byte.


Using a unique URI to identify each of your services makes each of your resources linkable. Service references can be embedded in documents or even email messages. For instance, consider the situation where somebody calls your company’s help desk with a problem related to your SOA application. A link could represent the exact problem the user is having. Customer support can email the link to a developer who can fix the problem. The developer can reproduce the problem by clicking on the link. Furthermore, the data that services publish can also be composed into larger data streams fairly easily:


```
<order id="111">
   <customer>http://customers.myintranet.com/customers/32133</customer>
   <order-entries>
     <order-entry>
        <quantity>5</quantity>
        <product>http://products.myintranet.com/products/111</product>
...
```