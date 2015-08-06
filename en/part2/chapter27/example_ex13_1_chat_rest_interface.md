# Example ex13_1: Chat REST Interface


Before we dive into code, let me explain the REST interface for our chat service. The service will share a URL to both send and receive chat messages. The service will work much like Twitter in that if one user posts a chat, anybody listening for chats will see it. Posting a chat is a simple HTTP POST request. Here’s an example request:


```
POST /chat HTTP/1.1
Host: localhost:8080
Content-Type: text/plain

Hello everybody
```


As you can see, all the user has to do is post a simple text message to the */chat* URL and messages will be sent to all listeners.


To receive chat messages, clients will make a blocking GET request to the chat server:


```
GET /chat HTTP/1.1
Host: localhost:8080
```


When a chat becomes available, this GET request returns with the next chat message. Additionally, a **next** **Link** header is sent back with the HTTP response:


```
GET /chat?current=1 HTTP/1.1
Host: localhost:800
```


The **next** link’s URI contains a query parameter identifying to the server the last message the client read. The server will use this index to obtain the next message so that the client sees all messages in order. This new GET request will either block again, or immediately return a queued chat message. The pattern then repeats itself. The response will contain a new **next** **Link** header with a new pointer into the message queue:


```
HTTP/1.1 200 OK
Content-Type: text/plain
Link: </chat?current=2>; rel=next

What's up?
```


The server will buffer the latest 10 chat messages in a linked list so that it can easily find the next message a particular chat client needs. This is an example of a HATEOAS flow, where the client transitions its state using a link passed back from the server.



### The Client Code


The client is a console program that takes input from the command line while at the same time printing out the current chat message. To run the client, you must specify the name you want to use to post messages as an initial argument when you start up the program.


```Java:src/main/java/ChatClient.java
public class ChatClient
{
   public static void main(String[] args) throws Exception
   {
      String name = args[0];
...
}
```


After grabbing the client’s name from the argument list, we then initialize a **Client** that we’ll use to invoke on the customer chat service:


```Java
      final Client client = new ResteasyClientBuilder()
                          .connectionPoolSize(3)
                          .build();
      WebTarget target = client.target("http://localhost:8080/services/chat");
```


By default, RESTEasy allows only one connection per **Client** to be open at one time. So we use the proprietary **ClientBuilder** implementation of RESTEasy to set a connection pool size of 3. We also initialize a **WebTarget** with the URL of the chat service.


Next, we use the JAX-RS client asynchronous callback API to set up a loop to pull chat messages from the server:


```Java
      target.request().async().get(new InvocationCallback<Response>()
      {
         @Override
         public void completed(Response response)
         {
            Link next = response.getLink("next");
            String message = response.readEntity(String.class);
            System.out.println();
            System.out.print(message);// + "\r");
            System.out.println();
            System.out.print("> ");
            client.target(next).request().async().get(this);
         }

         @Override
         public void failed(Throwable throwable)
         {
            System.err.println("FAILURE!");
         }
      });
```


The code starts off by making an async request to the base chat URI. This invocation registers an **InvocationCallback** interface that we’ve implemented as a Java inner class. When the initial GET request is complete, the **InvocationCallback.complete()** method is invoked, passing in the **Response** from the server. We first extract the **next** **Link** header and the chat message from the **Response**. We then print the message to the console. Finally, we make a new asynchronous GET request using the URI contained in the **next** **Link** header. We register the current **InvocationCallback** instance with this new request. This will set up a continuous pull request with the chat service.


After we’ve set up our receive loop, we set up another loop that allows us to send chat messages:


```Java
      while (true)
      {
         System.out.print("> ");
         BufferedReader br = new BufferedReader(
                                   new InputStreamReader(System.in));
         String message = br.readLine();
         target.request().post(Entity.text(name + ": " + message));
      }
```


We simply read from **stdin** until the user hits Enter and then do an HTTP POST request to the chat service with the command-line input.



### The Server Code


The server side is doing a lot of different things to implement our chat service. Let’s break it down:



```Java:src/main/java/com/restfully/shop/services/CustomerChat.java
@Path("chat")
public class CustomerChat
{
   class Message
   {
      String id;
      String message;
      Message next;
   }
```


The **CustomerChat** class is annotated with **@Path** to specify the root resource path of our JAX-RS service. It then declares a simple inner class called **Message** that will represent the queued chat messages. A message is represented by a **String** **id** and a **String** **message**, and also contains a reference to the next queued **Message**.


```Java
   protected Message first;
   protected Message last;
```


The service remembers what the current first and last message is. It stores these in the **first** and **last** member variables of the class.


```Java
   protected int maxMessages = 10;
   protected LinkedHashMap<String, Message> messages =
                                        new LinkedHashMap<String, Message>()
   {
      @Override
      protected boolean removeEldestEntry(Map.Entry<String, Message> eldest)
      {
         boolean remove = size() > maxMessages;
         if (remove) first = eldest.getValue().next;
         return remove;
      }
   };
```


**Message** objects are stored in a **java.util.LinkedHashMap** so that they can be easily looked up when a chat client makes a GET request. The key of this map is the **id** of the **Message**. The service will always queue the last 10 messages posted to the server. We use a **LinkedHashMap** so that we can easily evict the oldest chat message when the maximum number of buffered messages is reached. The **removeEldestEntry()** method is used to determine when to evict the oldest entry in the map. It simply checks to see if the size of the map is greater than the maximum amount of messages. It then resets what the **first** message is. Returning **true** triggers the removal of the eldest entry.


```Java
   protected AtomicLong counter = new AtomicLong(0);
```


The **AtomicLong** **counter** variable is used to generate message IDs.


