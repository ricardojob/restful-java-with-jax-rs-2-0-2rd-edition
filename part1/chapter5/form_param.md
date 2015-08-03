# @FormParam


The **@javax.ws.rs.FormParam** annotation is used to access **application/x-www-form-urlencoded** request bodies. In other words, it’s used to access individual entries posted by an HTML form document. For example, let’s say we set up a form on our website to register new customers:


```html
<FORM action="http://example.com/customers" method="post">
    <P>
    First name: <INPUT type="text" name="firstname"><BR>
    Last name: <INPUT type="text" name="lastname"><BR>
    <INPUT type="submit" value="Send">
    </P>
 </FORM>
```


We could post this form directly to a JAX-RS backend service described as follows:


```Java
@Path("/customers")
public class CustomerResource {

   @POST
   public void createCustomer(@FormParam("firstname") String first,
                               @FormParam("lastname") String last) {
     ...
   }
}
```


Here, we are injecting **firstname** and **lastname** from the HTML form into the Java parameters **first** and **last**. Form data is URL-encoded when it goes across the wire. When using **@FormParam**, JAX-RS will automatically decode the form entry’s value before injecting it.