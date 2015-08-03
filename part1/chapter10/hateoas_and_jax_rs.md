# HATEOAS and JAX-RS


JAX-RS doesn’t have many facilities to help with HATEOAS. HATEOAS is defined by the application, so there’s not much a framework can add. What it does have, though, are helper classes that you can use to build the URIs that you link to in your data formats.



### Building URIs with UriBuilder


One such helper class is **javax.ws.rs.core.UriBuilder**. The **UriBuilder** class allows you to construct a URI piece by piece and is also sensitive to template parameters:


```Java
public abstract class UriBuilder {
   public static UriBuilder fromUri(URI uri)
                                      throws IllegalArgumentException
   public static UriBuilder fromUri(String uri)
                                      throws IllegalArgumentException
   public static UriBuilder fromPath(String path)
                                      throws IllegalArgumentException
   public static UriBuilder fromResource(Class<?> resource)
                                      throws IllegalArgumentException
   public static UriBuilder fromLink(Link link)
                                      throws IllegalArgumentException
```


**UriBuilder** instances can only be instantiated from the static helper methods listed. They can be initialized by a URI path, or the **@Path** annotation of a JAX-RS resource class:


```Java
   public abstract UriBuilder clone();
   public abstract UriBuilder uri(URI uri)
                                      throws IllegalArgumentException;
   public abstract UriBuilder scheme(String scheme)
                                      throws IllegalArgumentException;
   public abstract UriBuilder schemeSpecificPart(String ssp)
                                      throws IllegalArgumentException;

   public abstract UriBuilder userInfo(String ui);
   public abstract UriBuilder host(String host)
                                      throws IllegalArgumentException;
   public abstract UriBuilder port(int port)
                                      throws IllegalArgumentException;
   public abstract UriBuilder replacePath(String path);
   public abstract UriBuilder path(String path)
                                      throws IllegalArgumentException;
   public abstract UriBuilder path(Class resource)
                                      throws IllegalArgumentException;
   public abstract UriBuilder path(Class resource, String method)
                                      throws IllegalArgumentException;
   public abstract UriBuilder path(Method method)
                                      throws IllegalArgumentException;
   public abstract UriBuilder segment(String... segments)
                                      throws IllegalArgumentException;
   public abstract UriBuilder replaceMatrix(String matrix)
                                      throws IllegalArgumentException;
   public abstract UriBuilder matrixParam(String name, Object... vals)
                                      throws IllegalArgumentException;
   public abstract UriBuilder replaceMatrixParam(String name,
                    Object... values) throws IllegalArgumentException;
   public abstract UriBuilder replaceQuery(String query)
                                      throws IllegalArgumentException;
   public abstract UriBuilder queryParam(String name, Object... values)
                                      throws IllegalArgumentException;
   public abstract UriBuilder replaceQueryParam(String name,
                    Object... values) throws IllegalArgumentException;
   public abstract UriBuilder fragment(String fragment);
```


These methods are used to piece together various parts of the URI. You can set the values of a specific part of a URI directly or by using the **@Path** annotation values declared on JAX-RS resource methods. Both string values and **@Path** expressions are allowed to contain template parameters:


```Java
   public abstract URI buildFromMap(Map<String, ? extends Object> values)
                 throws IllegalArgumentException, UriBuilderException;
   public abstract URI buildFromEncodedMap(
                 Map<String, ? extends Object> values)
           throws IllegalArgumentException, UriBuilderException;
   public abstract URI build(Object... values)
           throws IllegalArgumentException, UriBuilderException;
   public abstract URI buildFromEncoded(Object... values)
           throws IllegalArgumentException, UriBuilderException;
}
```


The **build()** methods create the actual URI. Before building the URI, though, any template parameters you have defined must be filled in. The **build()** methods take either a map of name/value pairs that can match up to named template parameters or you can provide a list of values that will replace template parameters as they appear in the templated URI. These values can either be encoded or decoded values, your choice. Let’s look at a few examples:


```Java
UriBuilder builder = UriBuilder.fromPath("/customers/{id}");
builder.scheme("http")
       .host("{hostname}")
       .queryParam("param={param}");
```


In this code block, we have defined a URI pattern that looks like this:


```
http://{hostname}/customers/{id}?param={param}
```


Since we have template parameters, we need to initialize them with values passed to one of the build arguments to create the final URI. If you want to reuse this builder, you should **clone()** it before calling a **build()** method, as the template parameters will be replaced in the internal structure of the object:



```Java
UriBuilder clone = builder.clone();
URI uri = clone.build("example.com", "333", "value");
```


This code would create a URI that looks like this:


```
http://example.com/customers/333?param=value
```


We can also define a map that contains the template values:


```Java
Map<String, Object> map = new HashMap<String, Object>();
map.put("hostname", "example.com");
map.put("id", 333);
map.put("param", "value");

UriBuilder clone = builder.clone();
URI uri = clone.buildFromMap(map);
```


Another interesting example is to create a URI from the **@Path** expressions defined in a JAX-RS annotated class. Here’s an example of a JAX-RS resource class:


```Java
@Path("/customers")
public class CustomerService {

   @Path("{id}")
   public Customer getCustomer(@PathParam("id") int id) {...}
}
```


We can then reference this class and the **getCustomer()** method within our **UriBuilder** initialization to define a new template:


```Java
UriBuilder builder = UriBuilder.fromResource(CustomerService.class);
builder.host("{hostname}")
builder.path(CustomerService.class, "getCustomer");
```


This builder code defines a URI template with a variable hostname and the patterns defined in the **@Path** expressions of the **CustomerService** class and the **getCustomer()** method. The pattern would look like this in the end:


```
http://{hostname}/customers/{id}
```


You can then build a URI from this template using one of the **build()** methods discussed earlier.


There’s also a few peculiarities with this interface. The **build(Object..)** and **build(Map<String, ?>)** methods automatically encode / characters. Take this, for example:


```Java
URI uri = UriBuilder.fromUri("/{id}").build("a/b");
```


This expression would result in:


```
/a%2Fb
```


Oftentimes, you may not want to encode the **/** character. So, two new **build()** methods were introduced in JAX-RS 2.0:


```Java
public abstract URI build(Object[] values, boolean encodeSlashInPath)
            throws IllegalArgumentException, UriBuilderException
public abstract URI buildFromMap(Map<String, ?> values, boolean encodeSlashInPath)
            throws IllegalArgumentException, UriBuilderException
```


If you set the **encodeSlashInPath** to **false**, then the **/** character will not be encoded.


Finally, you may also want to use **UriBuilder** to create template strings. These are often embedded within Atom links. A bunch of new **resolveTemplate()** methods were added to **UriBuilder** in JAX-RS 2.0:


```Java
    public abstract UriBuilder resolveTemplate(String name, Object value);
    public abstract UriBuilder resolveTemplate(String name, Object value,
                                               boolean encodeSlashInPath);
    public abstract UriBuilder resolveTemplateFromEncoded(String name,
                                                          Object value);
    public abstract UriBuilder resolveTemplates(Map<String, Object>
                                                templateValues);
    public abstract UriBuilder resolveTemplates(Map<String, Object>
                                                templateValues, boolean
                                                encodeSlashInPath)
            throws IllegalArgumentException;
    public abstract UriBuilder resolveTemplatesFromEncoded(Map<String, Object>
                                                           templateValues);
```


These work similarly to their **build()** counterparts and are used to partially resolve URI templates. Each of them returns a new **UriBuilder** instance that resolves any of the supplied URI template parameters. You can then use the **toTemplate()** method to obtain the template as a **String**. Here’s an example:


```Java
String original = "http://{host}/{id}";
String newTemplate = UriBuilder.fromUri(original)
                    .resolveTemplate("host", "localhost")
                    .toTemplate();
```


### Relative URIs with UriInfo


When you’re writing services that distribute links, there’s certain information that you cannot know at the time you write your code. Specifically, you will probably not know the hostnames of the links. Also, if you are linking to other JAX-RS services, you may not know the base paths of the URIs, as you may be deployed within a servlet container.


While there are ways to write your applications to get this base URI information from configuration data, JAX-RS provides a cleaner, simpler way through the use of the **javax.ws.rs.core.UriInfo** interface. You were introduced to a few features of this interface in [Chapter 5](../chapter5/jax_rs_injection.md). Besides basic path information, you can also obtain **UriBuilder** instances preinitialized with the base URI used to define all JAX-RS services or the URI used to invoke the current HTTP request:



```Java
public interface UriInfo {
   public URI getRequestUri();
   public UriBuilder getRequestUriBuilder();
   public URI getAbsolutePath();
   public UriBuilder getAbsolutePathBuilder();
   public URI getBaseUri();
   public UriBuilder getBaseUriBuilder();
```



For example, let’s say you have a JAX-RS service that exposes the customers in a customer database. Instead of having a base URI that returns all customers in a document, you want to embed **previous** and **next** links so that you can navigate through subsections of the database (I described an example of this earlier in this chapter). You will want to create these link relations using the URI to invoke the request:


