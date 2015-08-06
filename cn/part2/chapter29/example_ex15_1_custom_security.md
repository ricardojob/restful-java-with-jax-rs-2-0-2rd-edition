# Example ex15_1: Custom Security


In the first example, we will write two custom security features using JAX-RS filters. The first feature is a custom authentication protocol. The second will be a custom access policy. The example applies these security features to the code we wrote in *ex06_1*.


### One-Time Password Authentication


The first custom security feature we’ll write is one-time password (OTP) authentication. The client will use a credential that changes once per minute. This credential will be a hash that we generate by combining a static password with the current time in minutes. The client will send this generated one-time password in the **Authorization** header. For example:


```
GET /customers HTTP/1.1
Authorization: <username> <generated_password>
```


The header will contain the username of the user followed by the one-time password.


#### The server code


We will enforce OTP authentication only on JAX-RS methods annotated with the **@OTPAuthenticated** annotation:


```Java:src/main/java/com/restfully/shop/features/OTPAuthenticated.java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@NameBinding
public @interface OTPAuthenticated
{
}
```


When declared on a JAX-RS method, this annotation will trigger the binding of a **ContainerRequestFilter** that implements the OTP algorithm using the **@NameBinding** technique discussed in Name Bindings. To apply a name binding, the **OTPAuthenticated** annotation interface is annotated with **@NameBinding**.


With our custom annotation defined, let’s take a look at the filter that implements the OTP algorithm:


```Java:src/main/java/com/restfuly/shop/features/OneTimePasswordAuthenticator.java
@OTPAuthenticated
@Priority(Priorities.AUTHENTICATION)
public class OneTimePasswordAuthenticator implements ContainerRequestFilter
{
```


The **OneTimePasswordAuthenticator** class is annotated with **@OTPAuthenticated**. This completes the **@NameBinding** we started when we implemented the **@OTPAuthenticated** annotation interface. The class is also annotated with **@Priority**. This annotation affects the ordering of filters as they are applied to a JAX-RS method. We’ll discuss specifically why we need this later in the chapter, but you usually want authentication filters to run before any other filter.


```Java
   protected Map<String, String> userSecretMap;

   public OneTimePasswordAuthenticator(Map<String, String> userSecretMap)
   {
      this.userSecretMap = userSecretMap;
   }
```


Our filter will be a singleton object and will be initialized with a map. The key of the map will be a username, while the value will be the secret password used by the user to create a one-time password.


```Java
   @Override
   public void filter(ContainerRequestContext requestContext) throws IOException
   {
      String authorization = requestContext.getHeaderString(
                                                     HttpHeaders.AUTHORIZATION);
      if (authorization == null) throw new NotAuthorizedException("OTP");

      String[] split = authorization.split(" ");
      final String user = split[0];
      String otp = split[1];
```


In the first part of our **filter()** method, we parse the **Authorization** header that was sent by the client. The username and encoded password are extracted from the header into the **user** and **otp** variables.


```Java
      String secret = userSecretMap.get(user);
      if (secret == null) throw new NotAuthorizedException("OTP");

      String regen = OTP.generateToken(secret);
      if (!regen.equals(otp)) throw new NotAuthorizedException("OTP");
```

Next, our **filter()** method looks up the secret of the user in its map and generates its own one-time password. This token is compared to the value sent in the **Authorization** header. If they match, then the user is authenticated. If the user does not exist or the one-time password is not validated, then a 401, “Not Authorized,” response is sent back to the client.


```Java
      final SecurityContext securityContext =
                                   requestContext.getSecurityContext();
      requestContext.setSecurityContext(new SecurityContext()
      {
         @Override
         public Principal getUserPrincipal()
         {
            return new Principal()
            {
               @Override
               public String getName()
               {
                  return user;
               }
            };
         }

         @Override
         public boolean isUserInRole(String role)
         {
            return false;
         }

         @Override
         public boolean isSecure()
         {
            return securityContext.isSecure();
         }

         @Override
         public String getAuthenticationScheme()
         {
            return "OTP";
         }
      });
```


