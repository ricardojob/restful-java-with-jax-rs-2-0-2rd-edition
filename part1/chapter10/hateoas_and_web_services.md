# HATEOAS and Web Services


How does HATEOAS relate to web services? When you’re applying HATEOAS to web services, the idea is to embed links within your XML or JSON documents. While this can be as easy as inserting a URL as the value of an element or attribute, most XML-based RESTful applications use syntax from the Atom Syndication Format as a means to implement HATEOAS.[^10] From the Atom RFC:

> *Atom is an XML-based document format that describes lists of related information known as “feeds.” Feeds are composed of a number of items, known as “entries,” each with an extensible set of attached metadata.*


Think of Atom as the next evolution of RSS. It is generally used to publish blog feeds on the Internet, but a few data structures within the format are particularly useful for web services, particularly Atom links.


### Atom Links


The Atom link XML type is a very simple yet standardized way of embedding links within your XML documents. Let’s look at an example:


```xml
<customers>
   <link rel="next"
         href="http://example.com/customers?start=2&size=2"
         type="application/xml"/>
   <customer id="123">
      <name>Bill Burke</name>
   </customer>
   <customer id="332">
      <name>Roy Fielding</name>
   </customer>
</customers>
```

The Atom link is just a simple XML element with a few specific attributes.

* The **rel** attribute 

    The **rel** attribute is used for link relationships. It is the logical, simple name used to reference the link. This attribute gives meaning to the URL you are linking to, much in the same way that text enclosed in an HTML **&lt;a&gt;** element gives meaning to the URL you can click in your browser. 

* The **href** attribute 

    This is the URL you can traverse in order to get new information or change the state of your application. 
    
* The **type** attribute 

    This is the exchanged media type of the resource the URL points to. 
    
* The **hreflang** attribute 

    Although not shown in the example, this attribute represents the language the data format is translated into. Some examples are French, English, German, and Spanish. 


When a client receives a document with embedded Atom links, it looks up the relationship it is interested in and invokes the URI embedded within the **href** link attribute.


### Advantages of Using HATEOAS with Web Services

It is pretty obvious why links and forms have done so much to make the Web so prevalent. With one browser, we have a window to a wide world of information and services. Search engines crawl the Internet and index websites, so all that data is at our fingertips. This is all possible because the Web is self-describing. We get a document and we know how to retrieve additional information by following links. We know how to purchase something from Amazon because the HTML form tells us how.


Machine-based clients are a little different, though. Other than browsers, there aren’t a lot of generic machine-based clients that know how to interpret self-describing documents. They can’t make decisions on the fly like humans can. They require programmers to tell them how to interpret data received from a service and how to transition to other states in the interaction between client and server. So, does that make HATEOAS useless to machine-based clients? Not at all. Let’s look at some of the advantages.


#### Location transparency


One feature that HATEOAS provides is location transparency. In a RESTful system that leverages HATEOAS, very few URIs are published to the outside world. Services and information are represented within links embedded in the data formats returned by accessing these top-level URIs. Clients need to know the logical link names to look for, but don’t have to know the actual network locations of the linked services.


For those of you who have written EJBs, this isn’t much different than using the Java Naming and Directory Interface (JNDI). Like a naming service, links provide a level of indirection so that underlying services can change their locations on the network without breaking client logic and code. HATEOAS has an additional advantage in that the top-level web service has control over which links are transferred.


#### Decoupling interaction details


Consider a request that gives us a list of customers in a customer database: **GET /customers**. If our database has thousands and thousands of entries, we do not want to return them all with one basic query. What we could do is define a view into our database using URI query parameters:


```
/customers?start={startIndex}&size={numberReturned}
```

The **start** query parameter identifies the starting index for our customer list. The **size** parameter specifies how many customers we want returned from the query.


This is all well and good, but what we’ve just done is increased the amount of predefined knowledge the client must have to interact with the service beyond a simple URI of **/customers**. Let’s say in the future, the server wanted to change how view sets are queried. For instance, maybe the customer database changes rather quickly and a start index isn’t enough information anymore to calculate the view. If the service changes the interface, we’ve broken older clients.


Instead of publishing this RESTful interface for viewing our database, what if, instead, we embedded this information within the returned document?


```xml
<customers>
   <link rel="next"
         href="http://example.com/customers?start=2&size=2"
         type="application/xml"/>
   <customer id="123">
      <name>Bill Burke</name>
   </customer>
   <customer id="332">
      <name>Roy Fielding</name>
   </customer>
</customers>
```


By embedding an Atom link within a document, we’ve given a logical name to a state transition. The state transition here is the next set of customers within the database. We are still requiring the client to have predefined knowledge about how to interact with the service, but the knowledge is much simpler. Instead of having to remember which URI query parameters to set, all that’s needed is to follow a specific named link. The client doesn’t have to do any bookkeeping of the interaction. It doesn’t have to remember which section of the database it is currently viewing.


