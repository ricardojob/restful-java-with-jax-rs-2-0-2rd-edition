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