After the user is authenticated, the **filter()** method creates a custom **SecurityContext** implementation within an inner anonymous class. It then overrides the existing **SecurityContext** by calling **ContainerRequestContext.setSecurityContext()**. The **SecurityContext.getUserPrincipal()** is implemented to return a **Principal** initialized with the username sent in the **Authorization** header. Other JAX-RS code can now inject this custom **SecurityContext** to find out who the user principal is.


The algorithm for generating a one-time password is pretty simple. Let’s take a look:


```Java:src/main/java/com/restfully/shop/features/OTP.java
public class OTP
{
   public static String generateToken(String secret)
   {
      long minutes = System.currentTimeMillis() / 1000 / 60;
      String concat = secret + minutes;
      MessageDigest digest = null;
      try
      {
         digest = MessageDigest.getInstance("MD5");
      }
      catch (NoSuchAlgorithmException e)
      {
         throw new IllegalArgumentException(e);
      }
      byte[] hash = digest.digest(concat.getBytes(Charset.forName("UTF-8")));
      return Base64.encodeBytes(hash);
   }
}
```


**OTP** is a simple class. It takes any arbitrary password and combines it with the current time in minutes to generate a new **String** object. An MD5 hash is done on this **String** object. The hash bytes are then Base 64–encoded using a RESTEasy-specific library and returned as a **String**.


The **@OTPAuthenticated** annotation is then applied to two methods in the CustomerResource class to secure access to them:


```Java:src/main/java/com/restfully/shop/services/CustomerResource.java
   @GET
   @Path("{id}")
   @Produces("application/xml")
   @OTPAuthenticated
   public Customer getCustomer(@PathParam("id") int id)
   {
   ...
   }

   @PUT
   @Path("{id}")
   @Consumes("application/xml")
   @OTPAuthenticated
   @AllowedPerDay(1)
   public void updateCustomer(@PathParam("id") int id, Customer update)
   {
   ...
   }
```


The **getCustomer()** and **updateCustomer()** methods are now required to be OTP authenticated.



### Allowed-per-Day Access Policy


The next custom security feature we’ll implement is an allowed-per-day access policy. The idea is that for a certain JAX-RS method, we’ll specify how many times each user is allowed to execute that method per day. We will do this by applying the **@AllowedPerDay** annotation to a JAX-RS method:


```Java:src/main/java/com/restfuly/shop/features/AllowedPerDay.java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@NameBinding
public @interface AllowedPerDay
{
   int value();
}
```


As with **@OTPAuthenticated**, we’ll use a **@NameBinding** to bind the annotation to a specific **ContainerRequestFilter**. Let’s take a look at that filter:



```Java:src/main/java/com/restfuly/shop/features/PerDayAuthorizer.java
@AllowedPerDay(0)
@Priority(Priorities.AUTHORIZATION)
public class PerDayAuthorizer implements ContainerRequestFilter
{
```

The **PerDayAuthorizer** class is annotated with **@AllowedPerDay**. This completes the **@NameBinding** we started when we implemented the **@AllowedPerDay** annotation interface. The class is also annotated with **@Priority**. This annotation affects the ordering of filters as they are applied to a JAX-RS method. We want this filter to run after any authentication code, but before any application code, as we are figuring out whether or not a user is allowed to invoke the request. If we did not annotate the **OneTimePasswordAuthenticator** and **PerDayAuthorizer** classes with the **@Priority** annotation, it is possible that the **PerDayAuthorizer** would be invoked before the **OneTimePasswordAuthenticator** filter. The **PerDayAuthorizer** needs to know the authenticated user created in the **OneTimePasswordAuthenticator** filter; otherwise, it won’t work.


```Java
   @Context
   ResourceInfo info;
```


We inject a **ResourceInfo** instance into the filter instance using the **@Context** annotation. We’ll need this variable to know the current JAX-RS method that is being invoked.



```Java
   public void filter(ContainerRequestContext requestContext) throws IOException
   {
      SecurityContext sc = requestContext.getSecurityContext();
      if (sc == null) throw new ForbiddenException();
      Principal principal = sc.getUserPrincipal();
      if (principal == null) throw new ForbiddenException();
      String user = principal.getName();
```


