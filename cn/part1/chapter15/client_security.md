# Client Security


The JAX-RS 2.0 specification didn’t do much to define a common client security API. What’s weird is that while it has a stardard API for rarely used protocols like two-way SSL with client certificates, it doesn’t define one for simple protocols like . Instead, you have to rely on the vendor implementation of JAX-RS to provide these security features. For example, the RESTEasy framework provides a **ContainerRequestFilter** you can use to enable Basic Authentication:


```Java
import org.jboss.resteasy.client.jaxrs.BasicAuthentication;

Client client = Client.newClient();
client.register(new BasicAuthentication("username", "password"));
```


You construct the **BasicAuthentication** filter with the username and password you want to authenticate with. That’s it. Other JAX-RS implementations might have other mechanisms for doing this.



JAX-RS 2.0 does have an API for enabling two-way SSL with client certificates. The **ClientBuilder** class allows you to specify a **java.security.KeyStore** that contains the client certificate you want to use to authenticate:



```Java
abstract class ClientBuilder {
   public ClientBuilder keyStore(final KeyStore keyStore, final String password)
}
```


Alternatively, it has methods to create your own **SSLContext**, but creating one is quite complicated and beyond the scope of this book.



### Verifying the Server


HTTPS isn’t only about encrypting your network connection, it is also about establishing trust. One aspect of this on the client side is verifying that the server you are talking to is the actual server you want to talk to and not some middleman on the network that is spoofing it. With most secure Internet servers, you do not have to worry about establishing trust because the server’s certificates are signed by a trusted authority like VeriSign, and your JAX-RS client implementation will know how to verify certificates signed by these authorities.


In some cases, though, especially in test environments, you may be dealing with servers whose certificates are self-signed or signed by an unknown authority. In this case, you must obtain a truststore that contains the server certificates you trust and register them with the Client API. The **ClientBuilder** has a method for this:


```Java
abstract class ClientBuilder {
   public abstract ClientBuilder trustStore(final KeyStore trustStore);
}
```


How you initialize and populate the **KeyStore** is beyond the scope of this book.