```Java
@Path("/customers")
public class CustomerService {

   @GET
   @Produces("application/xml")
   public String getCustomers(@Context UriInfo uriInfo
) {

      UriBuilder nextLinkBuilder = uriInfo.getAbsolutePathBuilder();
      nextLinkBuilder.queryParam("start", 5);
      nextLinkBuilder.queryParam("size", 10);
      URI next = nextLinkBuilder.build();

      ... set up the rest of the document ...
   }
```


To get access to a **UriInfo** instance that represents the request, we use the **@javax.ws.rs.core.Context** annotation to inject it as a parameter to the JAX-RS resource method **getCustomers()**. Within **getCustomers()**, we call **uriInfo.getAbsolutePathBuilder()** to obtain a preinitialized **UriBuilder**. Depending on how this service was deployed, the URI created might look like this:


```
http://example.com/jaxrs/customers?start=5&size=10
```


**UriInfo** also allows you to relativize a URI based on the current request URI.


```Java
public URI relativize(URI uri);
```


So, for example, if the current request was **http://localhost/root/a/b/c** and you passed **a/d/e** as a parameter to the **relativize()** method, then the returned URI would be **../../d/e**. The **root** segment is the context root of your JAX-RS deployment. Relativization is based off of this root.


You can also resolve URIs with respect to the base URI of your JAX-RS deployment using the **resolve()** method:


```Java
public URI resolve(URI uri);
```


Invoking this method is the same as calling **uriInfo.getBaseURI().resolve(uri)**.


There are other interesting tidbits available for building your URIs. In [Chapter 4](../chapter4/http_method_and_uri_matching.md), I talked about the concept of subresource locators and subresources. Code running within a subresource can obtain partial URIs for each JAX-RS class and method that matches the incoming requests. It can get this information from the following methods on **UriInfo**:


```Java
public interface UriInfo {
...
   public List<String> getMatchedURIs();
   public List<String> getMatchedURIs(boolean decode);
}
```


So, for example, let’s reprint the subresource locator example in [Chapter 4](../chapter4/http_method_and_uri_matching.md):



```Java
@Path("/customers")
public class CustomerDatabaseResource {

   @Path("{database}-db")
   public CustomerResource getDatabase(@PathParam("database") String db) {
      Map map = ...; // find the database based on the db parameter
      return new CustomerResource(map);
   }
}
```


**CustomerDatabaseResource** is the subresource locator. Let’s also reprint the subresource example from [Chapter 4]../chapter4/http_method_and_uri_matching.md) with a minor change using these **getMatchedURIs()** methods:


```Java
public class CustomerResource {
  private Map customerDB;

  public CustomerResource(Map db) {
     this.customerDB = db;
  }

   @GET
   @Path("{id}")
   @Produces("application/xml")
   public StreamingOutput getCustomer(@PathParam("id") int id,
                                       @Context UriInfo uriInfo
) {

      for(String uri : uriInfo.getMatchedURIs()) {
        System.out.println(uri);
      }
     ...
   }
}
```


If the request is **GET http://example.com/customers/usa-db/333**, the output of the **for** loop in the **getCustomer()** method would print out the following:


```
http://example.com/customers
http://example.com/customers/usa-db
http://example.com/customers/usa-db/333
```


The matched URIs correspond to the **@Path** expressions on the following:


* **CustomerDatabaseResource**
* **CustomerDatabaseResource.getDatabase()**
* **CustomerResource.getCustomer()**


Honestly, I had a very hard time coming up with a use case for the **getMatchedURIs()** methods, so I can’t really tell you why you might want to use them.



The final method of this category in **UriInfo** is the **getMatchedResources()** method:


```Java
public interface UriInfo {
...
   public List<Object> getMatchedResources();
}
```


This method returns a list of JAX-RS resource objects that have serviced the request. Let’s modify our **CustomerResource.getCustomer()** method again to illustrate how this method works:


```Java
public class CustomerResource {
  private Map customerDB;

  public CustomerResource(Map db) {
     this.customerDB = db;
  }

   @GET
   @Path("{id}")
   @Produces("application/xml")
   public StreamingOutput getCustomer(@PathParam("id") int id,
                                       @Context UriInfo uriInfo
) {

      for(Object match : uriInfo.getMatchedResources()) {
        System.out.println(match.getClass().getName());
      }
     ...
   }
}
```



The **for** loop in **getCustomer()** prints out the class names of the JAX-RS resource objects that were used to process the request. If the request is **GET http://example.com/customers/usa-db/333**, the output of the **for** loop would be:




```Java
com.acme.CustomerDatabaseResource
com.acme.CustomerResource
```



Again, I’m hard-pressed to find a use case for this method, but it’s in the specification and you should be aware of it.

