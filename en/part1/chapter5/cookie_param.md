# @CookieParam


Servers can store state information in cookies on the client, and can retrieve that information when the client makes its next request. Many web applications use cookies to set up a session between the client and the server. They also use cookies to remember identity and user preferences between requests. These cookie values are transmitted back and forth between the client and server via cookie headers.


The **@javax.ws.rs.CookieParam** annotation allows you to inject cookies sent by a client request into your JAX-RS resource methods. For example, let’s say our applications push a **customerId** cookie to our clients so that we can track users as they invoke and interact with our web services. Code to pull in this information might look like this:


```Java
@Path("/myservice")
public class MyService {

   @GET
   @Produces("text/html")
   public String get(@CookieParam("customerId") int custId) {
     ...
   }
}
```


The use of **@CookieParam** here makes the JAX-RS provider search all cookie headers for the **customerId** cookie value. It then converts it into an int and injects it into the **custId** parameter.


If you need more information about the cookie other than its base value, you can instead inject a **javax.ws.rs.core.Cookie** object:


```Java
@Path("/myservice")
public class MyService {

   @GET
   @Produces("text/html")
   public String get(@CookieParam("customerId") Cookie custId) {
     ...
   }
}
```


The **Cookie** class has additional contextual information about the cookie beyond its name and value:


```Java
package javax.ws.rs.core;

public class Cookie
{
   public String getName() {...}
   public String getValue() {...}
   public int getVersion() {...}
   public String getDomain() {...}
   public String getPath() {...}

...
}
```


The **getName()** and **getValue()** methods correspond to the string name and value of the cookie you are injecting. The **getVersion()** method defines the format of the cookie header—specifically, which version of the cookie specification the header follows.[^4] The **getDomain()** method specifies the DNS name that the cookie matched. The **getPath()** method corresponds to the URI path that was used to match the cookie to the incoming request. All these attributes are defined in detail by the IETF cookie specification.


You can also obtain a map of all cookies sent by the client by injecting a reference to **javax.ws.rs.core.HttpHeaders**:


```Java
public interface HttpHeaders {
...
   public Map<String, Cookie> getCookies();
}
```


As you saw in the previous section, you use the **@Context** annotation to get access to **HttpHeaders**. Here’s an example of logging all cookies sent by the client:


```Java
@Path("/myservice")
public class MyService {

   @GET
   @Produces("text/html")
   public String get(@Context HttpHeaders headers) {
      for (String name : headers.getCookies().keySet())
      {
         Cookie cookie = headers.getCookies().get(name);
         System.out.println("Cookie: " +
                         name + "=" + cookie.getValue());
      }
     ...
   }
}
```


---
[^4] For more information, see http://www.ietf.org/rfc/rfc2109.txt.