The **filter()** method first obtains the **SecurityContext** from the **ContainerRequestContext.getSecurityContext()** method. If the context is null or the user principal is null, it returns a 403, “Forbidden,” response to the client by throwing a **ForbiddenException**.


```Java
      if (!authorized(user))
      {
         throw new ForbiddenException();
      }
   }
```


The username value is passed to the **authorized()** method to check the permission. If the method returns false, a 401, “Forbidden,” response is sent back to the client via a **ForbiddenException**.


```Java
   protected static class UserMethodKey
   {
      String username;
      Method method;

      public UserMethodKey(String username, Method method)
      {
         this.username = username;
         this.method = method;
      }

      @Override
      public boolean equals(Object o)
      {
         if (this == o) return true;
         if (o == null || getClass() != o.getClass()) return false;

         UserMethodKey that = (UserMethodKey) o;

         if (!method.equals(that.method)) return false;
         if (!username.equals(that.username)) return false;

         return true;
      }

      @Override
      public int hashCode()
      {
         int result = username.hashCode();
         result = 31 * result + method.hashCode();
         return result;
      }
   }

   protected Map<UserMethodKey, Integer> count =
                             new HashMap<UserMethodKey, Integer>();
```


The filter instance remembers how many times in a day a particular user invoked a particular JAX-RS method. It stores this information in the **count** variable map. This map is keyed by a custom **UserMethodKey** class, which contains the username and JAX-RS method that is being tracked.


```Java
   protected long today = System.currentTimeMillis();

   protected synchronized boolean authorized(String user, AllowedPerDay allowed)
   {
      if (System.currentTimeMillis() > today + (24 * 60 * 60 * 1000))
      {
         today = System.currentTimeMillis();
         count.clear();
      }
```


The **authorized()** method is **synchronized**, as this filter may be concurrently accessed and we need to do this policy check atomically. It first checks to see if a day has elapsed. If so, it resets the **today** variable and clears the **count** map.


```Java
      UserMethodKey key = new UserMethodKey(user, info.getResourceMethod());
      Integer counter = count.get(user);
      if (counter == null)
      {
         counter = 0;
      }
```


The **authorized()** method then checks to see if the current user and method are already being tracked and counted.


```Java
      AllowedPerDay allowed =
      info.getResourceMethod().getAnnotation(AllowedPerDay.class);
      if (allowed.value() > counter)
      {
         count.put(user, counter + 1);
         return true;
      }
      return false;
   }
}
```


The method then extracts the **AllowedPerDay** annotation from the current JAX-RS method that is being invoked. This annotation will contain the number of times per day that a user is allowed to invoke the current JAX-RS method. If this value is greater than the current count for that user for that method, then we update the counter and return true. Otherwise, the policy check has failed and we return false.


We then apply this functionality to a JAX-RS resource method by using the **@AllowedPerDay** annotation:


```Java:src/main/java/com/restfully/shop/services/CustomerResource.java
   @PUT
   @Path("{id}")
   @Consumes("application/xml")
   @OTPAuthenticated
   @AllowedPerDay(1)
   public void updateCustomer(@PathParam("id") int id, Customer update)
   {
   ...
   }
```


A user will now only be able to invoke the **updateCustomer()** method once per day.


The last thing we have to do is initialize our deployment. Our **Application** class needs to change a little bit to enable this:


```Java:src/main/java/com/restfully/shop/services/ShoppingApplication/java
@ApplicationPath("/services")
public class ShoppingApplication extends Application
{
   private Set<Object> singletons = new HashSet<Object>();

   public ShoppingApplication()
   {
      singletons.add(new CustomerResource());
      HashMap<String, String> userSecretMap = new HashMap<String, String>();
      userSecretMap.put("bburke", "geheim");
      singletons.add(new OneTimePasswordAuthenticator(userSecretMap));
      singletons.add(new PerDayAuthorizer());
   }

   @Override
   public Set<Object> getSingletons()
   {
      return singletons;
   }
}
```


