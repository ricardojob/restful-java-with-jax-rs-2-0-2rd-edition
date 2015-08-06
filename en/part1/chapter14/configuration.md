# Configuration


All the examples in this book so far have been simple and pretty self-contained. Your RESTful web services will probably need to sit in front of a database and interact with other local and remote services. Your services will also need configuration settings that are described outside of code. I don’t want to get into too much detail, but the servlet and Java EE specifications provide annotations and XML configurations that allow you to get access to various Java EE services and configuration information. Let’s look at how JAX-RS can take advantage of these features.


### Basic Configuration


Any JAX-RS implementation, whether it sits within a JAX-RS-aware or Java EE container, must support the **@Context** injection of the **javax.servlet.ServletContext** and **javax.servlet.ServletConfig** interfaces. Through these interfaces, you can get access to configuration information expressed in the WAR’s *web.xml* deployment descriptor. Let’s take this *web.xml* file, for example:


```xml
<?xml version="1.0"?>
<web-app>
   <context-param>
      <param-name>max-customers-size</param-name>
      <param-value>10</param-value>
   </context-param>
</web-app>
```


In this *web.xml* file, we want to define a default maximum dataset size for a JAX-RS–based customer database that returns a collection of customers through XML. We do this by defining a **&lt;context-param&gt;** named **max-customers-size** and set the value to 10. We can get access to this value within our JAX-RS service by injecting a reference to **ServletContext** with the **@Context** annotation. For example:


```Java
@Path("/customers")
public class CustomerResource {

   protected int defaultPageSize = 5;

   @Context
   public void setServletContext(ServletContext context) {
       String size = context.getInitParameter("max-customers-size");
       if (size != null) {
           defaultPageSize = Integer.parseInt(size);
       }
   }

   @GET
   @Produces("application/xml")
   public String getCustomerList() {
      ... use defaultPageSize to create
             and return list of XML customers...
   }
}
```


Here, we use the **@Context** annotation on the **setServletContext()** method of our **CustomerResource** class. When an instance of **CustomerResource** gets instantiated, the **setServletContext()* method is called with access to a *javax.servlet.ServletContext*. From this, we can obtain the value of **max-customers-size** that we defined in our *web.xml* and save it in the member variable **defaultPageSize** for later use.


Another way you might want to do this is to use your **javax.ws.rs.core.Application** class as a factory for your JAX-RS services. You could define or pull in configuration information through this class and use it to construct your JAX-RS service. Let’s first rewrite our **CustomerResource** class to illustrate this technique:


```Java
@Path("/customers")
public class CustomerResource {

   protected int defaultPageSize = 5;

   public void setDefaultPageSize(int size) {
       defaultPageSize = size;
   }

   @GET
   @Produces("application/xml")
   public String getCustomerList() {
      ... use defaultPageSize to create and return list of XML customers...
   }
}
```


We first remove all references to the **ServletContext** injection we did in our previous incarnation of the **CustomerResource** class. We replace it with a setter method, **setDefaultPageSize()**, which initializes the **defaultPageSize** member variable. This is a better design for our **CustomerResource** class because we’ve abstracted away how it obtains configuration information. This gives the class more flexibility as it evolves over time.


We then inject the **ServletContext** into our **Application** class and extract the needed information to initialize our services:


```Java
import javax.ws.rs.core.Application;
import javax.naming.InitialContext;

@ApplicationPath("/")
public class ShoppingApplication extends Application {

   public ShoppingApplication() {}

   public Set<Class<?>> getClasses() {
      return Collections.emptySet();
   }

   @Context
   ServletContext servletContext

   public Set<Object> getSingletons() {
       int pageSize = 0;

       try {
          InitialContext ctx = new InitialContext();
          Integer size =
             (Integer)ctx.getInitParameter("max-customers-size");
          pageSize = size.getValue();
       } catch (Exception ex) {
         ... handle example ...
       }
       CustomerResource custService = new CustomerResource();
       custService.setDefaultPageSize(pageSize);

       HashSet<Object> set = new HashSet();
       set.add(custService);
       return set;
   }
}
```
