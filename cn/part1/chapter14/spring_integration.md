# Spring Integration


Spring is an open source framework similar to EJB. Like EJB, it provides a great abstraction for transactions, persistence, and security. Further explanation of Spring is beyond the scope of this book. If you want more information on it, check out Spring: A Developer’s Notebook by Bruce A. Tate and Justin Gehtland (O’Reilly). Most JAX-RS implementations have their own proprietary support for Spring and allow you to write Spring beans that are JAX-RS web services. If portability is not an issue for you, I suggest that you use the integration with Spring provided by your JAX-RS implementation.


There is a simple, portable way to integrate with Spring that we can talk about in this chapter. What you can do is write an **Application** class that loads your Spring XML files and then registers your Spring beans with JAX-RS through the **getSingletons()** method. First, let’s define a Spring bean that represents a customer database. It will pretty much look like the **CustomerResource** bean described in EJB Integration:


```Java
@Path("/customers")
public interface CustomerResource {

   @GET
   @Produces("application/xml")
   public String getCustomers();

   @GET
   @Produces("application/xml")
   @Path("{id}")
   public String getCustomer(@PathParam("id") int id);
}
```


In this example, we first create an interface for our **CustomerResource** that is annotated with JAX-RS annotations:


```Java
public class CustomerResourceBean implements CustomerResource {

   public String getCustomers() {...}
   public String getCustomer(int id) {...}
}
```


Our Spring bean class, **CustomerResourceBean**, simply implements the **CustomerResource** interface. Although you can opt to not define an interface and use JAX-RS annotations directly on the bean class, I highly suggest that you use an interface. Interfaces work better in Spring when you use features like Spring transactions.


Now that we have a bean class, we should declare it within a Spring XML file called *spring-beans.xml* (or whatever you want to name the file):



```xml
<beans xmlns="http://www.springframework.org/schema/beans"
   <bean id="custService"
         class="com.shopping.restful.services.CustomerResourceBean"/>
</beans>
```


Place this *spring-beans.xml* file within your WAR’s *WEB-INF/classes* directory or within a JAR within the *WEB-INF/lib* directory. For this example, we’ll put it in the *WEB-INF/classes* directory. We will find this file through a class loader resource lookup later on when we write our **Application** class.


Next we write our *web.xml* file:


```xml
<web-app>
    <context-param>
        <param-name>spring-beans-file</param-name>
        <param-value>META-INF/applicationContext.xml</param-value>
    </context-param>
</web-app>
```


In our *web.xml* file, we define a **&lt;context-param&gt;** that contains the classpath location of our Spring XML file. We use a **&lt;context-param&gt;** so that we can change this value in the future if needed. We then need to wire everything together in our **Application** class:



```Java
@ApplicationPath("/")
public class ShoppingApplication extends Application
{
   protected ApplicationContext springContext;

   @Context
   protected ServletContext servletContext;

   public Set<Object> getSingletons()
   {
      try
      {
         InitialContext ctx = new InitialContext();
         String xmlFile = (String)servletContext.getInitParameter
                          ("spring-beans-file");
         springContext = new ClassPathXmlApplicationContext(xmlFile);
      }
      catch (Exception ex)
      {
         throw new RuntimeException(ex);
      }
      HashSet<Object> set = new HashSet();
      set.add(springContext.getBean("customer"));
      return set;
   }

}
```


In this **Application** class, we look up the classpath location of the Spring XML file that we defined in the **&lt;context-param&gt;** of our *web.xml* deployment descriptor. We then load this XML file through Spring’s **ClassPathXmlApplicationContext**. This will also create the beans defined in this file. From the Spring **ApplicationContext**, we look up the bean instance for our **CustomerResource** using the **ApplicationContext.getBean()** method. We then create a **HashSet** and add the **CustomerResource** bean to it and return it to be registered with the JAX-RS runtime.
