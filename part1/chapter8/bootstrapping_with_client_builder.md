# Bootstrapping with ClientBuilder


The **javax.ws.rs.client.Client** interface is the main entry point into the JAX-RS Client API. Client instances manage client socket connections and are pretty heavyweight. Instances of this interface should be reused wherever possible, as it can be quite expensive to create and destroy these objects. **Client** objects are created with the **ClientBuilder** class:


```Java
package javax.ws.rs.client;

import java.net.URL;
import java.security.KeyStore;

import javax.ws.rs.core.Configurable;
import javax.ws.rs.core.Configuration;

import javax.net.ssl.HostnameVerifier;
import javax.net.ssl.SSLContext;

public abstract class ClientBuilder implements Configurable<ClientBuilder> {

    public static Client newClient() {...}
    public static Client newClient(final Configuration configuration) {...}

    public static ClientBuilder newBuilder() {...}

    public abstract ClientBuilder sslContext(final SSLContext sslContext);
    public abstract ClientBuilder keyStore(final KeyStore keyStore,
                                             final char[] password);
    public ClientBuilder keyStore(final KeyStore keyStore,
                                             final String password) {}
    public abstract ClientBuilder trustStore(final KeyStore trustStore);
    public abstract ClientBuilder
                        hostnameVerifier(final HostnameVerifier verifier);

    public abstract Client build();
}
```


The easiest way to create a **Client** is to call **ClientBuilder.newClient()**. It instantiates a preinitialized **Client** that you can use right away. To fine-tune the construction of your **Client** interfaces, the **newBuilder()** method creates a **ClientBuilder** instance that allows you to register components and set configuration properties. It inherits these capabilities by implementing the **Configurable** interface:


```Java
package javax.ws.rs.core;

public interface Configurable<C extends Configurable> {
    public C property(String name, Object value);

    public C register(Class<?> componentClass);
    public C register(Object component);

...

}
```


The **ClientBuilder** class also has methods to configure SSL. We’ll cover this in detail in [Chapter 15](../chapter15/securing_jax_rs.md). Let’s take a look at using **ClientBuilder**:


```Java
Client client = ClientBuilder.newBuilder()
                             .property("connection.timeout", 100)
                             .sslContext(sslContext)
                             .register(JacksonJsonProvider.class)
                             .build();
```


We create a **ClientBuilder** instance by calling the static method **ClientBuilder.newBuilder()**. We then set a proprietary, JAX-RS implementation–specific configuration property that controls socket connection timeouts. Next we specify the **sslContext** we want to use to manage **HTTPS** connections. The RESTful services we’re going to interact with are primarily JSON, so we **register()** an **@Provider** that knows how to marshal Java objects to and from JSON. Finally, we call **build()** to create the **Client** instance.


> **Warning**  Always remember to close() your Client objects. Client objects often pool connections for performance reasons. If you do not close them, you are leaking valuable system resources. While most JAX-RS implementations implement a finalize() method for Client, it is not a good idea to rely on the garbage collector to clean up poorly written code.





