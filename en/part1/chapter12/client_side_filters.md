# Client-Side Filters


The JAX-RS Client API also has its own set of request and response filter interfaces:



```Java
package javax.ws.rs.client;

public interface ClientRequestFilter {
    public void filter(ClientRequestContext requestContext) throws IOException;
}

public interface ClientResponseFilter {
    public void filter(ClientRequestContext requestContext,
                           ClientResponseContext responseContext)
            throws IOException;
}
```


Let’s use these two interfaces to implement a client-side cache. We want this cache to behave like a browser’s cache. This means we want it to honor the **Cache-Control** semantics discussed in [Chapter 11](../chapter11/scaling_jax_rs_applications.md). We want cache entries to expire based on the metadata within **Cache-Control** response headers. We want to perform conditional GETs if the client is requesting an expired cache entry. Let’s implement our **ClientRequestFilter** first:



```Java
import javax.ws.rs.client.ClientRequestFilter;
import javax.ws.rs.client.ClientRequestContext;

public class ClientCacheRequestFilter implements ClientRequestFilter {
   private Cache cache;

   public ClientCacheRequestFilter(Cache cache) {
      this.cache = cache;
   }

   public void filter(ClientRequestContext ctx) throws IOException {
      if (!ctx.getMethod().equalsIgnoreCase("GET")) return;

      CacheEntry entry = cache.getEntry(request.getUri());
      if (entry == null) return;

      if (!entry.isExpired()) {
         ByteArrayInputStream is = new ByteArrayInputStream(entry.getContent());
         Response response = Response.ok(is)
                                     .type(entry.getContentType()).build();
         ctx.abortWith(response);
         return;
      }

      String etag = entry.getETagHeader();
      String lastModified = entry.getLastModified();

      if (etag != null) {
         ctx.getHeaders.putSingle("If-None-Match", etag);
      }

      if (lastModified != null) {
         ctx.getHeaders.putSingle("If-Modified-Since", lastModified);
      }
   }

}
```


I’ll show you later how to register these client-side filters, but our request filter must be registered as a singleton and constructed with an instance of a **Cache**. I’m not going to go into the details of this **Cache** class, but hopefully you can make an educated guess of how its implemented.



Our **ClientCacheRequestFilter.filter()** method performs a variety of actions based on the state of the underlying cache. First, it checks the **ClientRequestContext** to see if we’re doing an HTTP GET. If not, it just returns and does nothing. Next, we look up the request’s URI in the cache. If there is no entry, again, just return. If there is an entry, we must check to see if it’s expired or not. If it isn’t, we create a Response object that returns a 200, “OK,” status. We populate the **Response** object with the content and **Content-Header** stored in the cache entry and abort the invocation by calling **ClientRequestContext.abortWith()**. Depending on how the application initiated the client invocation, the aborted **Response** object will either be returned directly to the client application, or unmarshalled into the appropriate Java type. If the cache entry has expired, we perform a conditional GET by setting the **If-None-Match** and/or **If-Modified-Since** request headers with values stored in the cache entry.



Now that we’ve seen the request filter, let’s finish this example by implementing the response filter:



```Java
public class CacheResponseFilter implements ClientResponseFilter {
   private Cache cache;

   public CacheResponseFilter(Cache cache) {
      this.cache = cache;
   }

   public void filter(ClientRequestContext request,
                      ClientResponseContext response)
            throws IOException {
      if (!request.getMethod().equalsIgnoreCase("GET")) return;

      if (response.getStatus() == 200) {
         cache.cacheResponse(response, request.getUri());
      } else if (response.getStatus() == 304) {
         CacheEntry entry = cache.getEntry(request.getUri());
         entry.updateCacheHeaders(response);
         response.getHeaders().clear();
         response.setStatus(200);
         response.getHeaders().putSingle("Content-Type", entry.getContentType());
         ByteArrayInputStream is = new ByteArrayInputStream(entry.getContent());
         response.setInputStream(is);
      }
   }
}
```


The **CacheResponseFilter.filter()** method starts off by checking if the invoked request was an HTTP GET. If not, it just returns. If the response status was 200, “OK,” then we ask the Cache object to cache the response for the specific request URI. The **Cache.cacheResponse()** method is responsible for buffering the response and storing relevant response headers and the message body. For brevity’s sake, I’m not going to go into the details of this method. If instead the response code is 304, “Not Modified,” this means that we have performed a successful conditional GET. We update the cache entry with any **ETag** or **Last-Modified** response headers. Also, because the response will have no message body, we must rebuild the response based on the cache entry. We clear all the headers from **ClientResponseContext** and set the appropriate **Content-Type**. Finally we override the response’s **InputStream** with the buffer stored in the cache entry.


