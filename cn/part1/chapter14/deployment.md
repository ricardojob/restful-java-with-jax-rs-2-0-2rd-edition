# Deployment


JAX-RS applications are deployed within a standalone servlet container, like Apache Tomcat, Jetty, JBossWeb, or the servlet container of your favorite application server, like JBoss, Wildfly, Weblogic, Websphere, or Glassfish. Think of a servlet container as a web server. It understands the HTTP protocol and provides a low-level component model (the servlet API) for receiving HTTP requests.


Servlet-based applications are organized in deployment units called Web ARchives (WAR). A WAR is a JAR-based packaging format that contains the Java classes and libraries used by the deployment as well as static content like images and HTML files that the web server will publish. Here’s what the structure of a WAR file looks like:


```
<any static content>
WEB-INF/
        web.xml
        classes/
        lib/
```


Any files outside and above the *WEB-INF/* directory of the archive are published and available directly through HTTP. This is where you would put static HTML files and images that you want to expose to the outside world. The *WEB-INF/* directory has two subdirectories. Within the *classes/* directory, you can put any Java classes you want. They must be in a Java package structure. The *lib/* directory can contain any application or third-party libraries that will be used by the deployment. The *WEB-INF/* directory also contains a *web.xml* deployment descriptor file. This file defines the configuration of the WAR and how the servlet container should initialize it.



You will need to define a *web.xml* file for your JAX-RS deployments. How JAX-RS is deployed within a servlet container varies between JAX-RS-aware (like within Java EE application servers or standalone Servlet 3.x containers like Tomcat) and older JAX-RS–unaware servlet containers. Let’s dive into these details.



### The Application Class


Before looking at what we have to do to configure a *web.xml* file, we need to learn about the **javax.ws.rs.core.Application** class. The **Application** class is the only portable way of telling JAX-RS which web services (**@Path** annotated classes) as well as which filters, interceptors, **MessageBodyReaders**, **MessageBodyWriters**, and **ContextResolvers** (providers) you want deployed. I first introduced you to the **Application** class back in [Chapter 3](../chapter3/deploying_our_service.md):


```Java
package javax.ws.rs.core;

import java.util.Collections;
import java.util.Set;

public abstract class Application {
   private static final Set<Object> emptySet =
                                             Collections.emptySet();

   public abstract Set<Class<?>> getClasses();


   public Set<Object> getSingletons() {
      return emptySet;
   }

}
```


The **Application** class is very simple. All it does is list classes and objects that JAX-RS is supposed to deploy. The **getClasses()** method returns a list of JAX-RS web service and provider classes. JAX-RS web service classes follow the *per-request* model mentioned in [Chapter 3](../chapter3/deploying_our_service.md). Provider classes are instantiated by the JAX-RS container and registered once per application.


The **getSingletons()** method returns a list of preallocated JAX-RS web services and providers. You, as the application programmer, are responsible for creating these objects. The JAX-RS runtime will iterate through the list of objects and register them internally. When these objects are registered, JAX-RS will also inject values for **@Context** annotated fields and setter methods.


Let’s look at a simple example of an **Application** class:



```Java
package com.restfully.shop.services;

import javax.ws.rs.core.Application;

public class ShoppingApplication extends Application {

   public ShoppingApplication() {}

   public Set<Class<?>> getClasses() {
      HashSet<Class<?>> set = new HashSet<Class<?>>();
      set.add(CustomerResource.class);
      set.add(OrderResource.class);
      set.add(ProduceResource.class);
      return set;
   }

   public Set<Object> getSingletons() {

       JsonWriter json = new JsonWriter();
       CreditCardResource service = new CreditCardResource();

       HashSet<Object> set = new HashSet();
       set.add(json);
       set.add(service);
       return set;
   }
}
```


Here, we have a **ShoppingApplication** class that extends the **Application** class. The **getClasses()** method allocates a **HashSet**, populates it with **@Path** annotated classes, and returns the set. The **getSingletons()** method allocates a **MessageBodyWriter** class named **JsonWriter** and an **@Path** annotated class **CreditCardResource**. It then creates a **HashSet** and adds these instances to it. This set is returned by the method.



### Deployment Within a JAX-RS-Aware Container


Java EE stands for Java Enterprise Edition. It is the umbrella specification of JAX-RS and defines a complete enterprise platform that includes services like a servlet container, EJB, transaction manager (JTA), messaging (JMS), connection pooling (JCA), database persistence (JPA), web framework (JSF), and a multitude of other services. Application servers that are certified under Java EE 6 are required to have built-in support for JAX-RS 1.1. Java EE 7 containers are required to have built-in support for JAX-RS 2.0.


For standalone Servlet 3.x containers like Tomcat and Jetty, most JAX-RS implementations can seamlessly integrate JAX-RS just as easily as with Java EE. They do this through the Servlet 3.0 **ServletContainerInitializer** SPI, which we will not cover here. The only difference between standalone servlet deployments and Java EE is that your WAR deployments will also need to include the libraries of your JAX-RS implementation.


Deploying a JAX-RS application is very easy in a JAX-RS-aware servlet container. You still need at least an empty *web.xml* file:



```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
      http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
      version="3.0">
</web-app>
```


If you have at least one **Application** class implementation annotated with **@ApplicationPath**, the JAX-RS–aware container will automatically deploy that **Application**. For example:


```Java
package com.restfully.shop.services;

import javax.ws.rs.core.Application;
import javax.ws.rs.ApplicationPath;

@ApplicationPath("/root")
public class ShoppingApplication extends Application {

   public ShoppingApplication() {}

   public Set<Class<?>> getClasses() {
      HashSet<Class<?>> set = new HashSet<Class<?>>();
      set.add(CustomerResource.class);
      set.add(OrderResource.class);
      set.add(ProduceResource.class);
      return set;
   }

   public Set<Object> getSingletons() {

       JsonWriter json = new JsonWriter();
       CreditCardResource service = new CreditCardResource();

       HashSet<Object> set = new HashSet();
       set.add(json);
       set.add(service);
       return set;
   }
}
```


The **@ApplicationPath** annotation here will set up a base path to whatever the WAR’s context root is, with **root** appended.


You can fully leverage the servlet class scanning abilities if you have both **getClasses()** and **getSingletons()** return an empty set. For example:



```Java
package com.restfully.shop.services;

import javax.ws.rs.core.Application;
import javax.ws.rs.ApplicationPath;

@ApplicationPath("/root")
public class ShoppingApplication extends Application {
   // complete
}
```


When scanning, the application server will look within *WEB-INF/classes* and any JAR file within the *WEB-INF/lib* directory. It will add any class annotated with **@Path** or **@Provider** to the list of things that need to be deployed and registered with the JAX-RS runtime. You can also deploy as many **Application** classes as you want in one WAR. The scanner will also ignore any **Application** classes not annotated with **@ApplicationPath**.


You can also override the **@ApplicationPath** annotation via a simple servlet mapping within *web.xml*:


```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                          http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
      version="3.0">

   <servlet-mapping>
      <servlet-name>com.rest.ShoppingApplication</servlet-name>
      <url-pattern>/*</url-pattern>
   </servlet-mapping>

</web-app>
```


The **servlet-name** is the fully qualified class name of your **Application** class. With this configuration, you can also omit the **@ApplicationPath** annotation entirely.



### Deployment Within a JAX-RS-Unaware Container


If you are running in 2.x or older Servlet containers, you’ll have to manually configure your *web.xml* file to load your JAX-RS implementation’s proprietary servlet class. For example:


```xml
<?xml version="1.0"?>
<web-app>
   <servlet>
      <servlet-name>JAXRS</servlet-name>
      <servlet-class>
         org.jboss.resteasy.plugins.server.servlet.HttpServletDispatcher
      </servlet-class>
      <init-param>
         <param-name>
            javax.ws.rs.Application
         </param-name>
         <param-value>
            com.restfully.shop.services.ShoppingApplication
         </param-value>
      </init-param>
   </servlet>

   <servlet-mapping>
      <servlet-name>JAXRS</servlet-name>
      <url-pattern>/*</url-pattern>
   </servlet-mapping>
</web-app>
```


Here, we’ve registered and initialized the RESTEasy JAX-RS implementation with the **ShoppingApplication** class we created earlier in this chapter. The **&lt;servlet-mapping&gt;** element specifies the base URI path for the JAX-RS runtime. The **/\* &lt;url-pattern&gt;** specifies that all incoming requests should be routed through our JAX-RS implementation.
