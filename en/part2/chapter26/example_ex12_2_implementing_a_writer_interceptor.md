# Example ex12_2: Implementing a WriterInterceptor


In this example, we implement support for generating the **Content-MD5** header. This header is defined in the HTTP 1.1 specification. Its purpose is to provide an additional end-to-end message integrity check of the HTTP message body. While not proof against malicious attacks, it’s a good way to detect accidental modification of the message body in transit just in case it was transformed by a proxy, cache, or some other intermediary. Well, OK, I admit it’s a pretty lame header, but let’s show how we can implement support for it using a **WriterInterceptor**:



```Java
public class ContentMD5Writer implements WriterInterceptor
{
   @Override
   public void aroundWriteTo(WriterInterceptorContext context)
                      throws IOException, WebApplicationException
   {
      MessageDigest digest = null;
      try
      {
         digest = MessageDigest.getInstance("MD5");
      }
      catch (NoSuchAlgorithmException e)
      {
         throw new IllegalArgumentException(e);
      }
```


To implement a **WriterInterceptor**, we must define an **aroundWriteTo()** method. We start off in this method by creating a **java.security.MessageDigest**. We’ll use this class to create an MD5 hash of the entity we’re marshalling.



```Java
      ByteArrayOutputStream buffer = new ByteArrayOutputStream();
      DigestOutputStream digestStream = new DigestOutputStream(buffer, digest);

      OutputStream old = context.getOutputStream();
      context.setOutputStream(digestStream);
```


Next we create a **java.io.ByteArrayOutputStream** and wrap it with a **java.security.DigestOutputStream**. The MD5 hash is created from the marshalled bytes of the entity. We need to buffer this marshalling in memory, as we need to set the **Content-MD5** before the entity is sent across the wire. We override the **OutputStream** of the **ContainerRequestContext** so that the **MessageBodyWriter** that performs the marshalling uses the **DigestOutputStream**.


```Java
      try
      {
         context.proceed();

         byte[] hash = digest.digest();
         String encodedHash = Base64.encodeBytes(hash);
         context.getHeaders().putSingle("Content-MD5", encodedHash);
```


Next, **context.proceed()** is invoked. This continues with the interceptor chain and until the underlying **MessageBodyWriter** is invoked. After **proceed()** finishes, we obtain the hash from the **MessageDigest** and Base-64–encode it using a RESTEasy utility class. We then set the **Content-MD5** header value with this encoded string.


```Java
         byte[] content = buffer.toByteArray();
         old.write(content);
      }
```


After the header is set, we write the buffered content to the real **OutputStream**.


```Java
      finally
      {
         context.setOutputStream(old);
      }
   }
}
```


Finally, if you override the context’s **OutputStream** it is always best practice to revert it after you finish intercepting. We do this in the **finally** block.


We enable this interceptor for all requests that return an entity by registering it within our **Application** class. I won’t go over this code, as you should be familiar with how to do this by now.



### The Client Code


The client code is basically the same as *ex12_1* except we are viewing the returned **Content-MD5** header:



```Java:src/test/java/com/restfully/shop/test/CustomerResourceTest.java
   @Test
   public void testCustomerResource() throws Exception
   {
...
      System.out.println("*** GET Created Customer **");
      response = client.target(location).request().get();
      String md5 = response.getHeaderString("Content-MD5");
      System.out.println("Content-MD5: " + md5);
   }
```


### Build and Run the Example Program


Perform the following steps:

1. Open a command prompt or shell terminal and change to the *ex12_2* directory of the workbook example code. 
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md). 
3. Perform the build and run the example by typing **maven install**.