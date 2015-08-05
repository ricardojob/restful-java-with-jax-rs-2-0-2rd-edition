# Example ex15_2: JSON Web Encryption


In [Chapter 15](../../part1/chapter15/securing_jax_rs.md), you learned a little bit about JSON Web Encryption (JWE) and how it can be used to encrypt HTTP message body or header values. This example augments the customer chat client implemented in [Chapter 27](../chapter27/examples_for_chapter_13.md). Chat clients will use a shared secret to encrypt and decrypt the messages they send to and receive from the chat server. Chat clients that know the shared secret see the decrypted message, while clients that don’t know it see only the JWE encoding. Let’s take a look at the code:


```Java:src/main/java/ChatClient.java
public class ChatClient
{
   public static void main(String[] args) throws Exception
   {
      String name = args[0];
      final String secret = args[1];
```


The **ChatClient** first starts out by storing the name and secret password that the client will use. It obtains these values from the command line.



```Java
      final Client client = new ResteasyClientBuilder()
                          .connectionPoolSize(3)
                          .build();
      WebTarget target = client.target("http://localhost:8080/services/chat");

      target.request().async().get(new InvocationCallback<Response>()
      {
         @Override
         public void completed(Response response)
         {
            Link next = response.getLink("next");
            String message = response.readEntity(String.class);
            try
            {
               JWEInput encrypted = new JWEInput(message);
               message = encrypted.decrypt(secret).readContent(String.class);
            }
            catch (Exception ignore)
            {
               //e.printStackTrace();
            }
            System.out.println();
            System.out.print(message);
            System.out.println();
            System.out.print("> ");
            client.target(next).request().async().get(this);
         }

         @Override
         public void failed(Throwable throwable)
         {
            System.err.println("FAILURE!");
         }
      });
```


The code then implements the receive loop we discussed in [Chapter 27](../chapter27/examples_for_chapter_13.md). The difference is that it uses the RESTEasy **org.jboss.resteasy.jose.jwe.JWEInput** class to decrypt the received message. A **JWEInput** instance is initialized with the received text message. The **JWEInput.decrypt()** method decrypts the JWE with the shared secret. The **readContext()** method extracts the decrypted bytes into a String object that we can output to the console. If the message is not a JWE or if the wrong secret is used, then the original received text message is outputted to the console.



Let’s now take a look at how sending a message has changed:


```Java
      while (true)
      {
         System.out.print("> ");
         BufferedReader br = new BufferedReader
                            (new InputStreamReader(System.in));
         String message = name + ": " + br.readLine();
         String encrypted = new JWEBuilder()
                                       .contentType(MediaType.TEXT_PLAIN_TYPE)
                                       .content(message)
                                       .dir(secret);
         target.request().post(Entity.text(encrypted));
      }
```


This **while** loop is similar to the code discussed in [Chapter 27](../chapter27/examples_for_chapter_13.md). The difference is that it uses the RESTEasy **org.jboss.resteasy.jose.jwe.JWEBuilder** class to encrypt the text message we want to post to the server. The **JWEBuilder.contentType()** method sets the **cty** header of the JWE. The **content()** method sets the entity we want to encrypt. The **dir()** method first takes the entity and marshals it using a **MessageBodyReader** picked from the content type and the entity’s class. The **dir()** method then generates the JWE based on this marshalled content and shared secret algorithm. Once we have our JWE-encoded string, we then post it to the chat server.


One thing to notice is that we have not changed the server at all. The server is a dumb intermediary that just forwards messages from one client to others. It doesn’t care about what is being sent across the wire.



### Build and Run the Example Program


You’ll need multiple console windows to run this example. In the first console window, perform the following steps:


1. Change to the *ex15_2* directory of the workbook example code.
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md).
3. Perform the build and run the example by typing **maven jetty:run**.


This will start the JAX-RS services for the example.


Open another console window and do the following.


1. Change to the *ex15_2* directory of the workbook example code.
2. Run the chat client by typing **maven exec:java -Dexec.mainClass=ChatClient -Dexec.args=" your-name your-secret"**.



Replace **your-name** with your first name and **your-secret** with your shared password. Repeat this process in yet another console window to run a second chat client. You may also want to start different chat clients that use different passwords to see what happens.
