# Caching


Caching is one of the more important features of the Web. When you visit a website for the first time, your browser stores images and static text in memory and on disk. If you revisit the site within minutes, hours, days, or even months, your browser doesn’t have to reload the data over the network and can instead pick it up locally. This greatly speeds up the rendering of revisited web pages and makes the browsing experience much more fluid. Browser caching not only helps page viewing, it also cuts down on server load. If the browser is obtaining images or text locally, it is not eating up scarce server bandwidth or CPU cycles.



Besides browser caching, there are also proxy caches. Proxy caches are pseudo–web servers that work as middlemen between browsers and websites. Their sole purpose is to ease the load on master servers by caching static content and serving it to clients directly, bypassing the main servers. Content delivery networks (CDNs) like Akamai have made multimillion-dollar businesses out of this concept. These CDNs provide you with a worldwide network of proxy caches that you can use to publish your website and scale to hundreds of thousand of users.


If your web services are RESTful, there’s no reason you can’t leverage the caching semantics of the Web within your applications. If you have followed the HTTP constrained interface religiously, any service URI that can be reached with an HTTP GET is a candidate for caching, as they are, by definition, read-only and idempotent.


So when do you cache? Any service that provides static unchanging data is an obvious candidate. Also, if you have more dynamic data that is being accessed concurrently, you may also want to consider caching, even if your data is valid for only a few seconds or minutes. For example, consider the free stock quote services available on many websites. If you read the fine print, you’ll see that these stock quotes are between 5 and 15 minutes old. Caching is viable in this scenario because there is a high chance that a given quote is accessed more than once within the small window of validity. So, even if you have dynamic web services, there’s still a good chance that web caching is viable for these services.



### HTTP Caching


Before we can leverage web caching, proxy caches, and CDNs for our web services, we need to understand how caching on the Web works. The HTTP protocol defines a rich set of built-in caching semantics. Through the exchange of various request and response headers, the HTTP protocol gives you fine-grained control over the caching behavior of both browser and proxy caches. The protocol also has validation semantics to make managing caches much more efficient. Let’s dive into the specifics.



### Expires Header


How does a browser know when to cache? In HTTP 1.0, a simple response header called **Expires** tells the browser that it can cache and for how long. The value of this header is a date in the future when the data is no longer valid. When this date is reached, the client should no longer use the cached data and should retrieve the data again from the server. For example, if a client submitted **GET /customers/123**, an example response using the **Expires** header would look like this:


```xml
HTTP/1.1 200 OK
Content-Type: application/xml
Expires: Tue, 15 May 2014 16:00 GMT

<customer id="123">...</customers>
```


This cacheable XML data is valid until Tuesday, May 15, 2014.


We can implement this within JAX-RS by using a **javax.ws.rs.core.Response** object. For example:



```Java
@Path("/customers")
public class CustomerResource {

   @Path("{id}")
   @GET
   @Produces("application/xml")
   public Response getCustomer(@PathParam("id") int id) {
      Customer cust = findCustomer(id);
      ResponseBuilder builder = Response.ok(cust, "application/xml");
      Date date = Calendar.getInstance(TimeZone.getTimeZone("GMT"))
                          .set(2010, 5, 15, 16, 0);
      builder.expires(date);
      return builder.build();
   }
```


In this example, we initialize a **java.util.Date** object and pass it to the **ResponseBuilder.expires()** method. This method sets the **Expires** header to the string date format the header expects.



### Cache-Control



HTTP caching semantics were completely redone for the HTTP 1.1 specification. The specification includes a much richer feature set that has more explicit controls over browser and CDN/proxy caches. The idea of cache revalidation was also introduced. To provide all this new functionality, the **Expires** header was deprecated in favor of the **Cache-Control** header. Instead of a date, **Cache-Control** has a variable set of comma-delimited directives that define who can cache, how, and for how long. Let’s take a look at them:


* **private**

	The **private** directive states that no shared intermediary (proxy or CDN) is allowed to cache the response. This is a great way to make sure that the client, and only the client, caches the data.

* **public**

	The **public** directive is the opposite of **private**. It indicates that the response may be cached by any entity within the request/response chain.

* **no-cache**

	Usually, this directive simply means that the response should not be cached. If it is cached anyway, the data should not be used to satisfy a request unless it is revalidated with the server (more on revalidation later).

* **no-store**

	A browser will store cacheable responses on disk so that they can be used after a browser restart or computer reboot. You can direct the browser or proxy cache to not store cached data on disk by using the **no-store** directive.

