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

    HTTP is a very ubiquitous protocol. Most programming languages have an HTTP client library available to them. So, if your web service is exposed over HTTP, there is a very high probability that people who want to use your service will be able to do so without any additional requirements beyond being able to exchange the data formats the service is expecting. With CORBA or SOAP, you have to install vendor-specific client libraries as well as loads and loads of IDL- or WSDL-generated stub code. How many of you have had a problem getting CORBA or WS-\* vendors to interoperate? It has traditionally been very problematic. The WS-\* set of specifications has also been a moving target over the years. So with WS-\* and CORBA, you not only have to worry about vendor interoperability, but you also have to make sure that your client and server are using the same specification version of the protocol. With REST over HTTP, you don’t have to worry about either of these things and can just focus on understanding the data format of the service. I like to think that you are focusing on what is really important: *application interoperability*, rather than *vendor interoperability*.
    

* Scalability

    Because REST constrains you to a well-defined set of methods, you have predictable behavior that can have incredible performance benefits. GET is the strongest example. When surfing the Internet, have you noticed that the second time you browse to a specific page it comes up faster? This is because your browser caches already visited pages and images. HTTP has a fairly rich and configurable protocol for defining caching semantics. Because GET is a read method that is both idempotent and safe, browsers and HTTP proxies can cache responses to servers, and this can save a huge amount of network traffic and hits to your website. Add HTTP caching semantics to your web services, and you have an incredibly rich way of defining caching policies for your services. We will discuss HTTP caching in detail within [Chapter 11](../chapter11/scaling_jax_rs_applications.md). 



It doesn’t end with caching, though. Consider both PUT and DELETE. Because they are idempotent, neither the client nor the server has to worry about handling duplicate message delivery. This saves a lot of bookkeeping and complex code.



### Representation-Oriented


The third architectural principle of REST is that your services should be representation-oriented. Each service is addressable through a specific URI and representations are exchanged between the client and service. With a GET operation, you are receiving a representation of the current state of that resource. A PUT or POST passes a representation of the resource to the server so that the underlying resource’s state can change.


In a RESTful system, the complexity of the client-server interaction is within the representations being passed back and forth. These representations could be XML, JSON, YAML, or really any format you can come up with.


With HTTP, the representation is the message body of your request or response. An HTTP message body may be in any format the server and client want to exchange. HTTP uses the **Content-Type** header to tell the client or server what data format it is receiving. The Content-Type header value string is in the Multipurpose Internet Mail Extension (MIME) format. The MIME format is very simple:

```shell
type/subtype;name=value;name=value...
```

**type** is the main format family and subtype is a category. Optionally, the MIME type can have a set of name/value pair properties delimited by the “;” character. Some examples are:

```shell
text/plain
text/html
application/xml
text/html; charset=iso-8859-1
```


One of the more interesting features of HTTP that leverages MIME types is the capability of the client and server to negotiate the message formats being exchanged between them. While not used very much by your browser, HTTP content negotiation is a very powerful tool when you’re writing web services. With the **Accept** header, a client can list its preferred response formats. Ajax clients can ask for JSON, Java for XML, Ruby for YAML. Another thing this is very useful for is versioning of services. The same service can be available through the same URI with the same methods (GET, POST, etc.), and all that changes is the MIME type. For example, the MIME type could be **application/vnd+xml** for an old service, while newer services could exchange **application/vnd+xml;version=1.1** MIME types. You can read more about these concepts in [Chapter 9](../chapter9/http_content_negotiation.md).


All in all, because REST and HTTP have a layered approach to addressability, method choice, and data format, you have a much more decoupled protocol that allows your service to interact with a wide variety of clients in a consistent way.



### Communicate Statelessly


The fourth RESTful principle I will discuss is the idea of statelessness. When I talk about statelessness, though, I don’t mean that your applications can’t have state. In REST, stateless means that there is no client session data stored on the server. The server only records and manages the state of the resources it exposes. If there needs to be session-specific data, it should be held and maintained by the client and transferred to the server with each request as needed. A service layer that does not have to maintain client sessions is a lot easier to scale, as it has to do a lot fewer expensive replications in a clustered environment. It’s a lot easier to scale up because all you have to do is add machines.


