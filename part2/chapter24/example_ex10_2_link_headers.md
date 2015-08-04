# Example ex10_2: Link Headers


There are two educational goals I want to get across with this example. The first is the use of **Link** headers within a RESTful application. The second is that if your services provide the appropriate links, you only need one published URL to navigate through your system. When you look at the client code for this example, you’ll see that only one URL is hardcoded to start the whole process of the example.



To illustrate these techniques, a few more additional JAX-RS services were built beyond the simple customer database example that has been repeated so many times throughout this book. [Chapter 2](../../part1/chapter2/designing_restful_services.md) discussed the design of an ecommerce application. This chapter starts the process of implementing this application by introducing an order-entry RESTful service.



### The Server Code


The **Order** and **LineItem** classes are added to the JAXB domain model. They are used to marshal the XML that represents order entries in the system. They are not that interesting, so I’m not going to get into much detail here.


#### OrderResource


The **OrderResource** class is used to create, post, and cancel orders in our ecommerce system. The purge operation is also available to destroy any leftover order entries that have been cancelled but not removed from the order entry database. Let’s look:


```Java:src/main/java/com/restfully/shop/services/OrderResource.java
@Path("/orders")
public class OrderResource
{
   private Map<Integer, Order> orderDB =
                                     new Hashtable<Integer, Order>();
   private AtomicInteger idCounter = new AtomicInteger();

   @POST
   @Consumes("application/xml")
   public Response createOrder(Order order, @Context UriInfo uriInfo)
   {
      order.setId(idCounter.incrementAndGet());
      orderDB.put(order.getId(), order);
      System.out.println("Created order " + order.getId());
      UriBuilder builder = uriInfo.getAbsolutePathBuilder();
      builder.path(Integer.toString(order.getId()));
      return Response.created(builder.build()).build();

   }
```


The **createOrder()** method handles **POST /orders** requests. It generates new **Order** IDs and adds the posted **Order** instance into the order database (the map). The **UriInfo.getAbsolutePathBuilder()** method generates the URL used to initialize the **Location** header returned by the **Response.created()** method. You’ll see later that the client uses this URL to further manipulate the created order.


```Java
   @GET
   @Path("{id}")
   @Produces("application/xml")
   public Response getOrder(@PathParam("id") int id,
                              @Context UriInfo uriInfo)
   {
      Order order = orderDB.get(id);
      if (order == null)
      {
        throw new WebApplicationException(Response.Status.NOT_FOUND);
      }
      Response.ResponseBuilder builder = Response.ok(order);
      if (!order.isCancelled()) addCancelHeader(uriInfo, builder);
      return builder.build();
   }
```


The **getOrder()** method processes **GET /orders/{id}** requests and retrieves individual orders from the database (the map). If the order has not been cancelled already, a **cancel** **Link** header is added to the Response so the client knows if an order can be cancelled and which URL to post a cancel request to:


```Java
   protected void addCancelHeader(UriInfo uriInfo,
                                    Response.ResponseBuilder builder)
   {
      UriBuilder absolute = uriInfo.getAbsolutePathBuilder();
      URI cancelUrl = absolute.clone().path("cancel").build();
      builder.links(Link.fromUri(cancelUrl).rel("cancel").build());
   }
```


The **addCancelHeader()** method creates a **Link** object for the **cancel** relationship using a URL generated from **UriInfo.getAbsolutePathBuilder()**.



```Java
   @HEAD
   @Path("{id}")
   @Produces("application/xml")
   public Response getOrderHeaders(@PathParam("id") int id,
                                    @Context UriInfo uriInfo)
   {
      Order order = orderDB.get(id);
      if (order == null)
      {
        throw new WebApplicationException(Response.Status.NOT_FOUND);
      }
      Response.ResponseBuilder builder = Response.ok();
      builder.type("application/xml");
      if (!order.isCancelled()) addCancelHeader(uriInfo, builder);
      return builder.build();
   }
```


The **getOrderHeaders()** method processes HTTP **HEAD /orders/{id}** requests. This is a convenience operation for HTTP clients that want the link relationships published by the resource but don’t want to have to parse an XML document to get this information. Here, the **getOrderHeaders()** method returns the **cancel** **Link** header with an empty response body:


