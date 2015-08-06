# Building Links and Link Headers


JAX-RS 2.0 added some support to help you build **Link** headers and to embed links in your XML documents through the Link and **Link.Builder** classes:


```Java
package javax.ws.rs.core;

public abstract class Link {
    public abstract URI getUri();
    public abstract UriBuilder getUriBuilder();
    public abstract String getRel();
    public abstract List<String> getRels();
    public abstract String getTitle();
    public abstract String getType();
    public abstract Map<String, String> getParams();
    public abstract String toString();
}
```


**Link** is an abstract class that represents all the metadata contained in either a **Link** header or Atom link. The **getUri()** method pertains to the href attribute of your Atom link. **getRel()** pertains to the **rel** attribute, and so on. You can also reference any of these attributes as well as any proprietary extension attributes through the **getParams()** method. The **toString()** method will convert the **Link** instance into a **Link** header.



**Link** instances are built through a **Link.Builder**, which is created by one of these methods:



```Java
public abstract class Link {
    public static Builder fromUri(URI uri)
    public static Builder fromUri(String uri)
    public static Builder fromUriBuilder(UriBuilder uriBuilder)
    public static Builder fromLink(Link link)
    public static Builder fromPath(String path)
    public static Builder fromResource(Class<?> resource)
    public static Builder fromMethod(Class<?> resource, String method)
```


All these **fromXXX()** methods work similarly to the **UriBuilder.fromXXX()** methods. They initialize an underlying **UriBuilder** that is used to build the **href** of the link.



The **link()**, **uri()**, and **uriBuilder()** methods allow you to override the underlying URI of the link you are creating:


```Java
public abstract class Link {
   interface Builder {
        public Builder link(Link link);
        public Builder link(String link);
        public Builder uri(URI uri);
        public Builder uri(String uri);
        public Builder uriBuilder(UriBuilder uriBuilder);
...
```


As you can probably guess, the following methods allow you to set various attributes on the link you are building:


```Java
        public Builder rel(String rel);
        public Builder title(String title);
        public Builder type(String type);
        public Builder param(String name, String value);
```


Finally, there’s the **build()** method that will create the link:



```Java
        public Link build(Object... values);
```


The **Link.Builder** has an underlying **UriBuilder**. The **values** passed into the **build()** method are passed along to this **UriBuilder** to create the URI for the **Link**. Let’s look at an example:


```Java
Link link = Link.fromUri("http://{host}/root/customers/{id}")
                .rel("update").type("text/plain")
                .build("localhost", "1234");
```


Calling **toString()** on the **link** instance will result in:


```
<http://localhost/root/customers/1234>; rel="update"; type="text/plain"
```


You can also build relativized links using the **buildRelativized()** method:


```Java
        public Link buildRelativized(URI uri, Object... values);
```


This method will build the **link** instance with a relativized URI based on the underlying URI of the **Link.Builder** and the passed-in **uri** parameter. For example:



```Java
Link link = Link.fromUri("a/d/e")
                .rel("update").type("text/plain")
                .buildRelativized(new URI("a"));
```


The URI is calculated internally like this:


```Java
URI base = new URI("a");
URI supplied = new URI("a/d/e");
URI result = base.relativize(supplied);
```


So, the **String** representation of the **link** variable from the example would be:


```
<d/e>; rel="update"; type="text/plain"
```


You can also use the **baseUri()** methods to specific a base URI to prefix your link’s URI. Take this, for example:



```Java
Link link = Link.fromUri("a/d/e")
                .rel("update").type("text/plain")
                .baseUri("http://localhost/")
                .buildRelativized(new URI("http://localhost/a"));
```


This example code would also output:


```
<d/e>; rel="update"; type="text/plain"
```



### Writing Link Headers



Built **Link** instances can be used to create **Link** headers. Here’s an example:



```Java
@Path
@GET
Response get() {
   Link link = Link.fromUri("a/b/c").build();
   Response response = Response.noContent()
                               .links(link)
                               .build();
   return response;
}
```


Just build your **Link** and add it as a header to your **Response**.



### Embedding Links in XML



The **Link** class also contains a JAXB **XmlAdapter** so that you can embed links within a JAXB class. For example, let’s take our familiar **Customer** domain class and enable it to add one or more embedded links:



```Java
import javax.ws.rs.core.Link;

@XmlRootElement
public class Customer {
   private String name;
   private List<Link> links = new ArrayList<Link>();

      @XmlElement
      public String getName()
      {
         return name;
      }

      public void setName(String name)
      {
         this.name = name;
      }

      @XmlElement(name = "link")
      XmlJavaTypeAdapter(Link.JaxbAdapter.class)
      public List<Link> getLinks()
      {
         return links;
      }
}
```



You can now build any links you want and add them to the **Customer** domain class. They will be converted into XML elements.
