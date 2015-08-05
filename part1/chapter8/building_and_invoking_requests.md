# Building and Invoking Requests


Once you have a **WebTarget** that represents the exact URI you want to invoke on, you can begin building and invoking HTTP requests through one of its **request()** methods:


```Java
public interface WebTarget extends Configurable<WebTarget> {
   ...

       public Invocation.Builder request();
       public Invocation.Builder request(String... acceptedResponseTypes);
       public Invocation.Builder request(MediaType... acceptedResponseTypes);
}
```


The **Invocation.Builder** interface hierarchy is a bit convoluted, so I’ll explain how to build requests using examples and code fragments:


```Java
package javax.ws.rs.client;

public interface Invocation {
...
   public interface Builder extends SyncInvoker, Configurable<Builder> {
        ...
        public Builder accept(String... types);
        public Builder accept(MediaType... types
        public Builder acceptLanguage(Locale... locales);
        public Builder acceptLanguage(String... locales);
        public Builder acceptEncoding(String... encodings);
        public Builder cookie(Cookie cookie);
        public Builder cookie(String name, String value);
        public Builder cacheControl(CacheControl cacheControl);
        public Builder header(String name, Object value);
        public Builder headers(MultivaluedMap<String, Object> headers);
   }
}
```


**Invocation.Builder** has a bunch of methods that allow you to set different types of request headers. The various **acceptXXX()** methods are for content negotiation (see [Chapter 9](../chapter9/language_negotiation.md)). The **cookie()** methods allow you to set HTTP cookies you want to return to the server. And then there are the more generic **header()** and **headers()** methods that cover the more esoteric HTTP headers and any custom ones your application might have.


After setting the headers the request requires, you can then invoke a specific HTTP method to get back a response from the server. GET requests have two flavors:


```Java
    <T> T get(Class<T> responseType);
    <T> T get(GenericType<T> responseType);

    Response get();
```


The first two generic **get()** methods will convert successful HTTP requests to specific Java types. Let’s look at these in action:


```Java
Customer customer = client.target("http://commerce.com/customers/123")
                          .accept("application/json")
                          .get(Customer.class);

List<Customer> customer = client.target("http://commerce.com/customers")
                                .accept("application/xml")
                                .get(new GenericType<List<Customer>>() {});
```


In the first request we want JSON from the server, so we set the **Accept** header with the **accept()** method. We want the JAX-RS client to grab this JSON from the server and convert it to a **Customer** Java type using one of the registered **MessageBodyReader** components.


The second request is a little more complicated. We have a special **MessageBodyReader** that knows how to convert XML into **List&lt;Customer&gt;**. The reader is very sensitive to the generic type of the Java object, so it uses the **javax.ws.rs.core.GenericType** class to obtain information about the type. **GenericType** is a sneaky trick that bypasses Java type erasure to obtain generic type information at runtime. To use it, you create an anonymous inner class that implements **GenericType** and fill in the Java generic type you want to pass information about to the template parameter. I know this is a little weird, but there’s no other way around the Java type system.


> ###### Tip
> **WebTarget** has additional **request()** methods whose parameters take one or more **String** or **MediaType** parameters. These parameters are media types you want to include in an **Accept** header. I think it makes the code more readable if you use the **Invocation.Builder.accept()** method instead. But this generally is a matter of personal preference.


There’s also a **get()** method that returns a **Response** object. This is the same **Response** class that is used on the server side. This gives you more fine-grained control of the HTTP response on the client side. Here’s an example:


```Java
import javax.ws.rs.core.Response;

Response response = client.target("http://commerce.com/customers/123")
                          .accept("application/json")
                          .get();
try {
   if (response.getStatus() == 200) {
      Customer customer = response.readEntity(Customer.class);
   }
} finally {
  response.close();
}
```


In this example, we invoke an HTTP GET to obtain a **Response** object. We check that the status is **OK** and if so, extract a **Customer** object from the returned JSON document by invoking **Response.readEntity()**. The **readEntity()** method matches up the requested Java type and the response content with an appropriate **MessageBodyReader**. This method can be invoked only once unless you buffer the response with the **bufferEntity()** method. For example:


```Java
Response response = client.target("http://commerce.com/customers/123")
                          .accept("application/json")
                          .get();
try {
   if (response.getStatus() == 200) {
      response.bufferEntity();
      Customer customer = response.readEntity(Customer.class);
      Map rawJson = response.readEntity(Map.class);
   }
} finally {
  response.close();
}
```


In this example, the call to **bufferEntity()** allows us to extract the HTTP response content into different Java types, the first type being a **Customer** and the second a **java.util.Map** that represents raw JSON data. If we didn’t buffer the entity, the second **readEntity()** call would result in an **IllegalStateException**.


> **Warning** Always remember to **close()** your **Response** objects. **Response** objects reference open socket streams. If you do not close them, you are leaking system resources. While most JAX-RS implementations implement a **finalize()** method for **Response**, it is not a good idea to rely on the garbage collector to clean up poorly written code. The default behavior of the RESTEasy JAX-RS implementation actually only lets you have one open **Response** per **Client** instance. This forces you to write responsible client code.


So far we haven’t discussed PUT and POST requests that submit a representation to the server. These types of requests have similar method styles to GET but also specify an entity parameter:


```Java
    <T> T put(Entity<?> entity, Class<T> responseType);
    <T> T put(Entity<?> entity, GenericType<T> responseType);
    <T> T post(Entity<?> entity, Class<T> responseType);
    <T> T post(Entity<?> entity, GenericType<T> responseType);

    Response post(Entity<?> entity);
    Response put(Entity<?> entity);
```


The **Entity** class encapsulates the Java object we want to send with the POST or GET request:


```Java
package javax.ws.rs.client;

public final class Entity<T> {
    public Variant getVariant() {}
    public MediaType getMediaType() {
    public String getEncoding() {
    public Locale getLanguage() {
    public T getEntity() {
    public Annotation[] getAnnotations() { }
...
}
```


The **Entity** class does not have a **public** constructor. You instead have to invoke one of the static convenience methods to instantiate one:


```Java
package javax.ws.rs.client;

import javax.ws.rs.core.Form;

public final class Entity<T> {
    public static <T> Entity<T> xml(final T entity) { }
    public static <T> Entity<T> json(final T entity) { }
    public static Entity<Form> form(final Form form) { }
    ...
}
```


The **xml()** method takes a Java object as a parameter. It sets the **MediaType** to **application/xml**. The **json()** method acts similarly, except with JSON. The **form()** method deals with form parameters and **application/x-www-form-urlencoded**, and requires using the **Form** type. There’s a few other helper methods, but for brevity we won’t cover them here.


Let’s look at two different examples that use the POST create pattern to create two different customer resources on the server. One will use JSON, while the other will use form parameters:


```Java
Customer customer = new Customer("Bill", "Burke");
Response response = client.target("http://commerce.com/customers")
                          .request().
                          .post(Entity.json(customer));
response.close();
```


Here we pass in an **Entity** instance to the **post()** method using the **Entity.json()** method. This method will automatically set the **Content-Type** header to **application/json**.


To submit form parameters, we must use the **Form** class:


```Java
package javax.ws.rs.core;

public class Form {
    public Form() { }
    public Form(final String parameterName, final String parameterValue) { }
    public Form(final MultivaluedMap<String, String> store) { }
    public Form param(final String name, final String value) { }
    public MultivaluedMap<String, String> asMap() { }
}
```

This class represents **application/x-www-form-urlencoded** in a request. Here’s an example of it in use:


```Java
Form form = new Form().param("first", "Bill")
                      .param("last", "Burke);
response = client.target("http://commerce.com/customers")
                 .request().
                 .post(Entity.form(form));
response.close();
```


### Invocation


The previous examples are how you’re going to typically interact with the Client API. JAX-RS has an additional invocation model that is slightly different. You can create full **Invocation** objects that represent the entire HTTP request without invoking it. There’s a few additional methods on **Invocation.Builder** that help you do this:


```Java
public interface Invocation {
...
   public interface Builder extends SyncInvoker, Configurable<Builder> {
      Invocation build(String method);
      Invocation build(String method, Entity<?> entity);
      Invocation buildGet();
      Invocation buildDelete();
      Invocation buildPost(Entity<?> entity);
      Invocation buildPut(Entity<?> entity);
      ...
   }
}
```


The **buildXXX()** methods fill in the HTTP method you want to use and finish up building the request by returning an **Invocation** instance. You can then execute the HTTP request by calling one of the **invoke()** methods:


```Java
package javax.ws.rs.client;

public interface Invocation {
    public Response invoke();
    public <T> T invoke(Class<T> responseType);
    public <T> T invoke(GenericType<T> responseType);
...
}
```


So what is the use of this invocation style? For one, the same **Invocation** object can be used for multiple requests. Just prebuild your **Invocation** instances and reuse them as needed. Also, since **invoke()** is a generic method, you could queue up **Invocation** instances or use them with the execute pattern. Let’s see an example:


```Java
Invocation generateReport = client.target("http://commerce.com/orders/report")
                                  .queryParam("start", "now - 5 minutes")
                                  .queryParam("end", "now")
                                  .request()
                                  .accept("application/json")
                                  .buildGet();

while (true) {
   Report report = generateReport.invoke(Report.class);
   renderReport(report);
   Thread.sleep(300000);
}
```


The example code prebuilds a GET **Invocation** that will fetch a JSON report summary of orders made in the last five minutes. We then loop every five minutes and reexecute the invocation. Sure, this example is a little bit contrived, but I think you get the idea.


### Exception Handling


One thing we didn’t discuss is what happens if an error occurs when you use an invocation style that automatically unmarshalls the response. Consider this example:


```Java
Customer customer = client.target("http://commerce.com/customers/123")
                          .accept("application/json")
                          .get(Customer.class);
```


In this scenario, the client framework converts any HTTP error code into one of the exception hierarchy exceptions discussed in Exception Hierarchy. You can then catch these exceptions in your code to handle them appropriately:


```Java
try {
    Customer customer = client.target("http://commerce.com/customers/123")
                              .accept("application/json")
                              .get(Customer.class);
} catch (NotAcceptableException notAcceptable) {
  ...
} catch (NotFoundException notFound) {
  ...
}
```


If the server responds with an HTTP error code not covered by a specific JAX-RS exception, then a general-purpose exception is thrown. **ClientErrorException** covers any error code in the 400s. **ServerErrorException** covers any error code in the 500s.


This invocation style will not automatically handle server redirects—that is, when the server sends one of the HTTP 3xx redirection response codes. Instead, the JAX-RS Client API throws a **RedirectionException** from which you can obtain the **Location** URL to do the redirect yourself. For example:


```Java
WebTarget target = client.target("http://commerce.com/customers/123");
boolean redirected = false;
Customer customer = null;
do {
    try {
        customer = target.accept("application/json")
                              .get(Customer.class);
    } catch (RedirectionException redirect) {
        if (redirected) throw redirect;
        redirected = true;
        target = client.target(redirect.getLocation());
    }

} while (customer == null);
```


In this example, we loop if we receive a redirect from the server. The code makes sure that we allow only one redirect by checking a flag in the catch block. We change the **WebTarget** in the catch block to the **Location** header provided in the server’s response. You might want to massage this code a little bit to handle other error conditions, but hopefully you get the concepts I’m trying to get across.
