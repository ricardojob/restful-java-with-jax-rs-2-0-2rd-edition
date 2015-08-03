# Common Functionality


Each of these injection annotations has a common set of functionality and attributes. Some can automatically be converted from their string representation within an HTTP request into a specific Java type. You can also define default values for an injection parameter when an item does not exist in the request. Finally, you can work with encoded strings directly, rather than having JAX-RS automatically decode values for you. Let’s look into a few of these.


### Automatic Java Type Conversion


All the injection annotations described in this chapter reference various parts of an HTTP request. These parts are represented as a string of characters within the HTTP request. You are not limited to manipulating strings within your Java code, though. JAX-RS can convert this string data into any Java type that you want, provided that it matches one of the following criteria:


1. It is a primitive type. The **int**, **short**, **float**, **double**, **byte**, **char**, and **boolean** types all fit into this category. 
2. It is a Java class that has a constructor with a single **String** parameter. 
3. It is a Java class that has a static method named **valueOf()** that takes a single **String** argument and returns an instance of the class. 
4. It is a **java.util.List&lt;T&gt;**, **java.util.Set&lt;T&gt;**, or **java.util.SortedSet&lt;T&gt;**, where **T** is a type that satisfies criteria 2 or 3 or is a **String**. Examples are **List&lt;Double&gt;**, **Set&lt;String&gt;**, or **SortedSet&lt;Integer&gt;**. 


#### Primitive type conversion


We’ve already seen a few examples of automatic string conversion into a primitive type. Let’s review a simple example again:


```Java
@GET
@Path("{id}")
public String get(@PathParam("id") int id) {...}
```


Here, we’re extracting an integer ID from a string-encoded segment of our incoming request URI.


#### Java object conversion


Besides primitives, this string request data can be converted into a Java object before it is injected into your JAX-RS method parameter. This object’s class must have a constructor or a static method named **valueOf()** that takes a single **String** parameter.


For instance, let’s go back to the **@HeaderParam** example we used earlier in this chapter. In that example, we used **@HeaderParam** to inject a string that represented the **Referer** header. Since **Referer** is a URL, it would be much more interesting to inject it as an instance of **java.net.URL**:


```Java
import java.net.URL;

@Path("/myservice")
public class MyService {

   @GET
   @Produces("text/html")
   public String get(@HeaderParam("Referer") URL referer) {
     ...
   }
}
```


The JAX-RS provider can convert the **Referer** string header into a **java.net.URL** because this class has a constructor that takes only one **String** parameter.


This automatic conversion also works well when only a **valueOf()** method exists within the Java type we want to convert. For instance, let’s revisit the **@MatrixParam** example we used in this chapter. In that example, we used the **@MatrixParam** annotation to inject the color of our vehicle into a parameter of a JAX-RS method. Instead of representing color as a string, let’s define and use a Java enum class:


```Java
public enum Color {
   BLACK,
   BLUE,
   RED,
   WHITE,
   SILVER
}
```


You cannot allocate Java enums at runtime, but they do have a built-in **valueOf()** method that the JAX-RS provider can use:


```Java
public class CarResource {

   @GET
   @Path("/{model}/{year}")
   @Produces("image/jpeg")
   public Jpeg getPicture(@PathParam("make") String make,
                          @PathParam("model") String model,
                          @MatrixParam("color") Color color) {
      ...
   }
```

JAX-RS has made our lives a bit easier, as we can now work with more concrete Java objects rather than doing string conversions ourselves.


#### ParamConverters


Sometimes a parameter class cannot use the default mechanisms to convert from string values. Either the class has no **String** constructor or no **valueOf()** method, or the ones that exist won’t work with your HTTP requests. For this scenario, JAX-RS 2.0 has provided an additional component to help with parameter conversions.


```Java
package javax.ws.rs.ext;

public interface ParamConverter<T> {
    public T fromString(String value);
    public String toString(T value);
}
```


As you can see from the code, **ParamConverter** is a pretty simple interface. The **fromString()** method takes a **String** and converts it to the desired Java type. The **toString()** method does the opposite. Let’s go back to our **Color** example. It pretty much requires full uppercase for all **Color** parameters. Instead, let’s write a **ParamConverter** that allows a **Color** string to be any case.


```Java
public class ColorConverter implements ParamConverter<Color> {

   public Color fromString(String value) {
      if (value.equalsIgnoreCase(BLACK.toString())) return BLACK;
      else if (value.equalsIgnoreCase(BLUE.toString())) return BLUE;
      else if (value.equalsIgnoreCase(RED.toString())) return RED;
      else if (value.equalsIgnoreCase(WHITE.toString())) return WHITE;
      else if (value.equalsIgnoreCase(SILVER.toString())) return SILVER;
      throw new IllegalArgumentException("Invalid color: " + value);
   }

   public String toString(Color value) { return value.toString(); }

}
```


