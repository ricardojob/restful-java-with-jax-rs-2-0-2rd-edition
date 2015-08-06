# Example ex04_3: Subresource Locators


The *ex04_3* example implements the subresource locator example shown in Full Dynamic Dispatching in [Chapter 4](../../part1/chapter4/http_method_and_uri_matching.md).


### Build and Run the Example Program


Perform the following steps:


1. Open a command prompt or shell terminal and change to the *ex04_3* directory of the workbook example code.
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md).
3. Perform the build and run the example by typing **maven install**.


### The Server Code


There’s really not much to go over that wasn’t explained in [Chapter 4](../../part1/chapter4/http_method_and_uri_matching.md). 


### The Client Code


The client code lives in *src/test/java/com/restfully/shop/test/CustomerResourceTest.java*:


```Java
public class CustomerResourceTest
{
   @Test
   public void testCustomerResource() throws Exception {
     ...
   }

   @Test
   public void testFirstLastCustomerResource() throws Exception {
     ...
   }
}
```


The code contains two methods: **testCustomerResource()** and **testFirstLastCustomerResource()**.


The **testCustomerResource()** method first performs a POST to **/customers/europe-db** to create a customer using the **CustomerResource** subresource. It then retrieves the created customer using **GET /customers/europe-db/1**.


The **testFirstLastCustomerResource()** method performs a POST to **/customers/northamerica-db** to create a customer using the **FirstLastCustomerResource** subresource. It then uses **GET /customers/northamerica-db/Bill-Burke** to retrieve the created customer.