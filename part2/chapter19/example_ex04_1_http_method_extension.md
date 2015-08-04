# Example ex04_1: HTTP Method Extension


This example shows you how your JAX-RS services can consume HTTP methods other than the common standard ones defined in HTTP 1.1. Specifically, the example implements the PATCH method. The PATCH method was originally mentioned in an earlier draft version of the HTTP 1.1 specification:[^21]

> *The PATCH method is similar to PUT except that the entity contains a list of differences between the original version of the resource identified by the Request-URI and the desired content of the resource after the PATCH action has been applied.*



The idea of PATCH is that instead of transmitting the entire representation of a resource to update it, you only have to provide a partial representation in your update request. PUT requires that you transmit the entire representation, so the original plan was to include PATCH for scenarios where sending everything is not optimal.


### Build and Run the Example Program


Perform the following steps:

1. Open a command prompt or shell terminal and change to the *ex04_1* directory of the workbook example code. 
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/installing_resteasy_and_the_examples.md).
3. Perform the build and run the example by typing **mvn install**. 


### The Server Code


Using PATCH within JAX-RS is very simple. The source code under the *ex04_1* directory contains a simple annotation that implements PATCH:


```Java:src/main/java/org/ieft/annotations/PATCH.java
package org.ieft.annotations;

import javax.ws.rs.HttpMethod;
import java.lang.annotation.*;

@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@HttpMethod("PATCH")
public @interface PATCH
{
}
```


As described in [Chapter 4](../../part1/chapter4/subresource_locators.md), all you need to do to use a custom HTTP method is annotate an annotation class with **@javax.ws.rs.HttpMethod**. This **@HttpMethod** declaration must contain the value of the new HTTP method you are defining.


To illustrate the use of our new **@PATCH** annotation, I expanded a little bit on the example code discussed in [Chapter 18](../chapter18/examples_for_chapter_3.md). A simple JAX-RS method is added to the **CustomerResource** class that can handle PATCH requests:



```Java:src/main/java/com/restfully/shop/services/CustomerResource.java
package com.restfully.shop.services;

@Path("/customers")
public class CustomerResource {
...

   @PATCH
   @Path("{id}")
   @Consumes("application/xml")
   public void patchCustomer(@PathParam("id") int id, InputStream is)
   {
      updateCustomer(id, is);
   }
...
}
```


The **@PATCH** annotation is used on the **patchCustomer()** method. The implementation of this method simply delegates to the original *updateCustomer()* method.


### The Client Code


The client code for *ex04_1* is pretty straightforward and similar to *ex03_1*. Let’s look at some initial minor changes we’ve made:


```Java:src/test/java/com/restfully/shop/test/PatchTest.java
package com.restfully.shop.test;

import org.junit.AfterClass;
import org.junit.BeforeClass;
import org.junit.Test;

import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.client.Entity;
import javax.ws.rs.core.Response;


/**
 * @author <a href="mailto:bill@burkecentral.com">Bill Burke</a>
 * @version $Revision: 1 $
 */
public class PatchTest
{
   private static Client client;

   @BeforeClass
   public static void initClient()
   {
      client = ClientBuilder.newClient();
   }

   @AfterClass
   public static void closeClient()
   {
      client.close();
   }
```


First, we initialize our **Client** object within a JUNit **@BeforeClass** block. Any static method you annotate with **@BeforeClass** in JUnit will be executed once before all **@Test** methods are executed. So, in the **initClient()** method we initialize an instance of **Client**. Static methods annotated with **@AfterClass** are executed once after all **@Test** methods have run. The **closeClient()** method cleans up our Client object by invoking **close()** after all tests have run. This is a nice way of putting repetitive initialization and cleanup code that is needed for each test in one place.


The rest of the class is pretty straightforward and similar to *ex03_1*. I’ll highlight only the interesting parts:


```Java
      String patchCustomer = "<customer>"
              + "<first-name>William</first-name>"
              + "</customer>";
      response = client.target(location)
                       .request().method("PATCH", Entity.xml(patchCustomer));
      if (response.getStatus() != 204)
          throw new RuntimeException("Failed to update");
      response.close();
```


To make a **PATCH** HTTP invocation, we use the **javax.ws.rs.client.SyncInvoker.method()** method. The parameters to this method are a string denoting the HTTP method you want to invoke and the entity you want to pass as the message body. Simple as that.



---
[^21] For more information, see http://www.ietf.org/rfc/rfc2068.txt.