```Java
   @POST
   @Path("{id}/cancel")
   public void cancelOrder(@PathParam("id") int id)
   {
      Order order = orderDB.get(id);
      if (order == null)
      {
         throw new WebApplicationException(Response.Status.NOT_FOUND);
      }
      order.setCancelled(true);
   }
```


Users can cancel an order by posting an empty message to **/orders/{id}/cancel**. The **cancelOrder()** method handles these requests and simply looks up the **Order** in the database and sets its state to cancelled.


```Java
   @GET
   @Produces("application/xml")
   @Formatted
   public Response getOrders(@QueryParam("start") int start,
                     @QueryParam("size") @DefaultValue("2") int size,
                    @Context UriInfo uriInfo)
   {
...
      Orders orders = new Orders();
      orders.setOrders(list);
      orders.setLinks(links);
      Response.ResponseBuilder responseBuilder = Response.ok(orders);
      addPurgeLinkHeader(uriInfo, responseBuilder);
      return responseBuilder.build();
   }
```


The **getOrders()** method is similar to the **CustomerResource.getCustomers()** method discussed in the *ex10_1* example, so I won’t go into a lot of details. One thing it does differently, though, is to publish a **purge** link relationship through a **Link** header. Posting to this link allows clients to purge the order entry database of any lingering cancelled orders:


```Java
   protected void addPurgeLinkHeader(UriInfo uriInfo,
                                     Response.ResponseBuilder builder)
   {
      UriBuilder absolute = uriInfo.getAbsolutePathBuilder();
      URI purgeUri = absolute.clone().path("purge").build();
      builder.links(Link.fromUri(purgeUri).rel("purge").build());
   }
```


The **addPurgeLinkHeader()** method creates a **Link** object for the **purge** relationship using a URL generated from **UriInfo.getAbsolutePathBuilder()**.



```Java
   @HEAD
   @Produces("application/xml")
   public Response getOrdersHeaders(@QueryParam("start") int start,
                     @QueryParam("size") @DefaultValue("2") int size,
                    @Context UriInfo uriInfo)
   {
      Response.ResponseBuilder builder = Response.ok();
      builder.type("application/xml");
      addPurgeLinkHeader(uriInfo, builder);
      return builder.build();
   }
```


The **getOrdersHeaders()** method is another convenience method for clients that are interested only in the link relationships provided by the resource:


```Java
   @POST
   @Path("purge")
   public void purgeOrders()
   {
      synchronized (orderDB)
      {
         List<Order> orders = new ArrayList<Order>();
         orders.addAll(orderDB.values());
         for (Order order : orders)
         {
            if (order.isCancelled())
            {
               orderDB.remove(order.getId());
            }
         }
      }
   }
```


Finally, the **purgeOrders()** method implements the purging of cancelled orders.


#### StoreResource


One of the things I want to illustrate with this example is that a client needs to be aware of only one URL to navigate through the entire system. The **StoreResource** class is the base URL of the system and publishes **Link** headers to the relevant services of the application:



```Java:src/main/java/com/restfully/shop/services/StoreResource.java
@Path("/shop")
public class StoreResource
{
   @HEAD
   public Response head(@Context UriInfo uriInfo)
   {
      UriBuilder absolute = uriInfo.getBaseUriBuilder();
      URI customerUrl = absolute.clone().path(CustomerResource.class).build();
      URI orderUrl = absolute.clone().path(OrderResource.class).build();

      Response.ResponseBuilder builder = Response.ok();
      Link customers = Link.fromUri(customerUrl)
                           .rel("customers")
                           .type("application/xml").build();
      Link orders = Link.fromUri(orderUrl)
                        .rel("orders")
                        .type("application/xml").build();
      builder.links(customers, orders);
      return builder.build();
   }
}
```


This class accepts HTTP **HEAD /shop** requests and publishes the **customers** and **orders** link relationships. These links point to the services represented by the **CustomerResource** and **OrderResource** classes.



