# Deploying Our Service


It is easiest to deploy JAX-RS within a Java EE–certified application server (e.g., JBoss) or standalone Servlet 3 container (e.g., Tomcat). Before we can do that, we need to write one simple class that extends **javax.ws.rs.core.Application**. This class tells our application server which JAX-RS components we want to register.


```Java
package javax.ws.rs.core;

import java.util.Collections;
import java.util.Set;

public abstract class Application {
   private static final Set<Object> emptySet = Collections.emptySet();

   public abstract Set<Class<?>> getClasses();


   public Set<Object> getSingletons() {
      return emptySet;
   }

}
```

The **getClasses()** method returns a list of JAX-RS service classes (and providers, but I’ll get to that in [Chapter 6](../chapter6/jax_rs_content_handlers.md)). Any JAX-RS service class returned by this method will follow the per-request model mentioned earlier. When the JAX-RS vendor implementation determines that an HTTP request needs to be delivered to a method of one of these classes, an instance of it will be created for the duration of the request and thrown away. You are delegating the creation of these objects to the JAX-RS runtime.


The **getSingletons()** method returns a list of JAX-RS service objects (and providers, too—again, see [Chapter 6](../chapter6/jax_rs_content_handlers.md)). You, as the application programmer, are responsible for creating and initializing these objects.


These two methods tell the JAX-RS vendor which services you want deployed. Here’s an example:

```Java
package com.restfully.shop.services;

import javax.ws.rs.ApplicationPath;
import javax.ws.rs.core.Application;
import java.util.HashSet;
import java.util.Set;

@ApplicationPath("/services")
public class ShoppingApplication extends Application {

   private Set<Object> singletons = new HashSet<Object>();
   private Set<Class<?>> empty = new HashSet<Class<?>>();

   public ShoppingApplication() {
      singletons.add(new CustomerResource());
   }

   @Override
   public Set<Class<?>> getClasses() {
      return empty;
   }

   @Override
   public Set<Object> getSingletons() {
      return singletons;
   }
}
```

The **@ApplicationPath** defines the relative base URL path for all our JAX-RS services in the deployment. So, in this example, all of our JAX-RS RESTful services will be prefixed with the **/services** path when we execute on them. For our customer service database example, we do not have any per-request services, so our **ShoppingApplication.getClasses()** method returns an empty set. Our **ShoppingApplication.getSingletons()** method returns the **Set** we initialized in the constructor. This **Set** contains an instance of **CustomerResource**.


In Java EE and standalone servlet deployments, JAX-RS classes must be deployed within the application server’s servlet container as a Web ARchive (WAR). Think of a servlet container as your application server’s web server. A WAR is a JAR file that, in addition to Java class files, also contains other Java libraries along with dynamic (like JSPs) and static content (like HTML files or images) that you want to publish on your website. We need to place our Java classes within this archive so that our application server can deploy them. Here’s what the structure of a WAR file looks like:


```xml
<any static content>
WEB-INF/
        web.xml
        classes/
                com/restfully/shop/domain/
                                          Customer.class
                com/restfully/shop/services/
                                            CustomerResource.class
                                            ShoppingApplication.class
```


Our application server’s servlet container publishes everything outside the *WEB-INF/* directory of the archive. This is where you would put static HTML files and images that you want to expose to the outside world. The *WEB-INF/* directory has two subdirectories. Within the *classes/* directory, you can put any Java classes you want. They must be in a Java package structure. This is where we place all of the classes we wrote and compiled in this chapter. The *lib/* directory can contain any third-party JARs we used with our application. Depending on whether your application server has built-in support for JAX-RS or not, you may have to place the JARs for your JAX-RS vendor implementation within this directory. For our customer example, we are not using any third-party libraries, so this *lib/* directory may be empty.


We are almost finished. All we have left to do is to create a *WEB-INF/web.xml* file within our archive.


```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                        http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
      version="3.0">
</web-app>
```


Because this example deploys within a Java EE application server or standalone Servlet 3.x container, all we need is an empty *web.xml* file. The server will detect that an **Application** class is within your WAR and automatically deploy it. Your application is now ready to use!




