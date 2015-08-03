# Per-JAX-RS Method Bindings


On the server side, you can apply a filter or interceptor on a per-JAX-RS-method basis. This allows you to do some really cool things like adding annotation extensions to your JAX-RS container. There are two ways to accomplish this. One is by registering an implementation of the **DynamicFeature** interface. The other is through annotation binding. Let’s look at **DynamicFeature** first.


### DynamicFeature


```Java
package javax.ws.rs.container;

public interface DynamicFeature {
    public void configure(ResourceInfo resourceInfo, FeatureContext context);
}

public interface ResourceInfo {

    /**
     * Get the resource method that is the target of a request,
     * or <code>null</code> if this information is not available.
     *
     * @return resource method instance or null
     * @see #getResourceClass()
     */
    Method getResourceMethod();

    /**
     * Get the resource class that is the target of a request,
     * or <code>null</code> if this information is not available.
     *
     * @return resource class instance or null
     * @see #getResourceMethod()
     */
    Class<?> getResourceClass();
}
```


The **DynamicFeature** interface has one callback method, **configure()**. This **configure()** method is invoked for each and every deployed JAX-RS method. The **ResourceInfo** parameter contains information about the current JAX-RS method being deployed. The **FeatureContext** is an extension of the **Configurable** interface. You’ll use the **register()** methods of this parameter to bind the filters and interceptors you want to assign to this method.



To illustrate how you’d use **DynamicFeature**, let’s expand on the **CacheControlFilter** response filter we wrote earlier in this chapter. The previous incarnation of this class would set the same **Cache-Control** header value for each and every HTTP request. Let’s modify this filter and create a custom annotation called **@MaxAge** that will allow you to set the **max-age** of the **Cache-Control** header per JAX-RS method:



```Java
package com.commerce.MaxAge;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MaxAge {
   int value();
}
```


The modification of the filter looks like this:


```Java
import javax.ws.rs.container.ContainerResponseFilter;
import javax.ws.rs.container.ContainerRequestContext;
import javax.ws.rs.container.ContainerResponseContext;
import javax.ws.rs.core.CacheControl;

public class CacheControlFilter implements ContainerResponseFilter {
   private int maxAge;

   public CacheControlFilter(int maxAge) {
      this.maxAge = maxAge;
   }

   public void filter(ContainerRequestContext req, ContainerResponseContext res)
        throws IOException
   {
      if (req.getMethod().equals("GET")) {
         CacheControl cc = new CacheControl();
         cc.setMaxAge(this.maxAge);
         res.getHeaders().add("Cache-Control", cc);
      }
   }
}
```


The **CacheControlFilter** has a new constructor that has a max age parameter. We’ll use this max age to set the **Cache-Control** header on the response. Notice that we do not annotate **CacheControlFilter** with **@Provider**. Removing **@Provider** will prevent this filter from being picked up on a scan when we deploy our JAX-RS application. Our **DynamicFeature** implementation is going to be responsible for creating and registering this filter:



```Java
import javax.ws.rs.container.DynamicFeature;
import javax.ws.rs.container.ResourceInfo;
import javax.ws.rs.core.FeatureContext;

@Provider
public class MaxAgeFeature implements DynamicFeature {

   public void configure(ResourceInfo ri, FeatureContext ctx) {
      MaxAge max = ri.getResourceMethod().getAnnotation(MaxAge.class);
      if (max == null) return;
      CacheControlFilter filter = new CacheControlFilter(max.value());
      ctx.register(filter);
   }
}
```


The **MaxAgeFeature.configure()** method is invoked for every deployed JAX-RS resource method. The **configure()** method first looks for the **@MaxAge** annotation on the **ResourceInfo**’s method. If it exists, it constructs an instance of the **CacheControlFilter**, passing in the value of the **@MaxAge** annotation. It then registers this created filter with the **FeatureContext** parameter. This filter is now bound to the JAX-RS resource method represented by the **ResourceInfo** parameter. We’ve just created a JAX-RS extension!



### Name Bindings



The other way to bind a filter or interceptor to a particular JAX-RS method is to use the **@NameBinding** meta-annotation:



```Java
package javax.ws.rs;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.ANNOTATION_TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface NameBinding {
}
```


You can bind a filter or interceptor to a particular annotation and when that custom annotation is applied, the filter or interceptor will automatically be bound to the annotated JAX-RS method. Let’s take our previous **BearerTokenFilter** example and bind to a new custom **@TokenAuthenticated** annotation. The first thing we do is define our new annotation:



```Java
import javax.ws.rs.NameBinding;

@NameBinding
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface TokenAuthenticated {}
```


Notice that **@TokenAuthenticated** is annotated with **@NameBinding**. This tells the JAX-RS runtime that this annotation triggers a specific filter or interceptor. Also notice that the **@Target** is set to both methods and classes. To bind the annotation to a specific filter, we’ll need to annotate the filter with it:



```Java
@Provider
@PreMatching
@TokenAuthenticated
public class BearerTokenFilter implements ContainerRequestFilter {
...
}
```


Now, we can use **@TokenAuthenticated** on any method we want and the **BearerTokenFilter** will be bound to that annotated method:


```Java
@Path("/customers")
public class CustomerResource {

   @GET
   @Path("{id}")
   @TokenAuthenticated
   public String getCustomer(@PathParam("id") String id) {...}
}
```


### DynamicFeature Versus @NameBinding



To be honest, I’m not a big fan of **@NameBinding** and lobbied for its removal from early specification drafts. For one, any application of **@NameBinding** can be reimplemented as a **DynamicFeature**. Second, using **@NameBinding** can be pretty inefficient depending on your initialization requirements. For example, let’s reimplement our **@MaxAge** example as an **@NameBinding**. The filter class would need to change as follows:


```Java
import javax.ws.rs.container.ContainerResponseFilter;
import javax.ws.rs.container.ContainerRequestContext;
import javax.ws.rs.container.ContainerResponseContext;
import javax.ws.rs.core.CacheControl;

@MaxAge
@Provider
public class CacheControlFilter implements ContainerResponseFilter {

   @Context ResourceInfo info;

   public void filter(ContainerRequestContext req, ContainerResponseContext res)
        throws IOException
   {
      if (req.getMethod().equals("GET")) {
         MaxAge max = info.getMethod().getAnnotation(MaxAge.class);
         CacheControl cc = new CacheControl();
         cc.setMaxAge(max.value());
         req.getHeaders().add("Cache-Control", cc);
      }
   }
}
```


If we bound **CacheControlFilter** via a name binding, the filter class would have to inject **ResourceInfo**, then look up the **@MaxAge** annotation of the JAX-RS method so it could determine the actual max age value to apply to the **Cache-Control** header. This is less efficient at runtime than our **DynamicFeature** implementation. Sure, in this case the overhead probably will not be noticeable, but if you have more complex initialization scenarios the overhead is bound to become a problem.


