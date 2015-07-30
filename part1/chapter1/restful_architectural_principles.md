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

In this example, an XML document describes an ecommerce order entry. We can reference data provided by different divisions in a company. From this reference, we can not only obtain information about the linked customer and products that were bought, but we also have the identifier of the service this data comes from. We know exactly where we can further interact and manipulate this data if we so desired.


### The Uniform, Constrained Interface


The REST principle of a constrained interface is perhaps the hardest pill for an experienced CORBA or SOAP developer to swallow. The idea behind it is that you stick to the finite set of operations of the application protocol you’re distributing your services upon. This means that you don’t have an “action” parameter in your URI and use only the methods of HTTP for your web services. HTTP has a small, fixed set of operational methods. Each method has a specific purpose and meaning. Let’s review them:

* GET

    GET is a read-only operation. It is used to query the server for specific information. It is both an idempotent and safe operation. Idempotent means that no matter how many times you apply the operation, the result is always the same. The act of reading an HTML document shouldn’t change the document. Safe means that invoking a GET does not change the state of the server at all. This means that, other than request load, the operation will not affect the server.

* PUT

    PUT requests that the server store the message body sent with the request under the location provided in the HTTP message. It is usually modeled as an insert or update. It is also idempotent. When using PUT, the client knows the identity of the resource it is creating or updating. It is idempotent because sending the same PUT message more than once has no effect on the underlying service. An analogy is an MS Word document that you are editing. No matter how many times you click the Save button, the file that stores your document will logically be the same document. 

* DELETE

    DELETE is used to remove resources. It is idempotent as well.

* POST

    POST is the only nonidempotent and unsafe operation of HTTP. Each POST method is allowed to modify the service in a unique way. You may or may not send information with the request. You may or may not receive information from the response. 

* HEAD

    HEAD is exactly like GET except that instead of returning a response body, it returns only a response code and any headers associated with the request. 

* OPTIONS

    OPTIONS is used to request information about the communication options of the resource you are interested in. It allows the client to determine the capabilities of a server and a resource without triggering any resource action or retrieval.


There are other HTTP methods (like TRACE and CONNECT), but they are unimportant when you are designing and implementing RESTful web services.


You may be scratching your head and thinking, “How is it possible to write a distributed service with only four to six methods?” Well…SQL only has four operations: SELECT, INSERT, UPDATE, and DELETE. JMS and other message-oriented middleware (MOM) really only have two logical operations: send and receive. How powerful are these tools? For both SQL and JMS, the complexity of the interaction is confined purely to the data model. The addressability and operations are well defined and finite, and the hard stuff is delegated to the data model (in the case of SQL) or the message body (in the case of JMS).


### Why Is the Uniform Interface Important?

Constraining the interface for your web services has many more advantages than disadvantages. Let’s look at a few:

* Familiarity 

    If you have a URI that points to a service, you know exactly which methods are available on that resource. You don’t need an IDL-like file describing which methods are available. You don’t need stubs. All you need is an HTTP client library. If you have a document that is composed of links to data provided by many different services, you already know which method to call to pull in data from those links.

* Interoperability

    HTTP is a very ubiquitous protocol. Most programming languages have an HTTP client library available to them. So, if your web service is exposed over HTTP, there is a very high probability that people who want to use your service will be able to do so without any additional requirements beyond being able to exchange the data formats the service is expecting. With CORBA or SOAP, you have to install vendor-specific client libraries as well as loads and loads of IDL- or WSDL-generated stub code. How many of you have had a problem getting CORBA or WS-\* vendors to interoperate? It has traditionally been very problematic. The WS-* set of specifications has also been a moving target over the years. So with WS-* and CORBA, you not only have to worry about vendor interoperability, but you also have to make sure that your client and server are using the same specification version of the protocol. With REST over HTTP, you don’t have to worry about either of these things and can just focus on understanding the data format of the service. I like to think that you are focusing on what is really important: *application interoperability*, rather than *vendor interoperability*.
    
