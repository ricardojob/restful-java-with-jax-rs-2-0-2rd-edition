# Built-in Content Marshalling


JAX-RS has a bunch of built-in handlers that can marshal to and from a few different specific Java types. While most are low-level conversions, they can still be useful to your JAX-RS classes.


### javax.ws.rs.core.StreamingOutput


We were first introduced to **StreamingOutput** back in [Chapter 3](../chapter3/developing_a_jax_rs_restful_service.md). **StreamingOutput** is a simple callback interface that you implement when you want to do raw streaming of response bodies:


```Java
public interface StreamingOutput  {
      void write(OutputStream output) throws IOException,
                                             WebApplicationException;
}
```


You allocate implemented instances of this interface and return them from your JAX-RS resource methods. When the JAX-RS runtime is ready to write the response body of the message, the **write()** method is invoked on the **StreamingOutput** instance. Let’s look at an example:


```Java
@Path("/myservice")
public class MyService {

   @GET
   @Produces("text/plain")
   StreamingOutput get() {
      return new StreamingOutput() {
         public void write(OutputStream output) throws IOException,
                                           WebApplicationException {
             output.write("hello world".getBytes());
         }
      };
   }
```


Here, we’re getting access to the raw **java.io.OutputStream** through the **write()** method and outputting a simple string to the stream. I like to use an anonymous inner class implementation of the **StreamingOutput** interface rather than creating a separate public class. Since the **StreamingOutput** interface is so tiny, I think it’s beneficial to keep the output logic embedded within the original JAX-RS resource method so that the code is easier to follow. Usually, you’re not going to reuse this logic in other methods, so it doesn’t make much sense to create a specific class.


You may be asking yourself, “Why not just inject an **OutputStream** directly? Why have a callback object to do streaming output?” That’s a good question! The reason for having a callback object is that it gives the JAX-RS implementation freedom to handle output however it wants. For performance reasons, it may sometimes be beneficial for the JAX-RS implementation to use a different thread other than the calling thread to output responses. More importantly, many JAX-RS implementations have an interceptor model that abstracts things out like automatic GZIP encoding or response caching. Streaming directly can usually bypass these architectural constructs. Finally, the Servlet 3.0 specification has introduced the idea of asynchronous responses. The callback model fits in very nicely with the idea of asynchronous HTTP within the Servlet 3.0 specification.



### java.io.InputStream, java.io.Reader


For reading request message bodies, you can use a raw **InputStream** or **Reader** for inputting any media type. For example:


```Java
@Path("/")
public class MyService {

   @PUT
   @Path("/stuff")
   public void putStuff(InputStream is) {
      byte[] bytes = readFromStream(is);
      String input = new String(bytes);
      System.out.println(input);
   }

   private byte[] readFromStream(InputStream stream)
           throws IOException
   {
      ByteArrayOutputStream baos = new ByteArrayOutputStream();

      byte[] buffer = new byte[1000];
      int wasRead = 0;
      do {
         wasRead = stream.read(buffer);
         if (wasRead > 0) {
            baos.write(buffer, 0, wasRead);
         }
      } while (wasRead > −1);
      return baos.toByteArray();
   }
```


Here, we’re reading the full raw bytes of the **java.io.InputStream** available and using them to create a **String** that we output to the screen:


```Java
   @PUT
   @Path("/morestuff")
   public void putMore(Reader reader) {
      LineNumberReader lineReader = new LineNumberReader(reader);
      do {
         String line = lineReader.readLine();
         if (line != null) System.out.println(line);
      } while (line != null);
   }
```


For this example, we’re creating a **java.io.LineNumberReader** that wraps our injected **Reader** object and prints out every line in the request body.


You are not limited to using **InputStream** and **Reader** for reading input request message bodies. You can also return these as response objects. For example:


```Java
@Path("/file")
public class FileService {

   private static final String basePath = "...";
   @GET
   @Path("{filepath: .*}")
   @Produces("text/plain")
   public InputStream getFile(@PathParam("filepath") String path) {
      FileInputStream is = new FileInputStream(basePath + path);
      return is;
   }
```


Here, we’re using an injected **@PathParam** to create a reference to a real file that exists on our disk. We create a **java.io.FileInputStream** based on this path and return it as our response body. The JAX-RS implementation will read from this input stream into a buffer and write it back out incrementally to the response output stream. We must specify the **@Produces** annotation so that the JAX-RS implementation knows how to set the **Content-Type** header.


### java.io.File


Instances of **java.io.File** can also be used for input and output of any media type. Here’s an example for returning a reference to a file on disk:


```Java
@Path("/file")
public class FileService {

   private static final String basePath = "...";
   @GET
   @Path("{filepath: .*}")
   @Produces("text/plain")
   public File getFile(@PathParam("filepath") String path) {
      return new File(basePath + path);
   }
```


