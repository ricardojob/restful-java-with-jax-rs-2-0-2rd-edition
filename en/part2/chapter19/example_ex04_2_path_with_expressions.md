# Example ex04_2: @Path with Expressions


For this section, I’ll illustrate the use of an **@Path** annotation with regular expressions. The example is a direct copy of the code in *ex03_1* with a few minor modifications.


### Build and Run the Example Program


Perform the following steps:

1. Open a command prompt or shell terminal and change to the *ex04_2* directory of the workbook example code.
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md).
3. Perform the build and run the example by typing **maven install**.


### The Server Code


The **CustomerResource** class copied from the *ex03_1* example is pretty much the same in *ex04_2*, except that a few of the **@Path** expressions have been modified. I also added an extra method that allows you to reference customers by their first and last names within the URL path:


```Java
@Path("/customers")
public class CustomerResource {
...

   @GET
   @Path("{id : \\d+}")
   @Produces("application/xml")
   public StreamingOutput getCustomer(@PathParam("id") int id)
   {
      ...
   }

   @PUT
   @Path("{id : \\d+}")
   @Consumes("application/xml")
   public void updateCustomer(@PathParam("id") int id, InputStream is)
   {
     ...
   }
```


The **@Path** expression for **getCustomer()** and **updateCustomer()** was changed a little bit to use a Java regular expression for the URI matching. The expression dictates that the id segment of the URI can only be a string of digits. So, **/customers/333** is a legal URI, but **/customers/a32ab** would result in a 404, “Not Found,” response code being returned to the client:


```Java
   @GET
   @Path("{first : [a-zA-Z]+}-{last:[a-zA-Z]+}")
   @Produces("application/xml")
   public StreamingOutput getCustomerFirstLast(
                                   @PathParam("first") String first,
                                   @PathParam("last") String last)
   {
     ...
   }
```


To show a more complex regular expression, I added the **getCustomerFirstLast()** method to the resource class. This method provides a URI pointing to a specific customer, using the customer’s first and last names instead of a numeric ID. This **@Path** expression matches a string of the first name and last name separated by a hyphen character. A legal URI is **/customers/Bill-Burke**. The name can only have letters within it, so **/customers/Bill7-Burke** would result in a 404, “Not Found,” being returned to the client.



### The Client Code


The client code is in *src/test/java/com/restfully/shop/test/ClientResourceTest.java*. It is really not much different than the code in example *ex03_1*, other than the fact that it additionally invokes the URI represented by the **getCustomerFirstLast()** method. If you’ve examined the code from [Chapter 18](../chapter18/examples_for_chapter_3.md), you can probably understand what is going on in this client example, so I won’t elaborate further.