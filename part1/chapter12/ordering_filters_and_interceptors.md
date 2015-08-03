# Ordering Filters and Interceptors


When you have more than one registered filter or interceptor, there may be some sensitivities on the order in which these components are executed. For example, you usually don’t want unauthenticated users executing any of your JAX-RS components. So, if you have a custom authentication filter, you probably want that filter to be executed first. Another example is the combination of our GZIP encoding example with a separate **WriterInterceptor** that encrypts the message body. You probably don’t want to encrypt a GZIP-encoded representation. Instead you’ll want to GZIP-encode an encrypted representation. So ordering is important.


In JAX-RS, filters and interceptors are assigned a numeric priority either through the **@Priority** annotation or via a programmatic interface defined by **Configurable**. The JAX-RS runtime sorts filters and interceptors based on this numeric priority. Smaller numbers are first in the chain:



```Java
package javax.annotation;

public @interface Priority {
   int value();
}
```


The **@Priority** annotation is actually reused from the injection framework that comes with JDK 7. This annotation would be used as follows:


```Java
import javax.annotation.Priority;
import javax.ws.rs.Priorities;

@Provider
@PreMatching
@Priority(Priorities.AUTHENTICATION)
public class BearerTokenFilter implements ContainerRequestFilter {
...
}
```


The **@Priority** annotation can take any numeric value you wish. The **Priorities** class specifies some common constants that you can use when applying the **@Priority** annotation:



```Java
package javax.ws.rs;

public final class Priorities {

    private Priorities() {
        // prevents construction
    }

    /**
     * Security authentication filter/interceptor priority.
     */
    public static final int AUTHENTICATION = 1000;
    /**
     * Security authorization filter/interceptor priority.
     */
    public static final int AUTHORIZATION = 2000;
    /**
     * Header decorator filter/interceptor priority.
     */
    public static final int HEADER_DECORATOR = 3000;
    /**
     * Message encoder or decoder filter/interceptor priority.
     */
    public static final int ENTITY_CODER = 4000;
    /**
     * User-level filter/interceptor priority.
     */
    public static final int USER = 5000;
}
```


If no priority is specified, the default is **USER**, **5000**. There’s a few **Configurable.register()** methods that you can use as an alternative to the **@Priority** annotation to manually assign or override the priority for a filter or interceptor. As mentioned before, the client classes **ClientBuilder**, **Client**, **WebTarget**, and **Invocation.Builder** all implement the **Configurable** interface. Here’s an example of manually setting an interceptor priority using this inherited **Configurable.register()**:


```Java
ClientBuilder builder = ClientBuilder.newBuilder();
builder.register(GZipEncoder.class, Priorities.ENTITY_CODER);
```


On the server side, you can inject an instance of **Configurable** into the constructor of your **Application** class:


```Java
import javax.ws.rs.core.Configurable;

@ApplicationPath("/")
public class MyApplication {

   public MyApplication(@Context Configurable configurable) {
      configurable.register(BearerTokenFilter.class, Priorities.AUTHENTICATION);
   }
}
```


Personally, I prefer using the **@Priority** annotation, as then my filters and interceptors are self-contained. Users can just plug in my components without having to worry about priorities.