* **no-transform**

	Some intermediary caches have the option to automatically transform their cached data to save memory or disk space or to simply reduce network traffic. An example is compressing images. For some applications, you might want to disallow this using the **no-transform** directive.

* **max-age**

	This directive is how long (in seconds) the cache is valid. If both an **Expires** header and a **max-age** directive are set in the same response, the **max-age** always takes precedence.

* **s-maxage**

	The **s-maxage** directive is the same as the **max-age** directive, but it specifies the maximum time a shared, intermediary cache (like a proxy) is allowed to hold the data. This directive allows you to have different expiration times than the client.



Let’s take a look at a simple example of a response to see **Cache-Control** in action:



```xml
HTTP/1.1 200 OK
Content-Type: application/xml
Cache-Control: private, no-store, max-age=300

<customers>...</customers>
```


In this example, the response is saying that only the client may cache the response. This response is valid for 300 seconds and must not be stored on disk.


The JAX-RS specification provides **javax.ws.rs.core.CacheControl**, a simple class to represent the **Cache-Control** header:



```Java
public class CacheControl {
   public CacheControl() {...}

   public static CacheControl valueOf(String value)
               throws IllegalArgumentException {...}
   public boolean isMustRevalidate() {...}
   public void setMustRevalidate(boolean mustRevalidate) {...}
   public boolean isProxyRevalidate() {...}
   public void setProxyRevalidate(boolean proxyRevalidate) {...}
   public int getMaxAge() {...}
   public void setMaxAge(int maxAge) {...}
   public int getSMaxAge() {...}
   public void setSMaxAge(int sMaxAge) {...}
   public List<String> getNoCacheFields() {...}
   public void setNoCache(boolean noCache) {...}
   public boolean isNoCache() {...}
   public boolean isPrivate() {...}
   public List<String> getPrivateFields() {...}
   public void setPrivate(boolean _private) {...}
   public boolean isNoTransform() {...}
   public void setNoTransform(boolean noTransform) {...}
   public boolean isNoStore() {...}
   public void setNoStore(boolean noStore) {...}
   public Map<String, String> getCacheExtension() {...}
}
```


The **ResponseBuilder** class has a method called **cacheControl()** that can accept a **CacheControl** object:


```Java
@Path("/customers")
public class CustomerResource {

   @Path("{id}")
   @GET
   @Produces("application/xml")
   public Response getCustomer(@PathParam("id") int id) {
      Customer cust = findCustomer(id);

      CacheControl cc = new CacheControl();
      cc.setMaxAge(300);
      cc.setPrivate(true);
      cc.setNoStore(true);
      ResponseBuilder builder = Response.ok(cust, "application/xml");
      builder.cacheControl(cc);
      return builder.build();
   }
```



In this example, we initialize a **CacheControl** object and pass it to the **ResponseBuilder.cacheControl()** method to set the **Cache-Control** header of the response. Unfortunately, JAX-RS doesn’t yet have any nice annotations to do this for you automatically.



### Revalidation and Conditional GETs



One interesting aspect of the caching protocol is that when the cache is stale, the cacher can ask the server if the data it is holding is still valid. This is called *revalidation*. To be able to perform revalidation, the client needs some extra information from the server about the resource it is caching. The server will send back a **Last-Modified** and/or an **ETag** header with its initial response to the client.


#### Last-Modified


The **Last-Modified** header represents a timestamp of the data sent by the server. Here’s an example response:


```xml
HTTP/1.1 200 OK
Content-Type: application/xml
Cache-Control: max-age=1000
Last-Modified: Tue, 15 May 2013 09:56 EST

<customer id="123">...</customer>
```



This initial response from the server is stating that the XML returned is valid for 1,000 seconds and has a timestamp of Tuesday, May 15, 2013, 9:56 AM EST. If the client supports revalidation, it will store this timestamp along with the cached data. After 1,000 seconds, the client may opt to revalidate its cache of the item. To do this, it does a *conditional* GET request by passing a request header called **If-Modified-Since** with the value of the cached **Last-Modified** header. For example:


```
GET /customers/123 HTTP/1.1
If-Modified-Since: Tue, 15 May 2013 09:56 EST
```


When a service receives this GET request, it checks to see if its resource has been modified since the date provided within the **If-Modified-Since** header. If it has been changed since the timestamp provided, the server will send back a 200, “OK,” response with the new representation of the resource. If it hasn’t been changed, the server will respond with 304, “Not Modified,” and return no representation. In both cases, the server should send an updated **Cache-Control** and **Last-Modified** header if appropriate.



