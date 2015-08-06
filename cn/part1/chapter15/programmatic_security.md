# Programmatic Security


The security features defined in this chapter have so far focused on declarative security metadata, or metadata that is statically defined before an application even runs. JAX-RS also has a small programmatic API for gathering security information about a secured request. Specifically, the **javax.ws.rs.core.SecurityContext** interface has a method for determining the identity of the user making the secured HTTP invocation. It also has a method that allows you to check whether or not the current user belongs to a certain role:



```Java
public interface SecurityContext {

   public Principal getUserPrincipal();
   public boolean isUserInRole(String role);
   public boolean isSecure();
   public String getAuthenticationScheme();
}
```


The **getUserPrincipal()** method returns a standard Java Standard Edition (SE) **javax.security.Principal** security interface. A **Principal** object represents the individual user who is currently invoking the HTTP request. The **isUserInRole()** method allows you to determine whether the current calling user belongs to a certain role. The **isSecure()** method returns true if the current request is a secure connection. The **getAuthenticationScheme()** tells you which authentication mechanism was used to secure the request. **BASIC**, **DIGEST**, **CLIENT_CERT**, and **FORM** are typical values returned by this method. You get access to a **SecurityContext** instance by injecting it into a field, setter method, or resource method parameter using the **@Context** annotation.


Let’s examine this security interface with an example. Let’s say we want to have a security log of all access to a customer database by users who are not administrators. Here is how it might look:


```Java
@Path("/customers")
public class CustomerService {

   @GET
   @Produces("application/xml")
   public Customer[] getCustomers(@Context SecurityContext sec
) {

      if (sec.isSecure() && !sec.isUserInRole("ADMIN")) {
        logger.log(sec.getUserPrincipal() +
                      " accessed customer database.");
      }
      ...
   }
}
```


In this example, we inject the **SecurityContext** as a parameter to our **getCustomer()** JAX-RS resource method. We use the method **SecurityContext.isSecure()** to determine whether or not this is an authenticated request. We then use the method **SecurityContext.isUserInRole()** to find out if the caller is an **ADMIN** or not. Finally, we print out to our audit log.


With the introduction of the filter API in JAX-RS 2.0, you can implement the **SecurityContext** interface and override the current request’s **SecurityContext** via the **ContainerRequestContext.setSecurityContext()** method. What’s interesting about this is that you can implement your own custom security protocols. Here’s an example:



```Java
import javax.ws.rs.container.ContainerRequestContext;
import javax.ws.rs.container.ContainerRequestFilter;
import javax.ws.rs.container.PreMatching;
import javax.ws.rs.core.SecurityContext;
import javax.ws.rs.core.HttpHeaders;

@PreMatching
public class CustomAuth implements ContainerRequestFilter {
   protected MyCustomerProtocolHandler customProtocol = ...;

   public void filter(ContainerRequestContext requestContext) throws IOException
   {
      String authHeader = request.getHeaderString(HttpHeaders.AUTHORIZATION);
      SecurityContext newSecurityContext = customProtocol.validate(authHeader);
      requestContext.setSecurityContext(authHeader);
   }

}
```


This filter leaves out a ton of detail, but hopefully you get the idea. It extracts the **Authorization** header from the request and passes it to the **customProtocol** service that you have written. This returns an implementation of **SecurityContext**. You override the default **SecurityContext** with this variable.

