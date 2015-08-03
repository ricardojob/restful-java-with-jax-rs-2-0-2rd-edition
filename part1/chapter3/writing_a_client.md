# Writing a Client


If you need to interact with a remote RESTful service like we just created, you can use the JAX-RS 2.0 Client API. The **Client** interface is responsible for managing client HTTP connections. I discuss the Client API in more detail in [Chapter 8](../chapter8/client_introduction.md), but let’s look at how we might create a customer by invoking the remote services defined earlier in this chapter.


```Java
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.client.Client;
import javax.ws.rs.client.Entity;
import javax.ws.rs.core.Response;

public class MyClient {
   public static void main(String[] args) throws Exception {
      Client client = ClientBuilder.newClient();
      try {
         System.out.println("*** Create a new Customer ***");

         String xml = "<customer>"
                 + "<first-name>Bill</first-name>"
                 + "<last-name>Burke</last-name>"
                 + "<street>256 Clarendon Street</street>"
                 + "<city>Boston</city>"
                 + "<state>MA</state>"
                 + "<zip>02115</zip>"
                 + "<country>USA</country>"
                 + "</customer>";

         Response response = client.target(
                 "http://localhost:8080/services/customers")
                 .request().post(Entity.xml(xml));
         if (response.getStatus() != 201) throw new RuntimeException(
                 "Failed to create");
         String location = response.getLocation().toString();
         System.out.println("Location: " + location);
         response.close();

         System.out.println("*** GET Created Customer **");
         String customer = client.target(location).request().get(String.class);
         System.out.println(customer);

         String updateCustomer = "<customer>"
                 + "<first-name>William</first-name>"
                 + "<last-name>Burke</last-name>"
                 + "<street>256 Clarendon Street</street>"
                 + "<city>Boston</city>"
                 + "<state>MA</state>"
                 + "<zip>02115</zip>"
                 + "<country>USA</country>"
                 + "</customer>";
         response = client.target(location)
                          .request()
                          .put(Entity.xml(updateCustomer));
         if (response.getStatus() != 204)
            throw new RuntimeException("Failed to update");
         response.close();
         System.out.println("**** After Update ***");
         customer = client.target(location).request().get(String.class);
         System.out.println(customer);
      } finally {
         client.close();
      }
   }
}
```


The Client API is a fluent API in that it tries to look like a domain-specific language (DSL). The Client API has a lot of method chaining, so writing client code can be as simple and compact as possible. In the preceding example, we first build and execute a POST request to create a customer. We then extract the URI of the created customer from a **Response** object to execute a GET request on the URI. After this, we update the customer with a new XML representation by invoking a PUT request. The example only uses **Strings**, but we’ll see in [Chapter 6](../chapter6/jax_rs_content_handlers.md) that JAX-RS also has content handlers you can use to marshal your Java objects automatically to and from XML and other message formats.