In this example, we’re using an injected **@PathParam** to create a reference to a real file that exists on our disk. We create a **java.io.File** based on this path and return it as our response body. The JAX-RS implementation will open up an **InputStream** based on this file reference and stream into a buffer that is written back incrementally to the response’s output stream. We must specify the **@Produces** annotation so that the JAX-RS implementation knows how to set the **Content-Type** header.


You can also inject **java.io.File** instances that represent the incoming request response body. For example:


```Java
   @POST
   @Path("/morestuff")
   public void post(File file) {
      Reader reader = new Reader(new FileInputStream(file));
      LineNumberReader lineReader = new LineNumberReader(reader);
      do {
         String line = lineReader.readLine();
         if (line != null) System.out.println(line);
      } while (line != null);
   }
```


The way this works is that the JAX-RS implementation creates a temporary file for input on disk. It reads from the network buffer and saves the bytes read into this temporary file. In our example, we create a **java.io.FileInputStream** from the **java.io.File** object that was injected by the JAX-RS runtime. We then use this input stream to create a **LineNumberReader** and output the posted data to the console.


### byte[]


A raw array of bytes can be used for the input and output of any media type. Here’s an example:


```Java
@Path("/")
public class MyService {

   @GET
   @Produces("text/plain")
   public byte[] get() {
     return "hello world".getBytes();
   }

   @POST
   @Consumes("text/plain")
   public void post(byte[] bytes) {
      System.out.println(new String(bytes));
   }
}
```


For JAX-RS resource methods that return an array of bytes, you must specify the **@Produces** annotation so that JAX-RS knows what media to use to set the **Content-Type** header.



### String, char[]


Most of the data formats on the Internet are text based. JAX-RS can convert any text-based format to and from either a **String** or an array of characters. For example:


```Java
@Path("/")
public class MyService {

   @GET
   @Produces("application/xml")
   public String get() {
     return "<customer><name>Bill Burke</name></customer>";
   }

   @POST
   @Consumes("text/plain")
   public void post(String str) {
      System.out.println(str);
   }
}
```


For JAX-RS resource methods that return a **String** or an array of characters, you must specify the **@Produces** annotation so that JAX-RS knows what media to use to set the **Content-Type** header.


The JAX-RS specification does require that implementations be sensitive to the character set specified by the **Content-Type** when creating an injected **String**. For example, here’s a client HTTP POST request that is sending some text data to our service:


```xml
POST /data
Content-Type: application/xml;charset=UTF-8

<customer>...</customer>
```


The **Content-Type** of the request is **application/xml**, but it is also stating the character encoding is **UTF-8**. JAX-RS implementations will make sure that the created Java **String** is encoded as **UTF-8** as well.


### MultivaluedMap<String, String> and Form Input


HTML forms are a common way to post data to web servers. Form data is encoded as the **application/x-www-form-urlencoded** media type. In [Chapter 5](../chapter5/jax_rs_injection.md), we saw how you can use the **@FormParam** annotation to inject individual form parameters from the request. You can also inject a **MultivaluedMap&lt;String, String&gt;** that represents all the form data sent with the request. For example:


```Java
@Path("/")
public class MyService {

   @POST
   @Consumes("application/x-www-form-urlencoded")
   @Produces("application/x-www-form-urlencoded")
   public MultivaluedMap<String,String> post(
                              MultivaluedMap<String, String> form) {

      return form;
   }
}
```


Here, our **post()** method accepts POST requests and receives a **MultivaluedMap&lt;String, String&gt;** containing all our form data. You may also return a **MultivaluedMap** of form data as your response. We do this in our example.


The JAX-RS specification does not say whether the injected **MultivaluedMap** should contain encoded strings or not. Most JAX-RS implementations will automatically decode the map’s string keys and values. If you want it encoded, you can use the **@javax.ws.rs.Encoded** annotation to notify the JAX-RS implementation that you want the data in its raw form.



### javax.xml.transform.Source


The **javax.xml.transform.Source** interface represents XML input or output. It is usually used to perform XSLT transformations on input documents. Here’s an example:


```Java
@Path("/transform")
public class TransformationService {

   @POST
   @Consumes("application/xml")
   @Produces("application/xml")
   public String post(Source source) {

      javax.xml.transform.TransformerFactory tFactory =
               javax.xml.transform.TransformerFactory.newInstance();

      javax.xml.transform.Transformer transformer =
           tFactory.newTransformer(
             new javax.xml.transform.stream.StreamSource("foo.xsl"));

      StringWriter writer = new StringWriter();
      transformer.transform(source,
              new javax.xml.transform.stream.StreamResult(writer));

      return writer.toString();
   }
```


In this example, we’re having JAX-RS inject a **javax.xml.transform.Source** instance that represents our request body and we’re transforming it using an XSLT transformation.


Except for JAXB, **javax.xml.transform.Source** is the only XML-based construct that the specification requires implementers to support. I find it a little strange that you can’t automatically inject and marshal **org.w3c.dom.Document** objects. This was probably just forgotten in the writing of the specification.