We’re still not done yet. We also have to implement the **ParamConverterProvider** interface.


```Java
package javax.ws.rs.ext;
public interface ParamConverterProvider {
    public <T> ParamConverter<T> getConverter(Class<T> rawType,
                                              Type genericType,
                                              Annotation annotations[]);
}
```


This is basically a factory for **ParamConverters** and is the component that must be scanned or registered with your **Application** deployment class.


```Java
@Provider
public class ColorConverterProvider {

   private final ColorConverter converter = new ColorConverter();

   public  <T> ParamConverter<T> getConverter(Class<T> rawType,
                                              Type genericType,
                                              Annotation[] annotations) {
      if (!rawType.equals(Color.class)) return null;

      return converter;
   }
}
```


In our implementation here, we check to see if the **rawType** is a **Color**. If not, return null. If it is, then return an instance of our **ColorConverter** implementation. The **Annotation[]** parameter for the **getConverter()** method points to whatever parameter annotations are applied to the JAX-RS method parameter you are converting. This allows you to tailor the behavior of your converter based on any additional metadata applied.



#### Collections


All the parameter types described in this chapter may have multiple values for the same named parameter. For instance, let’s revisit the **@QueryParam** example from earlier in this chapter. In that example, we wanted to pull down a set of customers from a customer database. Let’s expand the functionality of this query so that we can order the data sent back by any number of customer attributes:


```
GET /customers?orderBy=last&orderBy=first
```


In this request, the **orderBy** query parameter is repeated twice with different values. We can let our JAX-RS provider represent these two parameters as a **java.util.List** and inject this list with one **@QueryParam** annotation:


```Java
import java.util.List;

@Path("/customers")
public class CustomerResource {

   @GET
   @Produces("application/xml")
   public String getCustomers(
                   @QueryParam("start") int start,
                   @QueryParam("size") int size,
                   @QueryParam("orderBy") List<String> orderBy) {
     ...
   }
}
```


You must define the generic type the **List** will contain; otherwise, JAX-RS won’t know which objects to fill it with.


#### Conversion failures


If the JAX-RS provider fails to convert a string into the Java type specified, it is considered a client error. If this failure happens during the processing of an injection for an **@MatrixParam**, **@QueryParam**, or **@PathParam**, an error status of 404, “Not Found,” is sent back to the client. If the failure happens with **@HeaderParam** or **@CookieParam**, an error response code of 400, “Bad Request,” is sent.



### @DefaultValue


In many types of JAX-RS services, you may have parameters that are optional. When a client does not provide this optional information within the request, JAX-RS will, by default, inject a null value for object types and a zero value for primitive types.


Many times, though, a null or zero value may not work as a default value for your injection. To solve this problem, you can define your own default value for optional parameters by using the **@javax.ws.rs.DefaultValue** annotation.



For instance, let’s look back again at the **@QueryParam** example given earlier in this chapter. In that example, we wanted to pull down a set of customers from a customer database. We used the **start** and **size** query parameters to specify the beginning index and the number of customers desired. While we do want to control the amount of customers sent back as a response, we do not want to require the client to send these query parameters when making a request. We can use **@DefaultValue** to set a base index and dataset size:



```Java
import java.util.List;

@Path("/customers")
public class CustomerResource {

   @GET
   @Produces("application/xml")
   public String getCustomers(@DefaultValue("0") @QueryParam("start") int start,
                           @DefaultValue("10") @QueryParam("size") int size) {
     ...
   }
}
```


Here, we’ve used **@DefaultValue** to specify a default start index of 0 and a default dataset size of **10**. JAX-RS will use the string conversion rules to convert the string value of the **@DefaultValue** annotation into the desired Java type.


### @Encoded


URI template, matrix, query, and form parameters must all be encoded by the HTTP specification. By default, JAX-RS decodes these values before converting them into the desired Java types. Sometimes, though, you may want to work with the raw encoded values. Using the **@javax.ws.rs.Encoded** annotation gives you the desired effect:


```Java
@GET
@Produces("application/xml")
public String get(@Encoded @QueryParam("something") String str) {...}
```


Here, we’ve used the **@Encoded** annotation to specify that we want the encoded value of the **something** query parameter to be injected into the **str** Java parameter. If you want to work solely with encoded values within your Java method or even your entire class, you can annotate the method or class with **@Encoded** and only encoded values will be used.































