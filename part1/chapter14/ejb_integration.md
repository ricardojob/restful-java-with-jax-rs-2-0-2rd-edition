# EJB Integration


EJBs are Java EE components that help you write business logic more easily. They support integration with security, transactions, and persistence. Further explanation of EJB is beyond the scope of this book. I suggest reading the book that I co-wrote with Andrew Rubinger, Enterprise JavaBeans 3.1 (O’Reilly), if you want more information. Java EE requires that EJB containers support integration with JAX-RS. You are allowed to use JAX-RS annotations on local interfaces or no-interface beans of stateless session or singleton beans. No other integration with other bean types is supported.


If you are using the full-scanning deployment mechanism I mentioned before, you can just implement your services and put the classes of your EJBs directly within the WAR, and JAX-RS will find them automatically. Otherwise, you have to return the bean class of each JAX-RS EJB from your Application.getClasses() method. For example, let’s say we have this EJB bean class:


```Java
@Stateless
@Path("/customers")
public class CustomerResourceBean implements CustomerResource {
...
}
```


If you are manually registering your resources via your **Application** class, you must register the bean class of the EJB via the **Application.getClasses()** method. For example:



```Java
package com.restfully.shop.services;

import javax.ws.rs.core.Application;
import javax.ws.rs.ApplicationPath;

@ApplicationPath("/root")
public class ShoppingApplication extends Application {

   public Set<Class<?>> getClasses() {
      HashSet<Class<?>> set = new HashSet<Class<?>>();
      set.add(CustomerResourceBean.class);
      return set;
   }
}
```