### The Client Code


The client code creates a new customer and order. It then cancels the order, purges it, and, finally, relists the order entry database. All URLs are accessed via **Link** headers or Atom links:



```Java
public class OrderResourceTest
{
   @Test
   public void testCreateCancelPurge() throws Exception
   {
      String base = "http://localhost:8080/services/shop";
      Response response = client.target(base).request().head();

      Link customers = response.getLink("customers");
      Link orders = response.getLink("orders");
      response.close();
```


The **testCreateCancelPurge()** method starts off by doing a HEAD request to **/shop** to obtain the service links provided by our application. The **Response.getLink()** method allows you to query for a **Link** header sent back with the HTTP response.


```Java
      System.out.println("** Create a customer through this URL: "
                            + customers.getHref());

      Customer customer = new Customer();
      customer.setFirstName("Bill");
      customer.setLastName("Burke");
      customer.setStreet("10 Somewhere Street");
      customer.setCity("Westford");
      customer.setState("MA");
      customer.setZip("01711");
      customer.setCountry("USA");

      response = client.target(customers).request().post(Entity.xml(customer));
      Assert.assertEquals(201, response.getStatus());
      response.close();
```


We create a customer in the customer database by POSTing an XML representation to the URL referenced in the **customers** link relationship. This relationship is retrieved from our initial HEAD request to **/shop**.


```Java
      Order order = new Order();
      order.setTotal("$199.99");
      order.setCustomer(customer);
      order.setDate(new Date().toString());
      LineItem item = new LineItem();
      item.setCost("$199.99");
      item.setProduct("iPhone");
      order.setLineItems(new ArrayList<LineItem>());
      order.getLineItems().add(item);

      System.out.println();
      System.out.println("** Create an order through this URL: "
                                           + orders.getUri().toString());
      response = client.target(orders).request().post(Entity.xml(order));
      Assert.assertEquals(201, response.getStatus());
      URI createdOrderUrl = response.getLocation();
      response.close();
```


Next, we create an order entry by posting to the **orders** link relationship. The URL of the created order is extracted from the returned **Location** header. We will need this later when we want to cancel this order:



```Java
      System.out.println();
      System.out.println("** New list of orders");
      response = client.target(orders).request().get();
      String orderList = response.readEntity(String.class);
      System.out.println(orderList);
      Link purge = response.getLink("purge");
      response.close();
```


A **GET /orders** request is initiated to show all the orders posted to the system. We extract the **purge** link returned by this invocation so it can be used later when the client wants to purge cancelled orders:


```Java
      response = client.target(createdOrderUrl).request().head();
      Link cancel = response.getLink("cancel");
      response.close();
```


Next, the client cancels the order that was created earlier. A HEAD request is made to the created order’s URL to obtain the **cancel** link relationship:


```Java
      if (cancel != null)
      {
         System.out.println("** Cancelling the order at URL: "
                                    + cancel.getUri().toString());
         response = client.target(cancel).request().post(null);
         Assert.assertEquals(204, response.getStatus());
         response.close();
      }
```


If there is a **cancel** link relationship, the client posts an empty message to this URL to cancel the order:


```Java
      System.out.println();
      System.out.println("** New list of orders after cancel: ");
      orderList = client.target(orders).request().get(String.class);
      System.out.println(orderList);
```


The client does another **GET /orders** to show that the state of our created order was set to cancelled:


```Java
      System.out.println();
      System.out.println("** Purge cancelled orders at URL: "
                                      + purge.getUri().toString());
      response = client.target(purge).request().post(null);
      Assert.assertEquals(204, response.getStatus());
      response.close();

      System.out.println();
      System.out.println("** New list of orders after purge: ");
      orderList = client.target(orders).request().get(String.class);
      System.out.println(orderList);
   }
```


Finally, by posting an empty message to the **purge** link, the client cleans the order entry database of any cancelled orders.



### Build and Run the Example Program


Perform the following steps:

1. Open a command prompt or shell terminal and change to the *ex10_2* directory of the workbook example code. 
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md). 
3. Perform the build and run the example by typing **maven install**.