Also, this returned XML is self-contained. What if we were to hand off this document to a third party? We would have to tell the third party that it is only a partial view of the database and specify the start index. Since we now have a link, this information is all a part of the document.


By embedding an Atom link, we’ve decoupled a specific interaction between the client and server. We’ve made our web service a little more transparent and change-resistant because we’ve simplified the predefined knowledge the client must have to interact with the service. Finally, the server has the power to guide the client through interactions by providing links.


#### Reduced state transition errors


Links are not used only as a mechanism to aggregate and navigate information. They can also be used to change the state of a resource. Consider an order in an ecommerce website obtained by traversing the URI **/orders/333**:


```xml
<order id="333">
  <customer id="123">...</customer>
  <amount>$99.99</amount>
  <order-entries>
    ...
  </order-entries>
</order>
```


Let’s say a customer called up and wanted to cancel her order. We could simply do an HTTP DELETE on **/orders/333**. This isn’t always the best approach, as we usually want to retain the order for data warehousing purposes. So, instead, we might PUT a new representation of the order with a **cancelled** element set to **true**:


```
PUT /orders/333 HTTP/1.1
Content-Type: application/xml

<order id="333">
  <customer id="123">...</customer>
  <amount>$99.99</amount>
  <cancelled>true</cancelled>
  <order-entries>
    ...
  </order-entries>
</order>
```


But what happens if the order can’t be cancelled? We may be at a certain state in our order process where such an action is not allowed. For example, if the order has already been shipped, it cannot be cancelled. In this case, there really isn’t a good HTTP status code to send back that represents the problem. A better approach would be to embed a **cancel** link:


```xml
<order id="333">
  <customer id="123">...</customer>
  <amount>$99.99</amount>
  <cancelled>false</cancelled>
  <link rel="cancel"
        href="http://example.com/orders/333/cancelled"/>
  <order-entries>
    ...
  </order-entries>
</order>
```

The client would do a **GET /orders/333** and get the XML document representing the order. If the document contains the **cancel** link, the client is allowed to change the order status to “cancelled” by doing an empty POST or PUT to the URI referenced in the link. If the document doesn’t contain the link, the client knows that this operation is not possible. This allows the web service to control how the client is able to interact with it in real time.



#### W3C standardized relationships


An interesting thing that is happening in the REST community is an effort to define, register, and standardize a common set of link relationship names and their associated behaviors.[^11] Some examples are given in Table 10-1.


Table 10-1. W3C standard relationship names

| Relationship | Description |
| -- | :-- |
| previous | A URI that refers to the immediately preceding document in a series of documents. |
| next | A URI that refers to the immediately following document in a series of documents. |
| edit | A URI that can be retrieved, updated, and deleted. |
| payment | A URI where payment is accepted. It is meant as a general way to facilitate acts of payment. |


This is not an exhaustive list, but hopefully you get the general idea where this registry is headed. Registered relationships can go a long way to help make data formats even more self-describing and intuitive to work with.


### Link Headers Versus Atom Links


While Atom links have become very popular for publishing links in RESTful systems, there is an alternative. Instead of embedding a link directly in your document, you can instead use Link response headers.[^12] This is best explained with an example.


Consider the order cancellation example described in the previous section. An Atom link is used to specify whether or not the cancelling of an order is allowed and which URL to use to do a POST that will cancel the order. Instead of using an Atom link embedded within the order XML document, let’s use a Link header. So, if a user submits **GET /orders/333**, he will get back the following HTTP response:


```
HTTP/1.1 200 OK
Content-Type: application/xml
Link: <http://example.com/orders/333/cancelled>; rel=cancel

<order id="333">
  ...
</order>
```

The **Link** header has all the same characteristics as an Atom link. The URI is enclosed within &lt;&gt; followed by one or more attributes delimited by semicolons. The **rel** attribute is required and means the same thing as the corresponding Atom attribute of the same name. This part isn’t shown in the example, but you may also specify a media type via the **type** attribute.


Personally, I really like **Link** headers as an alternative to embedding Atom links. Many times, I find that my client isn’t interested in the resource representation and is only interested in the link relations. You shouldn’t have to parse a whole XML or JSON document just to find the URL you’re interested in invoking on. Another nice thing is that instead of doing a GET invocation, you can do a HEAD invocation and avoid getting the XML document entirely. In general, I like to use Atom links for data aggregation and **Link** headers for everything else.


---
[^10] For more information, see [the w3 website](http://www.w3.org/2005/atom).

[^11] For more information, see http://www.iana.org/assignments/link-relations/link-relations.xhtml.

[^12] For more information, see [9 Method Definitions](http://tools.ietf.org/html/rfc5988).