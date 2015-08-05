# Assigning HTTP Methods


The final thing we have to do is decide which HTTP methods will be exposed for each of our resources and what these methods will do. It is crucial that we do not assign functionality to an HTTP method that supersedes the specification-defined boundaries of that method. For example, an HTTP GET on a particular resource should be read-only. It should not change the state of the resource it is invoking on. Intermediate services like a proxy-cache, a CDN (Akamai), or your browser rely on you to follow the semantics of HTTP strictly so that they can perform built-in tasks like caching effectively. If you do not follow the definition of each HTTP method strictly, clients and administration tools cannot make assumptions about your services, and your system becomes more complex.


Let’s walk through each method of our object model to determine which URIs and HTTP methods are used to represent them.


### Browsing All Orders, Customers, or Products


The **Order**, **Customer**, and **Product** objects in our object model are all very similar in how they are accessed and manipulated. One thing our remote clients will want to do is to browse all the **Orders**, **Customers**, or **Products** in the system. These URIs represent these objects as a group:

```
/orders
/products
/customers
```


To get a list of **Orders**, **Products**, or **Customers**, the remote client will call an HTTP GET on the URI of the object group it is interested in. An example request would look like the following:

```
GET /products HTTP/1.1
```

Our service will respond with a data format that represents all **Orders**, **Products**, or **Customers** within our system. Here’s what a response would look like:

```xml
HTTP/1.1 200 OK
Content-Type: application/xml

<products>
   <product id="111">
      <link rel="self" href="http://example.com/products/111"/>
      <name>iPhone</name>
      <cost>$199.99</cost>
   </product>
   <product id="222">
      <link rel="self" href="http://example.com/products/222"/>
      <name>Macbook</name>
      <cost>$1599.99</cost>
   </product>
...
</products>
```

One problem with this bulk operation is that we may have thousands of **Orders**, **Customers**, or **Products** in our system and we may overload our client and hurt our response times. To mitigate this problem, we will allow the client to specify query parameters on the URI to limit the size of the dataset returned:

```
GET /orders?startIndex=0&size=5 HTTP/1.1
GET /products?startIndex=0&size=5 HTTP/1.1
GET /customers?startIndex=0&size=5 HTTP/1.1
```

Here we have defined two query parameters: **startIndex** and **size**. The **startIndex** parameter represents where in our large list of **Orders**, **Products**, or **Customers** we want to start sending objects from. It is a numeric index into the object group being queried. The size parameter specifies how many of those objects in the list we want to return. These parameters will be optional. The client does not have to specify them in its URI when crafting its request to the server.


### Obtaining Individual Orders, Customers, or Products

I mentioned in the previous section that we would use a URI pattern to obtain individual **Orders**, **Customers**, or **Products**:

```
/orders/{id}
/products/{id}
/customers/{id}
```

We will use the HTTP GET method to retrieve individual objects in our system. Each GET invocation will return a data format that represents the object being obtained:

```
GET /orders/233 HTTP/1.1
```

For this request, the client is interested in getting a representation of the **Order** with an **order id** of **233**. GET requests for **Products** and **Customers** would work the same. The HTTP response message would look something like this:

```xml
HTTP/1.1 200 OK
Content-Type: application/xml

<order id="233">...</order>
```

The response code is 200, “OK,” indicating that the request was successful. The **Content-Type** header specifies the format of our message body as XML, and finally we have the actual representation of the **Order**.



### Creating an Order, Customer, or Product

There are two possible ways in which a client could create an **Order**, **Customer**, or **Product** within our order entry system: by using either the HTTP PUT or POST method. Let’s look at both ways.


#### Creating with PUT

The HTTP definition of PUT states that it can be used to create or update a resource on the server. To create an **Order**, **Customer**, or **Product** with PUT, the client simply sends a representation of the new object it is creating to the exact URI location that represents the object:

```
PUT /orders/233 HTTP/1.1
PUT /customers/112 HTTP/1.1
PUT /products/664 HTTP/1.1
```

PUT is required by the specification to send a response code of 201, “Created,” if a new resource was created on the server as a result of the request.

The HTTP specification also states that PUT is idempotent. Our PUT is idempotent, because no matter how many times we tell the server to “create” our **Order**, the same bits are stored at the **/orders/233** location. Sometimes a PUT request will fail and the client won’t know if the request was delivered and processed at the server. Idempotency guarantees that it’s OK for the client to retransmit the PUT operation and not worry about any adverse side effects.


The disadvantage of using PUT to create resources is that the client has to provide the unique ID that represents the object it is creating. While it usually possible for the client to generate this unique ID, most application designers prefer that their servers (usually through their databases) create this ID. In our hypothetical order entry system, we want our server to control the generation of resource IDs. So what do we do? We can switch to using POST instead of PUT.


#### Creating with POST

Creating an **Order**, **Customer**, or **Product** using the POST method is a little more complex than using PUT. To create an **Order**, **Customer**, or **Product** with POST, the client sends a representation of the new object it is creating to the parent URI of its representation, leaving out the numeric target ID. For example:

```xml
POST /orders HTTP/1.1
Content-Type: application/xml

<order>
   <total>$199.02</total>
   <date>December 22, 2008 06:56</date>
...
</order>
```

The service receives the POST message, processes the XML, and creates a new order in the database using a database-generated unique ID. While this approach works perfectly fine, we’ve left our client in a quandary. What if the client wants to edit, update, or cancel the order it just posted? What is the ID of the new order? What URI can we use to interact with the new resource? To resolve this issue, we will add a bit of information to the HTTP response message. The client would receive a message something like this:

