# Server Asynchronous Response Processing



For a typical HTTP server, when a request comes in, one thread is dedicated to the processing of that request and its HTTP response to the client. This is fine for the vast majority of HTTP traffic both on the Internet and on your company’s internal networks. Most HTTP requests are short-lived, so a few hundred threads can easily handle a few thousand concurrent users and have relatively decent response times.



The nature of HTTP traffic started to change somewhat as JavaScript clients started to become more prevalent. One problem that popped up often was the need for the server to push events to the client. A typical example is a stock quote application where you need to update a string of clients with the latest stock price. These clients would make an HTTP GET or POST request and just block indefinitely until the server was ready to send back a response. This resulted in a large amount of open, long-running requests that were doing nothing other than idling. Not only were there a lot of open, idle sockets, but there were also a lot of dedicated threads doing nothing at all. Most HTTP servers were designed for short-lived requests with the assumption that one thread could process requests from multiple concurrent users. When you have a very large number of threads, you start to consume a lot of operating system resources. Each thread consumes memory, and context switching between threads starts to get quite expensive when the OS has to deal with a large number of threads. It became really hard to scale these types of server-push applications since the Servlet API, and by association JAX-RS, was a “one thread per connection” model.



In 2009, the Servlet 3.0 specification introduced asynchronous HTTP. With the Servlet 3.0 API, you can suspend the current server-side request and have a separate thread, other than the calling thread, handle sending back a response to the client. For a server-push app, you could then have a small handful of threads manage sending responses back to polling clients and avoid all the overhead of the “one thread per connection” model. JAX-RS 2.0 introduced a similar API that we’ll discuss in this section.


> **Note**  #####Note
> Server-side async response processing is only meant for a specific small subset of applications. Asynchronous doesn’t necessarily mean automatic scalability. For the typical web app, using server asynchronous response processing will only complicate your code and make it harder to maintain. It may even hurt performance.



### AsyncResponse API


To use server-side async response processing, you interact with the **AsyncResponse** interface:


```Java
package javax.ws.rs.container;

public interface AsyncResponse {
   boolean resume(Object response);
   boolean resume(Throwable response);

   ...
}
```


You get access to an **AsyncResponse** instance by injecting it into a JAX-RS method using the **@Suspended** annotation:



```Java
import javax.ws.rs.container.AsyncResponse;
import javax.ws.rs.container.Suspended;

@Path("/orders")
public class OrderResource {

   @POST
   @Consumes("application/json")
   public void submit(final Order order,
                      final @Suspended AsyncResponse response) {
   }
}
```



Here we have our very familiar **OrderResource**. Order submission has been turned into an asynchronous operation. When you inject an instance of **AsyncResponse** using the **@Suspended** annotation, the HTTP request becomes suspended from the current thread of execution. In this particular example, the **OrderResource.submit()** method will never send back a response to the client. The client will just time out with an error condition. Let’s expand on this example:



```Java
import javax.ws.rs.container.AsyncResponse;
import javax.ws.rs.container.Suspended;

@Path("/orders")
public class OrderResource {

   @POST
   @Consumes("application/json")
   @Produces("application/json")
   public void submit(final Order order,
                      final @Suspended AsyncResponse response) {
      new Thread() {
         public void run() {
            OrderConfirmation confirmation = orderProcessor.process(order);
            response.resume(order);
         }
      }.start();
   }
}
```


In the previous example, the client would just time out. Now, the **OrderResource.submit()** method spawns a new thread to handle order submission. This background thread processes the **Order** to obtain an **OrderConfirmation**. It then sends a response back to the client by calling the **AsyncResponse.resume()** method, passing in the **OrderConfirmation** instance. Invoking **resume()** in this manner means that it is a successful response. So, a status code of 200 is sent back to the client. Also, because we’re passing a Java object, the **resume()** method will marshal this object and send it within the HTTP response body. The media type used is determined by the **@Produces** annotation placed on the original JAX-RS method. If the **@Produces** annotation has more than one value, then the request’s Accept header is examined to pick the returned media type. Basically, this is the same algorithm a regular JAX-RS method uses to determine the media type.


Alternatively, you can pass **resume()** a **Response** object to send the client a more specific response:



```Java
import javax.ws.rs.container.AsyncResponse;
import javax.ws.rs.container.Suspended;

@Path("/orders")
public class OrderResource {

   @POST
   @Consumes("application/json")
   public void submit(final Order order,
                      final @Suspended AsyncResponse response) {
      new Thread() {
         public void run() {
            OrderConfirmation confirmation = orderProcessor.process(order);
            Response response = Response.ok(confirmation,
                                            MediaType.APPLICATION_XML_TYPE)
                                        .build();
            response.resume(response);
         }
      }.start();
   }
}
```


In this example, we’ve manually created a **Response**. We set the entity to the **OrderConfirmation** and the content type to XML.



### Exception Handling


