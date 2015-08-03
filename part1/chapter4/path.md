# @Path


There’s more to the **@javax.ws.rs.Path** annotation than what we saw in our simple example in [Chapter 3](../chapter3/your_first_jax_rs_service.md). **@Path** can have complex matching expressions so that you can be more specific about what requests get bound to which incoming URIs. **@Path** can also be used on a Java method as sort of an object factory for subresources of your application. We’ll examine both in this section.


### Binding URIs


The **@javax.ws.rs.Path** annotation in JAX-RS is used to define a URI matching pattern for incoming HTTP requests. It can be placed upon a class or on one or more Java methods. For a Java class to be eligible to receive any HTTP requests, the class must be annotated with at least the **@Path("/")** expression. These types of classes are called JAX-RS *root resources*.


The value of the **@Path** annotation is an expression that denotes a relative URI to the context root of your JAX-RS application. For example, if you are deploying into a WAR archive of a servlet container, that WAR will have a base URI that browsers and remote clients use to access it. **@Path** expressions are relative to this URI.


To receive a request, a Java method must have at least an HTTP method annotation like **@javax.ws.rs.GET** applied to it. This method is not required to have an **@Path** annotation on it, though. For example:



```Java
@Path("/orders")
public class OrderResource {
   @GET
   public String getAllOrders() {
       ...
   }
}
```


An HTTP request of **GET /orders** would dispatch to the **getAllOrders()** method.


You can also apply **@Path** to your Java method. If you do this, the URI matching pattern is a concatenation of the class’s **@Path** expression and that of the method’s. For example:

```Java
@Path("/orders")
public class OrderResource {

   @GET
   @Path("unpaid")
   public String getUnpaidOrders() {
      ...
   }
}
```

So, the URI pattern for **getUnpaidOrders()** would be the relative URI **/orders/unpaid**.


### @Path Expressions


The value of the **@Path** annotation is usually a simple string, but you can also define more complex expressions to satisfy your URI matching needs.


#### Template parameters


In [Chapter 3](../chapter3/your_first_jax_rs_service.md), we wrote a customer access service that allowed us to query for a specific customer using a wildcard URI pattern:


```Java
@Path("/customers")
public class CustomerResource {

   @GET
   @Path("{id}")
   public String getCustomer(@PathParam("id") int id) {
      ...
   }
}
```

These template parameters can be embedded anywhere within an **@Path** declaration. For example:


```Java
@Path("/")
public class CustomerResource {

   @GET
   @Path("customers/{firstname}-{lastname}")
   public String getCustomer(@PathParam("firstname") String first,
                             @PathParam("lastname") String last) {
      ...
   }
}
```


In our example, the URI is constructed with a customer’s first name, followed by a hyphen, ending with the customer’s last name. So, the request **GET /customers/333** would no longer match to **getCustomer()**, but a **GET/customers/bill-burke** request would.


#### Regular expressions


**@Path** expressions are not limited to simple wildcard matching expressions. For example, our **getCustomer()** method takes an integer parameter. We can change our **@Path** value to match only digits:


```Java
@Path("/customers")
public class CustomerResource {

   @GET
   @Path("{id : \\d+}")
   public String getCustomer(@PathParam("id") int id) {
      ...
   }
}
```


Regular expressions are not limited in matching one segment of a URI. For example:


```Java
@Path("/customers")
public class CustomerResource {

   @GET
   @Path("{id : .+}")
   public String getCustomer(@PathParam("id") String id) {
      ...
   }

   @GET
   @Path("{id : .+}/address")
   public String getAddress(@PathParam("id") String id) {
      ...
   }

}
```


We’ve changed **getCustomer()**’s **@Path** expression to **{id : .+}**. The **.+** is a regular expression that will match any stream of characters after **/customers**. So, the **GET /customers/bill/burke** request would be routed to **getCustomer()**.


The **getAddress()** method has a more specific expression. It will map any stream of characters after **/customers** that ends with **/address**. So, the **GET /customers/bill/burke/address** request would be routed to the **getAddress()** method.


#### Precedence rules


You may have noticed that, together, the **@Path** expressions for **getCustomer()** and **getAddress()** are ambiguous. A **GET /customers/bill/burke/address** request could match either **getCustomer()** or **getAddress()**, depending on which expression was matched first by the JAX-RS provider. The JAX-RS specification has defined strict sorting and precedence rules for matching URI expressions and is based on a *most specific match wins* algorithm. The JAX-RS provider gathers up the set of deployed URI expressions and sorts them based on the following logic:

