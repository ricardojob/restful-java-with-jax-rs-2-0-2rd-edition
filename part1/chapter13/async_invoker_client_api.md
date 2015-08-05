# AsyncInvoker Client API


The client asynchronous API allows you to spin off a bunch of HTTP requests in the background and then either poll for a response, or register a callback that is invoked when the HTTP response is available. To invoke an HTTP request asynchronously on the client, you interact with the **javax.ws.rs.client.AsyncInvoker** interface or the **submit()** methods on **javax.ws.rs.client.Invocation**. First, let’s take a look at polling HTTP requests that are run in the background.



### Using Futures


The **AsyncInvoker** interface has a bunch of methods that invoke HTTP requests asynchronously and that return a **java.util.concurrent.Future** instance. You can use the **AsyncInvoker** methods by invoking the **async()** method on the **Invocation.Builder** interface.


```Java
package javax.ws.rs.client;

public interface AsyncInvoker {
    Future<Response> get();
    <T> Future<T> get(Class<T> responseType);

    Future<Response> put(Entity<?> entity);
    <T> Future<T> put(Entity<?> entity, Class<T> responseType);

    Future<Response> post(Entity<?> entity);
    <T> Future<T> post(Entity<?> entity, Class<T> responseType);

    Future<Response> delete(Entity<?> entity);
    <T> Future<T> delete(Entity<?> entity, Class<T> responseType);

    ...
}
```


The **Future** interface is defined within the **java.util.concurrent** package that comes with the JDK. For JAX-RS, it gives us a nice reusable interface for polling HTTP responses in either a blocking or nonblocking manner. If you’ve used **java.util.concurrent.Executors** or **@Asynchronous** within an EJB container, using the **Future** interface should be very familiar to you.



```Java
package java.util.concurrent;

public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```


This is best explained in a full example:


```Java
Client client = ClientBuilder.newClient();

Future<Response> future1 = client.target("http://example.com/customers/123")
                                 .request()
                                 .async().get();

Future<Order> future2 = client.target("http://foobar.com/orders/456")
                               .request()
                               .async().get(Order.class);

// block until complete
Response res1 = future1.get();
Customer result1 = res.readEntity(Customer.class);

// Wait 5 seconds
try {
   Order result2 = future2.get(5, TimeUnit.SECONDS);
} catch (TimeoutException timeout ) {
    ... handle exception ...
}
```


In this example, two separate requests are executed in parallel. With **future1** we want a full **javax.ws.rs.core.Response**. After executing both requests, we poll and block indefinitely on future1 by calling **Future.get()** until we get a Response back from that service.



With **future2**, we instead poll and block for five seconds only. For this second HTTP asynchronous request, we let JAX-RS automatically unmarshal the HTTP response body into an **Order**. **java.util.concurrent.TimeoutException** is thrown if the call takes longer than five seconds. You can also invoke the nonblocking **isDone()** or **isCancelled()** methods on **Future** to see if the request is finished or cancelled.


#### Exception handling


Exceptions that can be thrown by **Future.get()** methods are defined by that interface. **java.util.concurrent.TimeoutException** occurs if we are calling **Future.get()** with a timeout. **InterruptedException** happens if the calling thread has been interrupted. **java.util.concurrent.ExecutionException** is a wrapper exception. Any exception thrown by the JAX-RS runtime is caught and wrapped by this exception. Let’s expand on the **future2** example to see how this works:


```Java
// Wait 5 seconds
try {
   Order result2 = future2.get(5, TimeUnit.SECONDS);
} catch (TimeoutException timeout ) {
    System.err.println("request timed out");
} catch (InterruptedException ie) {
    System.err.println("Request was interrupted");
} catch (ExecutionException ee) {
  Throwable cause = ee.getCause();

  if (cause instanceof WebApplicationException) {
     (WebApplicationException)wae = (WebApplicationException)cause;
     wae.close();
  } else if (cause instanceof ResponseProcessingException) {
     ResponseProcessingException rpe = (ResponseProcessingException)cause;
     rpe.close();
  } else if (cause instanceof ProcessingException) {
    // handle processing exception
  } else {
     // unknown
  }
}
```



You can obtain any exception thrown by the JAX-RS runtime when an asynchronous request is executed by calling the **ExecutionException.getCause()** method. The possible wrapped JAX-RS exceptions are the same as the synchronous ones discussed in [Chapter 8](../chapter8/client_introduction.md).


In the example, the call to **future2.get()** unmarshalls the response to an **Order** object. If the response is something other than 200, “OK,” then the JAX-RS runtime throws one of the exceptions from the JAX-RS error exception hierarchy (i.e., **NotFoundException** or **BadRequestException**). If an exception is thrown while you’re unmarshalling the response to a **Order**, then **ResponseProcessingException** is thrown.


> **Warning**  You should always make sure that the underlying JAX-RS response is closed. While most JAX-RS containers will have their **Response** objects implement a **finalize()** method, it is not a good idea to rely on the garbage collector to clean up your client connections. If you do not clean up your connections, you may end up with intermittent errors that pop up if the underlying **Client** or operating system has exhausted its limit of allowable open connections.



In fact, if we examine our initial example a bit further, there’s a lot of code we have to add to ensure that we are being good citizens and closing any open **Response** objects. Here’s what the final piece of code would look like:


```Java
Client client = ClientBuilder.newClient();

Future<Response> future1 = client.target("http://example.com/service")
                                 .request()
                                 .async().get();
Future<Order> future2 = null;
try {
   future2 = client.target("http://foobar.com/service2")
                   .request()
                   .async().get(Order.class);
} catch (Throwable ignored) {
   ignored.printStackTrace();
}

// block until complete
Response res1 = future1.get();
try {
   Customer result1 = res.readEntity(Customer.class);
} catch (Throwable ignored) {
   ignored.printStackTrace();
} finally {
   res1.close();
}

// if we successfully executed 2nd request
if (future2 != null) {
   // Wait 5 seconds
   try {
      Order result2 = future2.get(5, TimeUnit.SECONDS);
   } catch (TimeoutException timeout ) {
      System.err.println("request timed out");
   } catch (InterruptedException ie) {
      System.err.println("Request was interrupted");
   } catch (ExecutionException ee) {
      Throwable cause = ee.getCause();

      if (cause instanceof WebApplicationException) {
         (WebApplicationException)wae = (WebApplicationException)cause;
         wae.close();
      } else if (cause instanceof ResponseProcessingException) {
         ResponseProcessingException rpe = (ResponseProcessingException)cause;
         rpe.close();
      } else if (cause instanceof ProcessingException) {
         // handle processing exception
      } else {
         // unknown
      }
   }
}
```


As you can see, there’s a few more **try/catch** blocks we need to add to make sure that the response of each async request is closed.



### Using Callbacks


The **AsyncInvoker** interface has an additional callback invocation style. You can register an object that will be called back when the asynchronous invocation is ready for processing:


```Java
package javax.ws.rs.client;

public interface AsyncInvoker {
    <T> Future<T> get(InvocationCallback<T> callback);
    <T> Future<T> post(Entity<?> entity, InvocationCallback<T> callback);
    <T> Future<T> put(Entity<?> entity, InvocationCallback<T> callback);
    <T> Future<T> delete(Entity<?> entity, InvocationCallback<T> callback);

    ...
}
```



The **InvocationCallback** interface is a parameterized generic interface and has two simple methods you have to implement—one for successful responses, the other for failures:


```Java
package javax.rs.ws.client;

public interface InvocationCallback<RESPONSE> {
    public void completed(RESPONSE response);
    public void failed(Throwable throwable);
}
```



JAX-RS introspects the application class that implements **InvocationCallback** to determine whether your callback wants a **Response** object or if you want to unmarshal to a specific type. Let’s convert our **Future** example to use callbacks. First, we’ll implement a callback for our initial request:


```Java
public class CustomerCallback implements InvocationCallback<Response> {
   public void completed(Response response) {
      if (response.getStatus() == 200) {
         Customer cust = response.readEntity(Customer.class);
      } else {
         System.err.println("Request error: " + response.getStatus());
      }
   }

   public void failed(Throwable throwable) {
      throwable.printStackTrace();
   }
}
```


The **CustomerCallback** class implements **InvocationCallback** with a **Response** generic parameter. This means JAX-RS will call the **completed()** method and pass in an untouched **Response** object. If there is a problem sending the request to the server or the JAX-RS runtime is unable to create a **Response**, the **failed()** method will be invoked with the appropriate exception. Otherwise, if there is an HTTP response, then **completed()** will be called.


Next, let’s implement a different callback for our second parallel request. This time we want our successful HTTP responses to be converted into **Order** objects:



```Java
public class OrderCallback implements InvocationCallback<Order> {
   public void completed(Order order) {
      System.out.println("We received an order.");
   }

   public void failed(Throwable throwable) {
      if (throwable instanceof WebApplicationException) {
         WebApplicationException wae = (WebApplicationException)throwable;
         System.err.println("Failed with status:
                            " + wae.getResponse().getStatus());
      } else if (throwable instanceof ResponseProcessingException) {
         ResponseProcessingException rpe = (ResponseProcessingException)cause;
         System.err.println("Failed with status:
                            " + rpe.getResponse().getStatus());
      } else {
         throwable.printStackTrace();
      }
   }
}
```



This case is a little bit different than when we implement **InvocationCallback** with a **Response**. If there is a successful HTTP response from the server (like 200, “OK”), JAX-RS will attempt to unmarshal the response into an **Order** object. If there were an HTTP error response, or the JAX-RS runtime failed to unmarshal the response body into an **Order** object, then the **failed()** method is invoked with the appropriate exception. Basically, we see the same kind of exceptions thrown by similar synchronous invocations or the **Future** example we discussed earlier. You do not have to close the underlying **Response** object; JAX-RS will do this after **completed()** or **failed()** is invoked.


Now that our callback classes have been implemented, let’s finish our example by invoking on some services:



```Java
Client client = ClientBuilder.newClient();

Future<Response> future1 = client.target("http://example.com/customers/123")
                                 .request()
                                 .async().get(new CustomerCallback());

Future<Order> future2 = client.target("http://foobar.com/orders/456")
                               .request()
                               .async().get(new OrderCallback());
```


That’s all we have to do. Notice that the **get()** methods return a **Future** object. You can ignore this **Future**, or interact with it as we did previously. I suggest that you only use the **Future.cancel()** and **Future.isDone()** methods, though, as you may have concurrency issues with **InvocationCallback**.



### Futures Versus Callbacks


Given that we have two different ways to do asynchronous client invocations, which style should you use? Futures or callbacks? In general, use futures if you need to *join* a set of requests you’ve invoked asynchronously. By *join*, I mean you need to know when each of the requests has finished and you need to perform another task after *all* the asynchronous requests are complete. For example, maybe you are gathering information from a bunch of different web services to build a larger aggregated document (a mashup).



Use callbacks when each invocation is its own distinct unit and you do not have to do any coordination or mashing up.