In [Chapter 7](../chapter7/server_responses_and_exception_handling.md), we discussed what happens when a JAX-RS method throws an exception. When you invoke **AsyncResponse.resume(Object)**, the response filter and interceptor chains (see [Chapter 12](../chapter12/filters_and_interceptors.md)) are invoked, and then finally the **MessageBodyWriter**. If an exception is thrown by any one of these components, then the exception is handled in the same way as its synchronous counterpart with one caveat. Unhandled exceptions are not propagated, but instead the server will return a 500, “Internal Server Error,” back to the client.


Finally, the previous example is pretty simple, but what if it were possible for **orderProcessor.process()** to throw an exception? We can handle this exception by using the **AsyncResponse.resume(Throwable)** method:



```Java
import javax.ws.rs.container.AsyncResponse;
import javax.ws.rs.container.Suspended;

@Path("/orders")
public class OrderResource {

   @POST
   @Consumes("application/json")
   public void submit(final Order order,
                      final @Suspended AsyncResponse response) {
      new Thread() {
         public void run() {
            OrderConfirmation confirmation = null;
            try {
               confirmation = orderProcessor.process(order);
            } catch (Exception ex) {
               response.resume(ex);
               return;
            }
            Response response = Response.ok(confirmation,
                                            MediaType.APPLICATION_XML_TYPE)
                                        .build();
            response.resume(response);
         }
      }.start();
   }
}
```


Invoking **AsyncResponse.resume(Throwable)** is like throwing an exception from a regular synchronous JAX-RS method. Standard JAX-RS exception handling is performed on the passed-in **Throwable**. If a matching **ExceptionMapper** exists for the passed-in **Throwable**, it will be used. Otherwise, the server will send back a 500 status code.


### Cancel


There’s a few other convenience methods on **AsyncResponse** we haven’t covered yet:


```Java
package javax.ws.rs.container;

public interface AsyncResponse {
   boolean cancel();
   boolean cancel(int retryAfter);
   boolean cancel(Date retryAfter);
   ...
}
```


Each of the **cancel()** methods is really a precanned call to **resume()**:


```Java
// cancel()
response.resume(Response.status(503).build());

// cancel(int)
response.resume(Response.status(503)
                        .header(HttpHeaders.RETRY_AFTER, 100)
                        .build());
// cancel(Date)
response.resume(Response.status(503)
                        .header(HttpHeaders.RETRY_AFTER, date)
                        .build());
```


Internally, a **Response** object is built with a 503 status code. For **cancel()** methods that accept input, the parameter is used to initialize a **Retry-After** HTTP response header.


### Status Methods


There’s a few status methods on **AsyncResponse** that specify the state of the response:


```Java
public interface AsyncResponse {
   boolean isSuspended();
   boolean isCancelled();
   boolean isDone();

   ...
}
```

The **AsyncResponse.isCancelled()** method can be called to see if a **AsyncResponse** has been cancelled. **isSuspended()** specifies whether or not the response can have **resume()** or **cancel()** invoked. The **isDone()** method tells you if the response is finished.


### Timeouts


If an **AsyncResponse** is not resumed or cancelled, it will eventually time out. The default timeout is container-specific. A timeout results in a 503, “Service Unavailable,” response code sent back to the client. You can explicitly set the timeout by invoking the **setTimeout()** method:


```Java
response.setTimeout(5, TimeUnit.SECONDS);
```


You can also register a callback that is triggered when a timeout occurs by implementing the **TimeoutHandler** interface. For example:



```Java
response.setTimeoutHandler(
   new TimeoutHandler {
      void handleTimeout(AsyncResponse response) {
         response.resume(Response.serverError().build());
      }
   }
);
```


Here, instead of sending the default 503 response code to the client on a timeout, the example registers a **TimeoutHandler** that sends a 500 response code instead.



### Callbacks


The **AsyncResponse** interface also allows you to register callback objects for other types of events:


```Java
package javax.ws.rs.container;

public interface CompletionCallback {
        public void onComplete(Throwable throwable);
}
```


**CompletionCallback.onComplete()** is called after the response has been sent to the client. The **Throwable** is set to any unmapped exception thrown internally when processing a **resume()**. Otherwise, it is null.


```Java
package javax.ws.rs.container;

public interface ConnectionCallback {
        public void onDisconnect(AsyncResponse response);
}
```


The JAX-RS container does not require implementation of the **ConnectionCallback**. It allows you to be notified if the socket connection is disconnected while processing the response.


You enable callbacks by invoking the **AsyncResponse.register()** methods. You can pass one or more classes that will be instantiated by the JAX-RS container, and you can pass one or more instances:



```Java
response.register(MyCompletionCallback.class);
response.register(new MyConnectionCallback());
```


Callbacks are generally used to receive notification of error conditions caused after invoking **resume()**. You may have resources to clean up or even transactions to roll back or undo as a result of an asynchronous failure.