1. The primary key of the sort is the number of literal characters in the full URI matching pattern. The sort is in descending order. In our ambiguous example, **getCustomer()**’s pattern has 11 literal characters: **/customers/**. The **getAddress()** method’s pattern has 18 literal characters: **/customers/** plus **address**. Therefore, the JAX-RS provider will try to match **getAddress()**’s pattern before **getCustomer()**. 
2. The secondary key of the sort is the number of template expressions embedded within the pattern—that is, **{id}** or **{id : .+}**. This sort is in descending order. 
3. The tertiary key of the sort is the number of nondefault template expressions. A default template expression is one that does not define a regular expression—that is, **{id}**. 


Let’s look at a list of sorted URI matching expressions and explain why one would match over another:


```
1 /customers/{id}/{name}/address
2 /customers/{id : .+}/address
3 /customers/{id}/address
4 /customers/{id : .+}
```


Expressions 1–3 come first because they all have more literal characters than expression 4. Although expressions 1–3 all have the same number of literal characters, expression 1 comes first because sorting rule #2 is triggered. It has more template expressions than either pattern 2 or 3. Expressions 2 and 3 have the same number of literal characters and same number of template expressions. Expression 2 is sorted ahead of 3 because it triggers sorting rule #3; it has a template pattern that is a regular expression.


These sorting rules are not perfect. It is still possible to have ambiguities, but the rules cover 90% of use cases. If your application has URI matching ambiguities, your application design is probably too complicated and you need to revisit and refactor your URI scheme.


#### Encoding

The URI specification only allows certain characters within a URI string. It also reserves certain characters for its own specific use. In other words, you cannot use these characters as part of your URI segments. This is the set of allowable and reserved characters:

* The US-ASCII alphabetic characters a–z and A–Z are allowable. 
* The decimal digit characters 0–9 are allowable. 
* All these other characters are allowable: **_-!.~'()\***. 
* These characters are allowed but are reserved for URI syntax: **,;:$&+=?/\\\[\]@**. 


All other characters must be encoded using the “%” character followed by a two-digit hexadecimal number. This hexadecimal number corresponds to the equivalent hexadecimal character in the ASCII table. So, the string **bill&burke** would be encoded as **bill_burke**.


When creating **@Path** expressions, you may encode its string, but you do not have to. If a character in your **@Path** pattern is an illegal character, the JAX-RS provider will automatically encode the pattern before trying to match and dispatch incoming HTTP requests. If you do have an encoding within your **@Path** expression, the JAX-RS provider will leave it alone and treat it as an encoding when doing its request dispatching. For example:


```Java
@Path("/customers"
public class CustomerResource {

   @GET
   @Path("roy&fielding")
   public String getOurBestCustomer() {
      ...
   }
}
```

The **@Path** expression for **getOurBestCustomer()** would match incoming requests like **GET /customers/roy%26fielding**.


### Matrix Parameters


One part of the URI specification that we have not touched on yet is matrix parameters. Matrix parameters are name-value pairs embedded within the path of a URI string. For example:

```
http://example.cars.com/mercedes/e55;
color=black/2006
```

They come after a URI segment and are delimited by the “;” character. The matrix parameter in this example comes after the URI segment **e55**. Its name is **color** and its value is **black**. Matrix parameters are different than query parameters, as they represent attributes of certain segments of the URI and are used for identification purposes. Think of them as adjectives. Query parameters, on the other hand, always come at the end of the URI and always pertain to the full resource you are referencing.



Matrix parameters are ignored when matching incoming requests to JAX-RS resource methods. It is actually illegal to specify matrix parameters within an **@Path** expression. For example:


```Java
@Path("/mercedes")
public class MercedesService {

   @GET
   @Path("/e55/{year}")
   @Produces("image/jpeg")
   public Jpeg getE55Picture(@PathParam("year") String year) {
     ...
   }
```


If we queried our JAX-RS service with **GET /mercedes/e55;color=black/2006**, the **getE55Picture()** method would match the incoming request and would be invoked. Matrix parameters are not considered part of the matching process because they are usually variable attributes of the request. We’ll see in [Chapter 5](../chapter5/jax_rs_injection.md) how to access matrix parameter information within our JAX-RS resource methods.


















