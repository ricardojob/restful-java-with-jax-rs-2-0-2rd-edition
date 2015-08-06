# Authentication and Authorization in JAX-RS


To enable authentication, you need to modify the *WEB-INF/web.xml* deployment descriptor of the WAR file your JAX-RS application is deployed in. You enable authorization through XML or by applying annotations to your JAX-RS resource classes. To see how all this is put together, let’s do a simple example. We have a customer database that allows us to create new customers by posting an XML document to the JAX-RS resource located by the **@Path("/customers")** annotation. This service is deployed by a scanned **Application** class annotated with **@ApplicationPath("/services")** so the full URI is **/services/customers**. We want to secure our customer service so that only administrators are allowed to create new customers. Let’s look at a full XML-based implementation of this example:



```xml
<?xml version="1.0"?>
<web-app>
   <security-constraint>
      <web-resource-collection>
         <web-resource-name>customer creation</web-resource-name>
         <url-pattern>/services/customers</url-pattern>
         <http-method>POST</http-method>
      </web-resource-collection>
      <auth-constraint>
         <role-name>admin</role-name>
      </auth-constraint>
    </security-constraint>

    <login-config>
        <auth-method>BASIC</auth-method>
        <realm-name>jaxrs</realm-name>
    </login-config>

    <security-role>
        <role-name>admin</role-name>
    </security-role>

</web-app>
```


The **&lt;login-config&gt;** element defines how we want our HTTP requests to be authenticated for our entire deployment. The **&lt;auth-method&gt;** subelement can be **BASIC**, **DIGEST**, or **CLIENT_CERT**. These values correspond to Basic, Digest, and Client Certificate Authentication, respectively.



The **&lt;login-config&gt;** element doesn’t turn on authentication. By default, any client can access any URL provided by your web application with no constraints. To enforce authentication, you must specify a URL pattern you want to secure. In our example, we use the **&lt;url-pattern&gt;** element to specify that we want to secure the **/services/customers** URL. The **&lt;http-method&gt;** element says that we only want to secure POST requests to this URL. If we leave out the **&lt;http-method&gt;** element, all HTTP methods are secured. In our example, we only want to secure POST requests, so we must define the **&lt;http-method&gt;** element.


Next, we have to specify which roles are allowed to POST to **/services/customers**. In the *web.xml* file example, we define an **&lt;auth-constraint&gt;** element within a **&lt;security-constraint&gt;**. This element has one or more **&lt;role-name&gt;** elements that define which roles are allowed to access the defined constraint. In our example, applying this XML only gives the admin role permission to access the **/services/customers** URL.


If you set a **&lt;role-name&gt;** of **\*** instead, any user would be able to access the constrained URL. Authentication with a valid user would still be required, though. In other words, a **&lt;role-name&gt;** of **\*** means anybody who is able to log in can access the resource.


Finally, there’s an additional bit of syntactic sugar we need to specify in *web.xml*. For every **&lt;role-name&gt;** we use in our **&lt;auth-constraints&gt;** declarations, we must define a corresponding **&lt;security-role&gt;** in the deployment descriptor.


There is a minor limitation when you’re declaring **&lt;security-constraints&gt;** for JAX-RS resources. The **&lt;url-pattern&gt;** element does not have as rich an expression syntax as JAX-RS **@Path** annotation values. In fact, it is extremely limited. It supports only simple wildcard matches via the **\*** character. No regular expressions are supported. For example:

* /\*
* /foo/\*
* \*.txt


The wildcard pattern can only be used at the end of a URL pattern or to match file extensions. When used at the end of a URL pattern, the wildcard matches every character in the incoming URL. For example, **/foo/\*** would match any URL that starts with **/foo**. To match file extensions, you use the format **\*.&lt;suffix&gt;**. For example, the **\*.txt** pattern matches any URL that ends with **.txt**. No other uses of the wildcard character are permitted in URL patterns. For example, here are some illegal expressions:

* /foo/\*/bar
* /foo/\*.txt



### Enforcing Encryption