```Java
   LinkedList<AsyncResponse> listeners = new LinkedList<AsyncResponse>();
```


The **listeners** variable stores a list of waiting chat clients. We’ll see how this is used later.


```Java
   ExecutorService writer = Executors.newSingleThreadExecutor();
```


We will have one and only one thread that is responsible for writing response messages back to the chat clients. Having one *writer* thread is what makes this whole application scale very well. Without asynchronous JAX-RS, this service would require a thread per blocking chat client. While most modern operating systems can handle one or two thousand threads, system performance starts to degrade quickly with all the context switching the operating system has to do. Asynchronous JAX-RS allows us to scale to a much larger number of concurrent users.


#### Posting a new message


Let’s look at how the service handles a new chat message:


```Java
   @Context
   protected UriInfo uriInfo;

   @POST
   @Consumes("text/plain")
   public void post(final String text)
   {
      final UriBuilder base = uriInfo.getBaseUriBuilder();
      writer.submit(new Runnable());
```


The **post()** method consumes plain-text data. The first thing we do is store a **UriBuilder** in a local variable of the base URI of our service. We then queue up a task for our writer thread. We cannot use the **UriInfo** member variable in this background task. The **CustomerChat** class is a singleton and can accept requests concurrently. Because of this, the **UriInfo uriInfo** member variable is a proxy that delegates to the request’s actual **UriInfo** by using an underlying **ThreadLocal** in most JAX-RS vendor implementations. If the *writer* background thread invokes on this proxy, it would get an error because **ThreadLocal** data is not transferred between different threads.


```Java
      writer.submit(new Runnable());
      {
         @Override
         public void run()
         {
            synchronized (messages)
            {
               Message message = new Message();
               message.id = Long.toString(counter.incrementAndGet());
               message.message = text;
```


Each new message post is queued up for the *writer* thread in an implementation of the **Runnable** interface. This task starts off by synchronizing on the **messages** variable. This protects the critical parts of our message service by serializing access to the **messages** map. The code then creates a **Message** instance using an **id** generated from the **counter**.


```Java
               if (messages.size() == 0)
               {
                  first = message;
               }
               else
               {
                  last.next = message;
               }
```


The *writer* thread next checks to see if this is the initial message to the system. If so, it sets the first member variable to point to the first message posted to the service. Otherwise, it points the tail of the **Message** linked list to this new **Message** instance.


```Java
               messages.put(message.id, message);
               last = message;
```


The code then stores the new message in the **messages** map and sets the **last** member variable to point to this new message.


```Java
               for (AsyncResponse async : listeners)
               {
                  try
                  {
                     send(base, async, message);
                  }
                  catch (Exception e)
                  {
                     e.printStackTrace();
                  }
               }
               listeners.clear();
            }
         }
      });
   }
```


Finally, the *writer* thread loops through all waiting chat clients and sends them the new message.


#### Handling poll requests


The **CustomerChat.receive()** method handles GET requests from chat clients:


```Java
   @GET
   public void receive(@QueryParam("current") String id,
                       @Suspended AsyncResponse async)
   {
      final UriBuilder base = uriInfo.getBaseUriBuilder();
      Message message = null;
      synchronized (messages)
      {
         Message current = messages.get(id);
         if (current == null) message = first;
         else message = current.next;

         if (message == null) {
            queue(async);
         }
      }
      // do this outside of synchronized block to reduce lock hold time
      if (message != null) send(base, async, message);
    }
```


The **receive()** method takes a query parameter, **current**. This parameter is the **id** of the last message the chat client read. This parameter is allowed to be **null** if this is the chat client’s first pull request. Injecting the **async** parameter via the **@Suspended** annotation detaches HTTP response processing from this request thread.


The method then begins by defining a **synchronized** block on the **messages** variable. This block allows the **receive()** method to perform atomic actions that do not conflict with the *writer* thread. Within the block, the code looks up the **current** query parameter in the **messages** map. If the message is **null**, then the code sets this variable to the **first** member variable of the class. Otherwise, it sets the message to the found message’s **next** field. If the message is still **null**, then there is no message available and the **AsyncResponse** is queued for the *writer* thread to pick up when a message is available.


Finally, after the **synchronized** block, if the message is not **null**, it is sent immediately back to the chat client.


```Java
   protected void queue(AsyncResponse async)
   {
      listeners.add(async);
   }
```


The **queue()** method just adds the **AsyncResponse** to the **listeners** list so the *writer* thread can pick it up.


```Java
   protected void send(UriBuilder base, AsyncResponse async, Message message)
   {
      URI nextUri = base.clone().path(CustomerChat.class)
              .queryParam("current", message.id).build();
      Link next = Link.fromUri(nextUri).rel("next").build();
      Response response = Response.ok(message.message, MediaType.TEXT_PLAIN_TYPE)
                                  .links(next).build();
      async.resume(response);
   }
```


The **send()** method can be called by the *writer* thread or the **receive()** method. It creates a **Response** populated with the message that will be sent back to the chat client. It also calculates and adds a **next** **Link** header to send back with the response. At the end of the method, the **AsyncResponse.resume()** method is invoked with the built **Response**.


### Build and Run the Example Program


You’ll need multiple console windows to run this example. In the first console window, perform the following steps:

1. Change to the *ex13_1* directory of the workbook example code. 
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md). 
3. Perform the build and run the example by typing **maven jetty:run**.


This will start the JAX-RS services for the example.


Open another console window and do the following.

1. Change to the *ex13_1* directory of the workbook example code. 
2. Run the chat client by typing **maven exec:java -Dexec.mainClass=ChatClient -Dexec.args=" your-name"**.


Replace **your-name** with your first name. Repeat this process in yet another console window to run a second chat client. Finally, start typing chat messages.