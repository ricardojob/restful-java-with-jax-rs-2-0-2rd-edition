# java.net.URL


Like most programming languages, Java has a built-in HTTP client library. It’s nothing fancy, but it’s good enough to perform most of the basic functions you need. The API is built around two classes, **java.net.URL** and **java.net.HttpURLConnection**. The **URL** class is just a Java representation of a URL. Here are some of the pertinent constructors and methods:



```Java
public class URL {

   public URL(java.lang.String s)
            throws java.net.MalformedURLException {}

   public java.net.URLConnection
            openConnection() throws java.io.IOException {}
...
}
```


From a **URL**, you can create an **HttpURLConnection** that allows you to invoke specific requests. Here’s an example of doing a simple GET request:



```Java
URL url = new URL("http://example.com/customers/1");
connection = (HttpURLConnection) url.openConnection();
connection.setRequestMethod("GET");
connection.setRequestProperty("Accept", "application/xml");

if (connection.getResponseCode() != 200) {
  throw new RuntimeException("Operation failed: "
                              + connection.getResponseCode());
}

System.out.println("Content-Type: " + connection.getContentType());

BufferedReader reader = new BufferedReader(new
              InputStreamReader(connection.getInputStream()));

String line = reader.readLine();
while (line != null) {
   System.out.println(line);
   line = reader.readLine();
}
connection.disconnect();
```


In this example, we instantiate a **URL** instance and then open a connection using the **URL.openConnection()** method. This method returns a generic **URLConnection** type, so we need to typecast it to an **HttpURLConnection**. Once we have a connection, we set the HTTP method we are invoking by calling **HttpURLConnection.setMethod()**. We want XML from the server, so we call the **setRequestProperty()** method to set the **Accept** header. We get the response code and **Content-Type** by calling **getResponseCode()** and **getContentType()**, respectively. The **getInputStream()** method allows us to read the content sent from the server using the Java streaming API. We finish up by calling **disconnect()**.


Sending content to the server via a PUT or POST is a little different. Here’s an example of that:


```Java
URL url = new URL("http://example.com/customers");
HttpURLConnection connection = (HttpURLConnection) url.openConnection();
connection.setDoOutput(true);
connection.setInstanceFollowRedirects(false);
connection.setRequestMethod("POST");
connection.setRequestProperty("Content-Type", "application/xml");
OutputStream os = connection.getOutputStream();
os.write("<customer id='333'/>".getBytes());
os.flush();
if (connection.getResponseCode() != HttpURLConnection.HTTP_CREATED) {
   throw new RuntimeException("Failed to create customer");
}
System.out.println("Location: " + connection.getHeaderField("Location"));
connection.disconnect();
```


In this example, we create a customer by using POST. We’re expecting a response of 201, “Created,” as well as a **Location** header in the response that points to the URL of our newly created customer. We need to call **HttpURLConnection.setDoOutput(true)**. This allows us to write a body for the request. By default, **HttpURLConnection** will automatically follow redirects. We want to look at our **Location** header, so we call **setInstanceFollowRedirects(false)** to disable this feature. We then call **setRequestMethod()** to tell the connection we’re making a POST request. The **setRequestProperty()** method is called to set the **Content-Type** of our request. We then get a **java.io.OutputStream** to write out the data and the **Location** response header by calling **getHeaderField()**. Finally, we call **disconnect()** to clean up our connection.



### Caching


By default, **HttpURLConnection** will cache results based on the caching response headers discussed in [Chapter 11](../chapter11/scaling_jax_rs_applications.md). You must invoke **HttpURLConnection.setUseCaches(false)** to turn off this feature.


### Authentication


The **HttpURLConnection** class supports Basic, Digest, and Client Certificate Authentication. Basic and Digest Authentication use the **java.net.Authenticator** API. Here’s an example:



```Java
Authenticator.setDefault(new Authenticator() {
    protected PasswordAuthentication getPasswordAuthentication() {
        return new PasswordAuthentication ("username, "password".toCharArray());
    }
});
```


The **setDefault()** method is a static method of **Authenticator**. You pass in an **Authenticator** instance that overrides the class’s **getPasswordAuthentication()** method. You return a **java.net.PasswordAuthentication** object that encapsulates the username and password to access your server. When you do **HttpURLConnection** invocations, authentication will automatically be set up for you using either Basic or Digest, depending on what the server requires.


The weirdest part of the API is that it is driven by the static method **setDefault()**. The problem with this is that your **Authenticator** is set VM-wide. So, doing authenticated requests in multiple threads to different servers is a bit problematic with the basic example just shown. You can address this by using **java.lang.ThreadLocal** variables to store username and passwords:



```Java
public class MultiThreadedAuthenticator extends Authenticator {

   private static ThreadLocal<String> username = new ThreadLocal<String>();
   private static ThreadLocal<String> password = new ThreadLocal<String>();

   public static void setThreadUsername(String user) {
      username.set(user);
   }

   public static void setThreadPassword(String pwd) {
      password.set(pwd);
   }

    protected PasswordAuthentication getPasswordAuthentication() {
        return new PasswordAuthentication (username.get(),
                                           password.get().toCharArray());
    }
}
```


The **ThreadLocal** class is a standard class that comes with the JDK. When you call **set()** on it, the value will be stored and associated with the calling thread. Each thread can have its own value. **ThreadLocal.get()** returns the thread’s current stored value. So, using this class would look like this:



```Java
Authenticator.setDefault(new MultiThreadedAuthenticator());

MultiThreadedAuthenticator.setThreadUsername("bill");
MultiThreadedAuthenticator.setThreadPassword("geheim");
```


#### Client Certificate Authentication


Client Certificate Authentication is a little different. First, you must generate a client certificate using the **keytool** command-line utility that comes with the JDK:


```shell
$ <JAVA_HOME>/bin/keytool -genkey -alias client-alias -keyalg RSA
-keypass changeit -storepass changeit -keystore keystore.jks
```


Next, you must export the certificate into a file so it can be imported into a truststore:


```shell
$ <JAVA_HOME>/bin/keytool -export -alias client-alias
-storepass changeit -file client.cer -keystore keystore.jks
```


Finally, you create a truststore and import the created client certificate:


```shell
$ <JAVA_HOME>\bin\keytool -import -v -trustcacerts
-alias client-alias -file client.cer
-keystore cacerts.jks
-keypass changeit -storepass changeit
```


Now that you have a truststore, use it to create a **javax.net.ssl.SSLSocketFactory** within your client code:



```Java
import javax.net.ssl.SSLContext;
import javax.net.ssl.KeyManagerFactory;
import javax.net.ssl.SSLSocketFactory;
import java.security.SecureRandom;
import java.security.KeyStore;
import java.io.FileInputStream;
import java.io.InputStream;
import java.io.File;

public class MyClient {

   public static SSLSocketFactory
             getFactory( File pKeyFile, String pKeyPassword )
                                                   throws Exception {
     KeyManagerFactory keyManagerFactory =
                           KeyManagerFactory.getInstance("SunX509");
     KeyStore keyStore = KeyStore.getInstance("PKCS12");

     InputStream keyInput = new FileInputStream(pKeyFile);
     keyStore.load(keyInput, pKeyPassword.toCharArray());
     keyInput.close();

     keyManagerFactory.init(keyStore, pKeyPassword.toCharArray());

     SSLContext context = SSLContext.getInstance("TLS");
     context.init(keyManagerFactory.getKeyManagers(), null
                    , new SecureRandom());

     return context.getSocketFactory();
   }
```


This code loads the truststore into memory and creates an **SSLSocketFactory**. The factory can then be registered with a **java.net.ssl.HttpsURLConnection**:


```Java
   public static void main(String args[]) throws Exception {
      URL url = new URL("https://someurl");
      HttpsURLConnection con = (HttpsURLConnection) url.openConnection();
      con.setSSLSocketFactory(getFactory(new File("cacerts.jks"),
                                "changeit"));
   }
}
```


You may then make invocations to the URL, and the client certificate will be used for authentication.


### Advantages and Disadvantages


The biggest advantage of using the **java.net** package as a RESTful client is that it is built in to the JDK. You don’t need to download and install a different client framework.


There are a few disadvantages to the java.net API. First, it is not JAX-RS–aware. You will have to do your own stream processing and will not be able to take advantage of any of the **MessageBodyReaders** and **MessageBodyWriters** that come with your JAX-RS implementation.


Second, the framework does not do preemptive authentication for Basic or Digest Authentication. This means that **HttpURLConnection** will first try to invoke a request without any authentication headers set. If the server requires authentication, the initial request will fail with a 401, “Unauthorized,” response code. The **HttpURLConnection** implementation then looks at the **WWW-Authenticate** header to see whether Basic or Digest Authentication should be used and retries the request. This can have an impact on the performance of your system because each authenticated request will actually be two requests between the client and server.
Third, the framework can’t do something as simple as form parameters. All you have to work with are **java.io.OutputStream** and **java.io.InputStream** to perform your input and output.


Finally, the framework only allows you to invoke the HTTP methods GET, POST, DELETE, PUT, TRACE, OPTIONS, and HEAD. If you try to invoke any HTTP method other than those, an exception is thrown and your invocation will abort. In general, this is not that important unless you want to invoke newer HTTP methods like those defined in the WebDAV specification.