The **ShoppingApplication** class populates the user-secret map that must be used to construct the singleton **OneTimePasswordAuthenticator** instance. The **PerDayAuthorizer** class is also a singleton and instantiated by this constructor.



#### The client code


The first thing we do on the client side is to implement a **ClientRequestFilter** that sets up the **Authorization** header that will be sent to the server:



```Java:src/main/java/com/restfully/shop/features/OneTimePasswordGenerator.java
public class OneTimePasswordGenerator implements ClientRequestFilter
{
   protected String user;
   protected String secret;

   public OneTimePasswordGenerator(String user, String secret)
   {
      this.user = user;
      this.secret = secret;
   }

   @Override
   public void filter(ClientRequestContext requestContext) throws IOException
   {
      String otp = OTP.generateToken(secret);
      requestContext.getHeaders().putSingle
      (HttpHeaders.AUTHORIZATION, user + " " + otp);
   }
}
```


This filter is very simple. It is constructed with the username and password we will use to generate the one-time password. The **filter()** method generates the one-time password by calling the **OTP.generateToken()** method we described earlier in this chapter. The **filter()** method then generates and sets the **Authorization** header for the HTTP request.


The client test code is the same as *ex06_1* except that we set it up to use OTP authentication. Let’s take a look:


```Java:src/test/java/com/restfully/shop/test/CustomerResourceTest.java
   @Test
   public void testCustomerResource() throws Exception
   {
      System.out.println("*** Create a new Customer ***");
      Customer newCustomer = new Customer();
      newCustomer.setFirstName("Bill");
      newCustomer.setLastName("Burke");
      newCustomer.setStreet("256 Clarendon Street");
      newCustomer.setCity("Boston");
      newCustomer.setState("MA");
      newCustomer.setZip("02115");
      newCustomer.setCountry("USA");

      Response response = client.target(
              "http://localhost:8080/services/customers")
              .request().post(Entity.xml(newCustomer));
      if (response.getStatus() != 201) throw new RuntimeException
         ("Failed to create");
      String location = response.getLocation().toString();
      System.out.println("Location: " + location);
      response.close();
```


The **testCustomerResource()** method starts off the same way as in *ex06_1*. It creates a customer and obtains its URI from the response. Creating a customer is not authenticated so we do not need to worry about setting up authorization here.


```Java
      System.out.println("*** GET Created Customer **");
      Customer customer = null;
      WebTarget target = client.target(location);
      try
      {
         customer = target.request().get(Customer.class);
         Assert.fail(); // should have thrown an exception
      }
      catch (NotAuthorizedException e)
      {
      }
```


This particular code shows what happens when an unauthenticated request is made. It makes a GET request on the new customer’s URI that fails with a **NotAuthorizedException** because we have not set up our OTP filter yet.


```Java
      target.register(new OneTimePasswordGenerator("bburke", "geheim"));
```


We register an instance of our **OneTimePasswordGenerator** filter initialized with our username and static password. We can now make an authenticated GET request without error.



```Java
      customer = target.request().get(Customer.class);
      System.out.println(customer);
```


To show our allowed-per-day policy in action, the code executes a customer update twice.


```Java
      customer.setFirstName("William");
      response = target.request().put(Entity.xml(customer));
      if (response.getStatus() != 204)
          throw new RuntimeException("Failed to update");
++++
<?hard-pagebreak?>
++++
      // Show the update
      System.out.println("**** After Update ***");
      customer = target.request().get(Customer.class);
      System.out.println(customer);

      // only allowed to update once per day
      customer.setFirstName("Bill");
      response = target.request().put(Entity.xml(customer));
      Assert.assertEquals(Response.Status.FORBIDDEN, response.getStatusInfo());

   }
```


The first invocation succeeds, but the second fails because we are allowed to invoke this method only once per day.




### Build and Run the Example Program


Perform the following steps:

1. Open a command prompt or shell terminal and change to the *ex15_1* directory of the workbook example code.
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md).
3. Perform the build and run the example by typing **maven install**.