```xml
HTTP/1.1 201 Created
Content-Type: application/xml
Location: http://example.com/orders/233

<order id="233">
   <link rel="self" href="http://example.com/orders/233"/>
   <total>$199.02</total>
   <date>December 22, 2008 06:56</date>
...
</order>
```


HTTP requires that if POST creates a new resource, it respond with a code of 201, “Created” (just like PUT). The Location header in the response message provides a URI to the client so it knows where to further interact with the **Order** that was created (i.e., if the client wanted to update the **Order**). It is optional whether the server sends the representation of the newly created **Order** with the response. Here, we send back an XML representation of the **Order** that was just created with the ID attribute set to the one generated by our database as well as a **link** element.


> **Note**  I didn’t pull the Location header out of thin air. The beauty of this approach is that it is defined within the HTTP specification. That’s an important part of REST—to follow the predefined behavior within the specification of the protocol you are using. Because of this, most systems are self-documenting, as the distributed interactions are already mostly defined by the HTTP specification.


### Updating an Order, Customer, or Product


We will model updating an **Order**, **Customer**, or **Product** using the HTTP PUT method. The client PUTs a new representation of the object it is updating to the exact URI location that represents the object. For example, let’s say we wanted to change the price of a product from $199.99 to $149.99. Here’s what the request would look like:

```xml
PUT /orders/233 HTTP/1.1
Content-Type: application/xml

<product id="111">
   <name>iPhone</name>
   <cost>$149.99
</cost>
</product>
```

As I stated earlier in this chapter, PUT is great because it is idempotent. No matter how many times we transmit this PUT request, the underlying **Product** will still have the same final state.


When a resource is updated with PUT, the HTTP specification requires that you send a response code of 200, “OK,” and a response message body or a response code of 204, “No Content,” without any response body. In our system, we will send a status of 204 and no response message.


> **Note**  We could use POST to update an individual Order, but then the client would have to assume the update was nonidempotent and we would have to take duplicate message processing into account.


### Removing an Order, Customer, or Product

We will model deleting an **Order**, **Customer**, or **Product** using the HTTP DELETE method. The client simply invokes the DELETE method on the exact URI that represents the object we want to remove. Removing an object will wipe its existence from the system.

When a resource is removed with DELETE, the HTTP specification requires that you send a response code of 200, “OK,” and a response message body or a response code of 204, “No Content,” without any response body. In our application, we will send a status of 204 and no response message.


### Cancelling an Order

So far, the operations of our object model have fit quite nicely into corresponding HTTP methods. We’re using GET for reading, PUT for updating, POST for creating, and DELETE for removing. We do have an operation in our object model that doesn’t fit so nicely. In our system, **Orders** can be cancelled as well as removed. While removing an object wipes it clean from our databases, cancelling only changes the state of the Order and retains it within the system. How should we model such an operation?


#### Overloading the meaning of DELETE

Cancelling an **Order** is very similar to removing it. Since we are already modeling remove with the HTTP DELETE method, one thing we could do is add an extra query parameter to the request:

```
DELETE /orders/233?cancel=true
```

Here, the **cancel** query parameter would tell our service that we don’t really want to remove the **Order**, but cancel it. In other words, we are overloading the meaning of DELETE.


While I’m not going to tell you not to do this, I will tell you that you shouldn’t do it. It is not good RESTful design. In this case, you are changing the meaning of the uniform interface. Using a query parameter in this way is actually creating a mini-RPC mechanism. HTTP specifically states that DELETE is used to delete a resource from the server, not cancel it.


#### States versus operations

When modeling a RESTful interface for the operations of your object model, you should ask yourself a simple question: is the operation a state of the resource? If you answer yes to this question, the operation should be modeled within the data format.

Cancelling an **Order** is a perfect example of this. The key with cancelling is that it is a specific state of an **Order**. When a client follows a particular URI that links to a specific **Order**, the client will want to know whether the **Order** was cancelled or not. Information about the cancellation needs to be in the data format of the **Order**. So let’s add a cancelled element to our **Order** data format:

```xml
<order id="233">
   <link rel="self" href="http://example.com/orders/233"/>
   <total>$199.02</total>
   <date>December 22, 2008 06:56</date>
   <cancelled>false</cancelled>
...
</order>
```


Since the state of being cancelled is modeled in the data format, we can now use our already defined mechanism of updating an **Order** to model the cancel operation. For example, we could PUT this message to our service:

```xml
PUT /orders/233 HTTP/1.1
Content-Type: application/xml

<order id="233">
   <total>$199.02</total>
   <date>December 22, 2008 06:56</date>
   <cancelled>true</cancelled>
...
</order>
```

In this example, we PUT a new representation of our order with the cancelled element set to true. By doing this, we’ve changed the state of our order from viable to cancelled.

This pattern of modeling an operation as the state of the resource doesn’t always fit, though. What if we expanded on our cancel example by saying that we wanted a way to clean up all cancelled orders? In other words, we want to purge all cancelled orders from our database. We can’t really model purging the same way we did cancel. While purge does change the state of our application, it is not in and of itself a state of the application.

To solve this problem, we model this operation as a subresource of /orders and we trigger a purging by doing a POST on that resource. For example:

```
POST /orders/purge HTTP/1.1
```

An interesting side effect of this is that because purge is now a URI, we can evolve its interface over time. For example, maybe **GET /orders/purge** returns a document that states the last time a purge was executed and which orders were deleted. What if we wanted to add some criteria for purging as well? Form parameters could be passed stating that we only want to purge orders older than a certain date. In doing this, we’re giving ourselves a lot of flexibility as well as honoring the uniform interface contract of REST.