By default, the servlet specification will not require access over HTTPS to any user constraints you declare in your *web.xml* file. If you want to enforce HTTPS access for these constraints, you can specify a **&lt;user-data-constraint&gt;** within your **&lt;security-constraint&gt;** definitions. Let’s modify our previous example to enforce HTTPS:


```xml
<web-app>
...

   <security-constraint>
      <web-resource-collection>
         <web-resource-name>customer creation</web-resource-name>
         <url-pattern>/services/customers</url-pattern>
         <http-method>POST</http-method>
      </web-resource-collection>
      <auth-constraint>
         <role-name>admin</role-name>
      </auth-constraint>
      <user-data-constraint>
         <transport-guarantee>CONFIDENTIAL</transport-guarantee>
      </user-data-constraint>
    </security-constraint>
...
</web-app>
```


All you have to do is declare a **&lt;transport-guarantee&gt;** element within a **&lt;user-data-constraint&gt;** that has a value of **CONFIDENTIAL**. If a user tries to access the URL pattern with HTTP, she will be redirected to an HTTPS-based URL.



### Authorization Annotations


Java EE defines a common set of annotations that can define authorization metadata. The JAX-RS specification suggests, but does not require, vendor implementations to support these annotations in a non–Java EE 6 environment. These annotations live in the **javax.annotation.security** package and are **@RolesAllowed**, **@DenyAll**, **@PermitAll**, and **@RunAs**.


The **@RolesAllowed** annotation defines the roles permitted to execute a specific operation. When placed on a JAX-RS annotated class, it defines the default access control list for all HTTP operations defined in the JAX-RS class. If placed on a JAX-RS method, the constraint applies only to the method that is annotated.


The **@PermitAll** annotation specifies that any authenticated user is permitted to invoke your operation. As with **@RolesAllowed**, you can use this annotation on the class to define the default for the entire class or you can use it on a per-method basis. Let’s look at an example:


```Java
@Path("/customers")
@RolesAllowed({"ADMIN", "CUSTOMER"})
public class CustomerResource {

   @GET
   @Path("{id}")
   @Produces("application/xml")
   public Customer getCustomer(@PathParam("id") int id) {...}

   @RolesAllowed("ADMIN")
   @POST
   @Consumes("application/xml")
   public void createCustomer(Customer cust) {...}

   @PermitAll
   @GET
   @Produces("application/xml")
   public Customer[] getCustomers() {}
}
```


Our **CustomerResource** class is annotated with **@RolesAllowed** to specify that, by default, only **ADMIN** and **CUSTOMER** users can execute HTTP operations and paths defined in that class. The **getCustomer()** method is not annotated with any security annotations, so it inherits this default behavior. The **createCustomer()** method is annotated with **@RolesAllowed** to override the default behavior. For this method, we only want to allow **ADMIN** access. The **getCustomers()** method is annotated with **@PermitAll**. This overrides the default behavior so that any authenticated user can access that URI and operation.


In practice, I don’t like to specify security metadata using annotations. Security generally does not affect the behavior of the business logic being executed and falls more under the domain of configuration. Administrators may want to add or remove role constraints periodically. You don’t want to have to recompile your whole application when they want to make a simple change. So, if I can avoid it, I usually use *web.xml* to define my authorization metadata.



There are some advantages to using annotations, though. For one, it is a workaround for doing fine-grained constraints that are just not possible in *web.xml* because of the limited expression capabilities of **&lt;url-pattern&gt;**. Also, because you can apply constraints per method using these annotations, you can fine-tune authorization per media type. For example:


```Java
@Path("/customers")
public class CustomerService {

   @GET
   @Produces("application/xml")
   @RolesAllowed("XML-USERS")
   public Customer getXmlCustomers() {}


   @GET
   @Produces("application/json")
   @RolesAllowed("JSON-USERS")
   public Customer getJsonCustomers() {}
}
```


Here we only allow **XML-USERS** to obtain **application/xml** content and **JSON-USERS** to obtain **application/json** content. This might be useful for limiting users in getting data formats that are expensive to create.
