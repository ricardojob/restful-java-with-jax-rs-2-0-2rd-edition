# Client and WebTarget


Now that we have a **Client**, there’s a bunch of stuff we can do with this object. Like **ClientBuilder**, the **Client** interface implements **Configurable**. This allows you to change configuration and register components for the **Client** on the fly as your application executes. The most important purpose of **Client**, though, is to create **WebTarget** instances:


```Java
package javax.ws.rs.client.Client;

public interface Client extends Configurable<Client> {

    public void close();

    public WebTarget target(String uri);
    public WebTarget target(URI uri);
    public WebTarget target(UriBuilder uriBuilder);
    public WebTarget target(Link link);

...
}
```


The **WebTarget** interface represents a specific URI you want to invoke on. Through the **Client** interface, you can create a **WebTarget** using one of the **target()** methods:


```Java
package javax.ws.rs.client.Client;

public interface WebTarget extends Configurable<WebTarget> {
    public URI getUri();
    public UriBuilder getUriBuilder();

    public WebTarget path(String path);
    public WebTarget resolveTemplate(String name, Object value);
    public WebTarget resolveTemplate(String name, Object value,
                                      boolean encodeSlashInPath);
    public WebTarget resolveTemplateFromEncoded(String name, Object value);
    public WebTarget resolveTemplates(Map<String, Object> templateValues);
    public WebTarget resolveTemplates(Map<String, Object> templateValues,
                                      boolean encodeSlashInPath);
    public WebTarget resolveTemplatesFromEncoded(
                                      Map<String, Object> templateValues);
    public WebTarget matrixParam(String name, Object... values);
    public WebTarget queryParam(String name, Object... values);

    ...
}
```


**WebTarget** has additional methods to extend the URI you originally constructed it with. You can add path segments or query parameters by invoking **path()** and **queryParam()**. If the **WebTarget** represents a URI template, the **resolveTemplate()** methods can fill in those variables:


```Java
WebTarget target = client.target("http://commerce.com/customers/{id}")
                         .resolveTemplate("id", "123")
                         .queryParam("verbose", true);
```


In this example, we initialized a **WebTarget** with a URI template string. The **resolveTemplate()** method fills in the **id** expression, and we add another query parameter. If you take a look at the **UriBuilder** class, you’ll see that **WebTarget** pretty much mirrors it. Instead of building URIs, though, **WebTarget** is building instances of **WebTargets** that you can use to invoke HTTP requests.