### Use Cases for AsyncResponse


The examples used in the previous section were really contrived to make it simple to explain the behavior of the asynchronous APIs. As I mentioned before, there is a specific set of use cases for async response processing. Let’s go over it.


#### Server-side push


With server-side push, the server is sending events back to the client. A typical example is stock quotes. The client wants to be notified when a new quote is available. It does a long-poll GET request until the quote is ready.


```Java
Client client = ClientBuilder.newClient();
final WebTarget target = client.target("http://quote.com/quote/RHT");
target.request().async().get(new InvocationCallback<String> {

   public void completed(String quote) {
       System.out.println("RHT: " + quote);
       target.request().async().get(this);
   }

   public void failed(Throwable t) {}
}
```


The preceding continuously polls for a quote using **InvocationCallback**. On the server side, we want our JAX-RS resource classes to use suspended requests so that we can have one thread that writes quotes back to polling clients. With one writer thread, we can scale this quote service to thousands and thousands of clients, as we’re not beholden to a “one thread per request” model. Here’s what the JAX-RS resource class might look like:


```Java
@Path("quote/RHT")
public class RHTQuoteResource {

   protected List<AsyncResponse> responses;

   @GET
   @Produces("text/plain")
   public void getQuote(@Suspended AsyncResponse response) {
      synchronized (responses) {
         responses.put(response);
      }
   }
}
```


The example code is overly simplified, but the idea is that there is a **List** of **AsyncResponse** objects that are waiting for the latest stock quote for Red Hat. This **List** would be shared by a background thread that would send a response back to all waiting clients when a new quote for Red Hat became available.


```Java
Executor executor = Executors.newSingleThreadExecutor();
final List<AsyncResponse> responses = ...;
final Ticker rhtTicker = nyse.getTicker("RHT");
executor.execute(new Runnable() {

   public void run() {
      while (true) {
         String quote = ticker.await();
         synchronized (responses) {
            for (AsyncResponse response : responses) response.resume(quote);
         }
      }
   }
});
```


So, here we’re starting a background thread that runs continuously using the **Executors** class from the **java.util.concurrent** package that comes with the JDK. This thread blocks until a quote for Red Hat is available. Then it loops over every awaiting **AsyncResponse** to send the quote back to each client. Some of the implementation is missing here, but hopefully you get the idea.


#### Publish and subscribe


Another great use case for **AsyncResponse** is publish and subscribe applications, an example being a chat service. Here’s what the server code might look like:


```Java
@Path("chat")
public class ChatResource {
   protected List<AsyncResponse> responses = new ArrayList<AsyncResponse>();

   @GET
   @Produces("text/plain")
   public synchronized void receive(@Suspended AsyncResponse response) {
      responses.add(response);
   }

   @POST
   @Consume("text/plain")
   public synchronized void send(String message) {
      for (AsyncResponse response : responses) {
         response.resume(message);
      }
   }
}
```


This is a really poor chat implementation, as messages could be lost for clients that are repolling, but hopefully it illustrates how you might create such an application. In [Chapter 27](../../part2/chapter27/example_ex13_1_chat_rest_interface.md), we’ll implement a more robust and complete chat service.


> ##### Note
> With protocols like WebSocket[^13] and Server Sent Events (SSE)[^14] being supported in most browsers, pure HTTP server push and pub-sub are fast becoming legacy. So, if you’re only going to have browser clients for this kind of app, you’re probably better off using WebSockets or SSE.



#### Priority scheduling


Sometimes there are certain services that are highly CPU-intensive. If you have too many of these types of requests running, you can completely starve users who are making simple, fast requests. To resolve this issue, you can queue up these expensive requests in a separate thread pool that guarantees that only a few of these expensive operations will happen concurrently:


```Java
@Path("orders")
public class OrderResource {

   Executor executor;

   public OrderResource {
      executor = Executors.newSingleThreadExecutor();
   }

   @POST
   @Path("year_to_date_report")
   @Produces("application/json")
   public void ytdReport(final @FormParam("product") String product,
                         @AsyncResponse response) {

      executor.execute( new Runnable() {
         public void run() {
            Report report = generateYTDReportFor(product);
            response.resume(report);
         }
      }

   }


   protected Report generateYTDReportFor(String product) {
      ...
   }
}
```


Here we’re back to our familiar **OrderResource** again. We have a **ytdReport()** method that calculates buying patterns for a specific product for the year to date. We want to allow only one of these requests to execute at a time, as the calculation is extremely expensive. We set up a single-threaded **java.util.concurrent.Executor** in the **OrderResource** constructor. The **ytdReport()** method queues up a **Runnable** in this **Executor** that generates the report and sends it back to the client. If the **Executor** is currently busy generating a report, the request is queued up and executed after that report is finished.


---
[^13] For more information, see http://www.websocket.org.

[^14] For more information, see http://www.w3.org/TR/2011/WD-eventsource-20110208.
