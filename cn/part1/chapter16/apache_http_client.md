# Apache HttpClient


The Apache foundation has written a nice, extensible, HTTP client library called HttpClient.[^20] It is currently on version 4.x as of the writing of this book. Although it is not JAX-RS–aware, it does have facilities for preemptive authentication and APIs for dealing with a few different media types like forms and multipart. Some of its other features are a full interceptor model, automatic cookie handling between requests, and pluggable authentication. Let’s look at a simple example:


```Java
import org.apache.http.*;
import org.apache.http.client.*;

public class MyClient {

  public static void main(String[] args) throws Exception {

      DefaultHttpClient client = new DefaultHttpClient();
      HttpGet get = new HttpGet("http://example.com/customers/1");
      get.addHeader("accept", "application/xml");

      HttpResponse response = client.execute(get);
      if (response.getStatusLine().getStatusCode() != 200) {
         throw new RuntimeException("Operation failed: " +
                   response.getStatusLine().getStatusCode());
      }

      System.out.println("Content-Type: " +
           response.getEntity().getContentType().getValue());

      BufferedReader reader = new BufferedReader(new
               InputStreamReader(response.getEntity()
                                         .getInputStream()));

      String line = reader.readLine();
      while (line != null) {
         System.out.println(line);
         line = reader.readLine();
      }
      client.getConnectionManager().shutdown();
   }
}
```


In Apache HttpClient 4.x, the **org.apache.http.impl.client.DefaultHttpClient** class is responsible for managing HTTP connections. It handles the default authentication settings, and pools and manages persistent HTTP connections (keepalive) and any other default configuration settings. It is also responsible for executing requests. The **org.apache.http.client.methods.HttpGet** class is used to build an actual HTTP GET request. You initialize it with a URL and set any request headers you want using the **HttpGet.addHeader()** method. There are similar classes in this package for doing POST, PUT, and DELETE invocations. Once you have built your request, you execute it by calling **DefaultHttpClient.execute()**, passing in the request you built. This returns an **org.apache.http.HttpResponse** object. To get the response code from this object, execute **HttpResponse.getStatusLine().getStatusCode()**. The **HttpResponse.getEntity()** method returns an **org.apache.http.HttpEntity** object, which represents the message body of the response. From it, you can get the **Content-Type** by executing **HttpEntity.getContentType()** as well as a **java.io.InputStream** so you can read the response. When you are done invoking requests, you clean up your connections by calling **HttpClient.getConnectionManager().shutdown()**.


To push data to the server via a POST or PUT operation, you need to encapsulate your data within an instance of the **org.apache.http.HttpEntity** interface. The framework has some simple prebuilt ones for sending strings, forms, byte arrays, and input streams. Let’s look at sending some XML.


In this example, we want to create a customer in a RESTful customer database. The API works by POSTing an XML representation of the new customer to a specific URI. A successful response is 201, “Created.” Also, a **Location** response header is returned that points to the newly created customer:



```Java
import org.apache.http.*;
import org.apache.http.client.*;
import org.apache.impl.client.*;

public class MyClient {

  public static void main(String[] args) throws Exception {

      DefaultHttpClient client = new DefaultHttpClient();
      HttpPost post = new HttpPost("http://example.com/customers");
      StringEntity entity = new StringEntity("<customer id='333'/>");
      entity.setContentType("application/xml");
      post.setEntity(entity);
      HttpClientParams.setRedirection(post.getParams(), false);
      HttpResponse response = client.execute(post);
      if (response.getStatusLine().getStatusCode() != 201) {
         throw new RuntimeException("Operation failed: " +
                   response.getStatusLine().getStatusCode());
      }

      String location = response.getLastHeader("Location")
                                 .getValue();

      System.out.println("Object created at: " + location);
      System.out.println("Content-Type: " +
           response.getEntity().getContentType().getValue());

      BufferedReader reader = new BufferedReader(new
           InputStreamReader(response.getEntity().getContent()));

      String line = reader.readLine();
      while (line != null) {
         System.out.println(line);
         line = reader.readLine();
      }
      client.getConnectionManager().shutdown();
   }
}
```


We create an **org.apache.http.entity.StringEntity** to encapsulate the XML we want to send across the wire. We set its **Content-Type** by calling **StringEntity.setContentType()**. We add the entity to the request by calling **HttpPost.setEntity()**. Since we are expecting a redirection header with our response and we do not want to be automatically redirected, we must configure the request to not do automatic redirects. We do this by calling **HttpClientParams.setRedirection()**. We execute the request the same way we did with our GET example. We get the **Location** header by calling **HttpResponse.getLastHeader()**.


### Authentication


The Apache HttpClient 4.x supports Basic, Digest, and Client Certificate Authentication. Basic and Digest Authentication are done through the **DefaultHttpClient.getCredentialsProvider().setCredentials()** method. Here’s an example:


```Java
DefaultHttpClient client = new DefaultHttpClient();
client.getCredentialsProvider().setCredentials(
    new AuthScope("example.com", 443),
    new UsernamePasswordCredentials("bill", "geheim");
);
```


The **org.apache.http.auth.AuthScope** class defines the server and port that you want to associate with a username and password. The **org.apache.http.auth.UsernamePasswordCredentials** class encapsulates the username and password into an object. You can call **setCredentials()** for every domain you need to communicate with securely.


Apache HttpClient, by default, does not do preemptive authentication for the Basic and Digest protocols, but does support it. Since the code to do this is a bit verbose, we won’t cover it in this book.



#### Client Certificate authentication



Apache HttpClient also supports Client Certificate Authentication. As with **HttpsURLConnection**, you have to load in a **KeyStore** that contains your client certificates. The section [java.net.URL](../chapter16/java_net_URL.md) describes how to do this. You initialize an **org.apache.http.conn.ssl.SSLSocketFactory** with a loaded **KeyStore** and associate it with the **DefaultHttpClient**. Here is an example of doing this:


```Java
import java.io.File;
import java.io.FileInputStream;
import java.security.KeyStore;

import org.apache.http.*;
import org.apache.http.HttpResponse;
import org.apache.http.client.methods.*;
import org.apache.http.conn.scheme.*;
import org.apache.http.conn.ssl.*;
import org.apache.http.impl.client.DefaultHttpClient;

public class MyClient {

   public final static void main(String[] args) throws Exception {
      DefaultHttpClient client = new DefaultHttpClient();

      KeyStore trustStore  = KeyStore.getInstance(
                                 KeyStore.getDefaultType());
      FileInputStream instream = new FileInputStream(
                                     new File("my.keystore"));
      try {
          trustStore.load(instream, "changeit".toCharArray());
      } finally {
          instream.close();
      }

      SSLSocketFactory socketFactory =
                                     new SSLSocketFactory(trustStore);
      Scheme scheme = new Scheme("https", socketFactory, 443);
      client.getConnectionManager()
             .getSchemeRegistry().register(scheme);

      HttpGet httpget = new HttpGet("https://localhost/");

      ... proceed with the invocation ...
   }
}
```


### Advantages and Disadvantages


Apache HttpClient is a more complete solution and is better designed than **java.net.HttpURLConnection**. Although you have to download it separately from the JDK, I highly recommend you take a look at it. It has none of the disadvantages of **HttpURLConnection**, except that it is not JAX-RS–aware. Many JAX-RS implementations, including RESTEasy, allow you to use Apache HttpClient as the underlying HTTP client engine, so you can get the best of both worlds.



---
[^20] For more information, see http://hc.apache.org.