#### ETag


The **ETag** header is a pseudounique identifier that represents the version of the data sent back. Its value is any arbitrary quoted string and is usually an MD5 hash. Here’s an example response:


```xml
HTTP/1.1 200 OK
Content-Type: application/xml
Cache-Control: max-age=1000
ETag: "3141271342554322343200"

<customer id="123">...</customer>
```


Like the **Last-Modified** header, when the client caches this response, it should also cache the **ETag** value. When the cache expires after 1,000 seconds, the client performs a revalidation request with the **If-None-Match** header that contains the value of the cached **ETag**. For example:


```
GET /customers/123 HTTP/1.1
If-None-Match: "3141271342554322343200"
```


When a service receives this GET request, it tries to match the current **ETag** hash of the resource with the one provided within the **If-None-Match** header. If the tags don’t match, the server will send back a 200, “OK,” response with the new representation of the resource. If it hasn’t been changed, the server will respond with 304, “Not Modified,” and return no representation. In both cases, the server should send an updated **Cache-Control** and **ETag** header if appropriate.


One final thing about **ETags** is they come in two flavors: strong and weak. A strong **ETag** should change whenever any bit of the resource’s representation changes. A weak **ETag** changes only on semantically significant events. Weak **ETags** are identified with a **W/** prefix. For example:


```xml
HTTP/1.1 200 OK
Content-Type: application/xml
Cache-Control: max-age=1000
ETag: W/"3141271342554322343200"

<customer id="123">...</customer>
```


Weak **ETags** give applications a bit more flexibility to reduce network traffic, as a cache can be revalidated when there have been only minor changes to the resource.


JAX-RS has a simple class called **javax.ws.rs.core.EntityTag** that represents the **ETag** header:


```Java
public class EntityTag {

   public EntityTag(String value) {...}
   public EntityTag(String value, boolean weak) {...}
   public static EntityTag valueOf(String value)
                throws IllegalArgumentException {...}
   public boolean isWeak() {...}
   public String getValue() {...}
}
```



It is constructed with a string value and optionally with a flag telling the object if it is a weak **ETag** or not. The **getValue()** and **isWeak()** methods return these values on demand.




#### JAX-RS and conditional GETs



To help with conditional GETs, JAX-RS provides an injectable helper class called **javax.ws.rs.core.Request**:


```Java
public interface Request {
   ...

   ResponseBuilder evaluatePreconditions(EntityTag eTag);
   ResponseBuilder evaluatePreconditions(Date lastModified);
   ResponseBuilder evaluatePreconditions(Date lastModified, EntityTag eTag);
}
```


The overloaded **evaluatePreconditions()** methods take a **javax.ws.rs.core.EntityTag**, a **java.util.Date** that represents the last modified timestamp, or both. These values should be current, as they will be compared with the values of the **If-Modified-Since**, **If-Unmodified-Since**, or **If-None-Match** headers sent with the request. If these headers don’t exist or if the request header values don’t pass revalidation, this method returns null and you should send back a 200, “OK,” response with the new representation of the resource. If the method does not return null, it returns a preinitialized instance of a **ResponseBuilder** with the response code preset to 304. For example:



```Java
@Path("/customers")
public class CustomerResource {

   @Path("{id}")
   @GET
   @Produces("application/xml")
   public Response getCustomer(@PathParam("id") int id,
                                @Context Request request) {
      Customer cust = findCustomer(id);
      EntityTag tag = new EntityTag(
                                Integer.toString(cust.hashCode()));

      CacheControl cc = new CacheControl();
      cc.setMaxAge(1000);

      ResponseBuilder builder = request.evaluatePreconditions(tag);
      if (builder != null) {
         builder.cacheControl(cc);
         return builder.build();
      }

      // Preconditions not met!

      builder = Response.ok(cust, "application/xml");
      builder.cacheControl(cc);
      builder.tag(tag);
      return builder.build();
   }
```



In this example, we have a **getCustomer()** method that handles GET requests for the **/customers/\\{id}** URI pattern. An instance of **javax.ws.rs.core.Request** is injected into the method using the **@Context** annotation. We then find a **Customer** instance and create a current **ETag** value for it from the hash code of the object (this isn’t the best way to create the **EntityTag**, but for simplicity’s sake, let’s keep it that way). We then call **Request.evaluatePreconditions()**, passing in the up-to-date tag. If the tags match, we reset the client’s cache expiration by sending a new **Cache-Control** header and return. If the tags don’t match, we build a **Response** with the new, current version of the **ETag** and **Customer**.