A world without server-maintained session data isn’t so hard to imagine if you look back 12–15 years ago. Back then, many distributed applications had a fat GUI client written in Visual Basic, Power Builder, or Visual C++ talking RPCs to a middle tier that sat in front of a database. The server was stateless and just processed data. The fat client held all session state. The problem with this architecture was an IT operations one. It was very hard for operations to upgrade, patch, and maintain client GUIs in large environments. Web applications solved this problem because the applications could be delivered from a central server and rendered by the browser. We started maintaining client sessions on the server because of the limitations of the browser. Around 2008, in step with the growing popularity of Ajax, Flex, and Java FX, the browsers became sophisticated enough to maintain their own session state like their fat-client counterparts in the mid-’90s used to do. We can now go back to that stateless scalable middle tier that we enjoyed in the past. It’s funny how things go full circle sometimes.



### HATEOAS


The final principle of REST is the idea of using Hypermedia As The Engine Of Application State (HATEOAS). Hypermedia is a document-centric approach with added support for embedding links to other services and information within that document format. I did indirectly talk about HATEOAS in Addressability when I discussed the idea of using hyperlinks within the data format received from a service.


One of the uses of hypermedia and hyperlinks is composing complex sets of information from disparate sources. The information could be within a company intranet or dispersed across the Internet. Hyperlinks allow us to reference and aggregate additional data without bloating our responses. The ecommerce order in Addressability is an example of this:

```
<order id="111">
   <customer>http://customers.myintranet.com/customers/32133</customer>
   <order-entries>
     <order-entry>
        <quantity>5</quantity>
        <product>http://products.myintranet.com/products/111</product>
...
```

In that example, links embedded within the document allowed us to bring in additional information as needed. Aggregation isn’t the full concept of HATEOAS, though. The more interesting part of HATEOAS is the “engine.”


#### The engine of application state


If you’re on *Amazon.com* buying a book, you follow a series of links and fill out one or two forms before your credit card is charged. You transition through the ordering process by examining and interacting with the responses returned by each link you follow and each form you submit. The server guides you through the order process by embedding where you should go next within the HTML data it provides your browser.


This is very different from the way traditional distributed applications work. Older applications usually have a list of precanned services they know exist, and they interact with a central directory server to locate these services on the network. HATEOAS is a bit different because with each request returned from a server it tells you what new interactions you can do next, as well as where to go to transition the state of your applications.

For example, let’s say we wanted to get a list of products available on a web store. We do an HTTP GET on *http://example.com/webstore/products* and receive back:


```
<products>
  <product id="123">
      <name>headphones</name>
      <price>$16.99</price>
   </product>
   <product id="124">
      <name>USB Cable</name>
     <price>$5.99</price>
   </product>
...
</products>
```


This could be problematic if we had thousands of products to send back to our client. We might overload it, or the client might wait forever for the response to finish downloading. We could instead list only the first five products and provide a link to get the next set:


```
<products>
   <link rel="next" href="http://example.com/store/products?startIndex=5"/>
   <product id="123">
      <name>headphones</name>
      <price>$16.99</price>
   </product>
...
</products>
```

When first querying for a list of products, clients don’t have to know they’re getting back a list of only five products. The data format can tell them that they didn’t get a full set and that to get the **next** set, they need to follow a specific link. Following the **next** link could get them back a new document with additional links:

```
<products>
   <link rel="previous" href="http://example.com/store/products?startIndex=0"/>
   <link rel="next" href="http://example.com/webstore/products?startIndex=10"/>
   <product id="128">
      <name>stuff</name>
      <price>$16.99</price>
   </product>
...
</products>
```


In this case, there is the additional state transition of **previous** so that clients can browse an earlier part of the product list. The **next** and **previous** links seem a bit trivial, but imagine if we had other transition types like **payment**, **inventory**, or **sales**.

This sort of approach gives the server a lot of flexibility, as it can change where and how state transitions happen on the fly. It could provide new and interesting opportunities to surf to. In [Chapter 10](../chapter10/hateoas.md), we’ll dive into HATEOAS again.

