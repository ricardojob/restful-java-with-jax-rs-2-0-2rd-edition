# Examining the Source Code


The server-side source code is exactly as posted in [Chapter 3](../../part1/chapter3/writing_a_client.md). The guts of the client code are the same as in [Chapter 3](../../part1/chapter3/writing_a_client.md), but the client code is structured as a JUnit class. JUnit is an open source Java library for defining unit tests. Maven automatically knows how to find JUnit-enabled test code and run it with the build. It scans the classes within the *src/test/java directory*, looking for classes that have methods annotated with **@org.junit.Test**. This example has only one: **com.restfully.shop.test.CustomerResourceTest**. Let’s go over the code for it that is different from the book:



```Java:src/test/java/com/restfully/shop/test/CustomerResourceTest.java
package com.restfully.shop.test;

import org.junit.Test;

import javax.ws.rs.client.Client;
import javax.ws.rs.client.ClientBuilder;
import javax.ws.rs.client.Entity;
import javax.ws.rs.core.Response;


/**
 * @author <a href="mailto:bill@burkecentral.com">Bill Burke</a>
 * @version $Revision: 1 $
 */
public class CustomerResourceTest
{
   @Test
   public void testCustomerResource() throws Exception {
```


Our test class has only one method: **testCustomerResource()**. It is annotated with **@Test**. This tells Maven that this method is a JUnit test. The code for this method is exactly the same as the client code in [Chapter 3](../../part1/chapter3/writing_a_client.md). When you run the build, Maven will execute the code within this method to run the example.



That’s it! The rest of the examples in this book have the same Maven structure as *ex03_1* and are tested using JUnit.


