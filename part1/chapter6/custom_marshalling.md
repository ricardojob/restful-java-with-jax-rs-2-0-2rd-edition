# Custom Marshalling


So far in this chapter, we’ve focused on built-in JAX-RS handlers that can marshal and unmarshal message content. Unfortunately, there are hundreds of data formats available on the Internet, and the built-in JAX-RS handlers are either too low level to be useful or may not match the format you need. Luckily, JAX-RS allows you to write your own handlers and plug them into the JAX-RS runtime.


To illustrate how to write your own handlers, we’re going to pretend that there is no built-in JAX-RS JAXB support and instead write one ourselves using JAX-RS APIs.


### MessageBodyWriter


The first thing we’re going to implement is JAXB-marshalling support. To automatically convert Java objects into XML, we have to create a class that implements the **javax.ws.rs.ext.MessageBodyWriter** interface:


```Java
public interface MessageBodyWriter<T> {

   boolean isWriteable(Class<?> type, Type genericType,
                       Annotation annotations[],
                                    MediaType mediaType);


   long getSize(T t, Class<?> type, Type genericType,
                 Annotation annotations[], MediaType mediaType);


   void writeTo(T t, Class<?> type, Type genericType,
                Annotation annotations[],
                MediaType mediaType,
                MultivaluedMap<String, Object> httpHeaders,
                OutputStream entityStream)
                       throws IOException, WebApplicationException;
}
```

The **MessageBodyWriter** interface has only three methods. The **isWriteable()** method is called by the JAX-RS runtime to determine if the writer supports marshalling the given type. The **getSize()** method is called by the JAX-RS runtime to determine the **Content-Length** of the output. Finally, the **writeTo()** method does all the heavy lifting and writes the content out to the HTTP response buffer. Let’s implement this interface to support JAXB:

```Java
@Provider
@Produces("application/xml")
public class JAXBMarshaller implements MessageBodyWriter {

   public boolean isWriteable(Class<?> type, Type genericType,
                     Annotation annotations[], MediaType mediaType) {
      return type.isAnnotationPresent(XmlRootElement.class);
   }
```


We start off the implementation of this class by annotating it with the **@javax.ws.rs.ext.Provider** annotation. This tells JAX-RS that this is a deployable JAX-RS component. We must also annotate it with **@Produces** to tell JAX-RS which media types this **MessageBodyWriter** supports. Here, we’re saying that our **JAXBMarshaller** class supports **application/xml**.


The **isWriteable()** method is a callback method that tells the JAX-RS runtime whether or not the class can handle writing out this type. JAX-RS follows this algorithm to find an appropriate **MessageBodyWriter** to write out a Java object into the HTTP response:

1. First, JAX-RS calculates a list of **MessageBodyWriters** by looking at each writer’s **@Produces** annotation to see if it supports the media type that the JAX-RS resource method wants to output. 
2. This list is sorted, with the best match for the desired media type coming first. In other words, if our JAX-RS resource method wants to output **application/xml** and we have three **MessageBodyWriters** (one produces **application/\***, one supports anything **\*/\***, and the last supports **application/xml**), the one producing **application/xml** will come first.
3. Once this list is calculated, the JAX-RS implementation iterates through the list in order, calling the **MessageBodyWriter.isWriteable()** method. If the invocation returns **true**, that **MessageBodyWriter** is used to output the data. 



The **isWriteable()** method takes four parameters. The first one is a **java.lang.Class** that is the type of the object that is being marshalled. We determine the type by calling the **getClass()** method of the object. In our example, we use this parameter to find out if our object’s class is annotated with the **@XmlRootElement** annotation.


The second parameter is a **java.lang.reflect.Type**. This is generic type information about the object being marshalled. We determine it by introspecting the return type of the JAX-RS resource method. We don’t use this parameter in our **JAXBMarshaller.isWriteable()** implementation. This parameter would be useful, for example, if we wanted to know the type parameter of a **java.util.List** generic type.


The third parameter is an array of **java.lang.annotation.Annotation** objects. These annotations are applied to the JAX-RS resource method we are marshalling the response for. Some **MessageBodyWriters** may be triggered by JAX-RS resource method annotations rather than class annotations. In our **JAXBMarshaller** class, we do not use this parameter in our **isWriteable()** implementation.


The fourth parameter is the media type that our JAX-RS resource method wants to produce.

Let’s examine the rest of our **JAXBMarshaller** implementation:

```Java
   public long getSize(Object obj, Class<?> type, Type genericType,
                     Annotation[] annotations, MediaType mediaType)
   {
      return −1;
   }
```


The **getSize()** method is responsible for determining the **Content-Length** of the response. If you cannot easily determine the length, just return –1. The underlying HTTP layer (i.e., a servlet container) will handle populating the **Content-Length** in this scenario or use the chunked transfer encoding.


The first parameter of **getSize()** is the actual object we are outputting. The rest of the parameters serve the same purpose as the parameters for the **isWriteable()** method.


Finally, let’s look at how we actually write the JAXB object as XML:

```Java
   public void writeTo(Object target,
                       Class<?> type,
                       Type genericType,
                       Annotation[] annotations,
                       MediaType mediaType,
                       MultivaluedMap<String, Object> httpHeaders,
                       OutputStream outputStream) throws IOException
   {
      try {
         JAXBContext ctx = JAXBContext.newInstance(type);
         ctx.createMarshaller().marshal(target, outputStream);
      } catch (JAXBException ex) {
        throw new RuntimeException(ex);
      }
   }
```


The **target**, **type**, **genericType**, **annotations**, and **mediaType** parameters of the **writeTo()** method are the same information passed into the **getSize()** and **isWriteable()** methods. The **httpHeaders** parameter is a **javax.ws.rs.core.MultivaluedMap** that represents the HTTP response headers. You may modify this map and add, remove, or change the value of a specific HTTP header as long as you do this before outputting the response body. The **outputStream****** parameter is a **java.io.OutputStream** and is used to stream out the data.


Our implementation simply creates a **JAXBContext** using the **type** parameter. It then creates a **javax.xml.bind.Marshaller** and converts the Java object to XML.


#### Adding pretty printing


By default, JAXB outputs XML without any whitespace or special formatting. The XML output is all one line of text with no new lines or indentation. We may have human clients looking at this data, so we want to give our JAX-RS resource methods the option to pretty-print the output XML. We will provide this functionality using an **@Pretty** annotation. For example:


```Java
@Path("/customers")
public class CustomerService {

   @GET
   @Path("{id}")
   @Produces("application/xml")
   @Pretty
   public Customer getCustomer(@PathParam("id") int id) {...}
}
```


Since the **writeTo()** method of our **MessageBodyWriter** has access to the **getCustomer()** method’s annotations, we can implement this easily. Let’s modify our **JAXBMarshaller** class:


```Java
 public void writeTo(Object target,
                       Class<?> type,
                       Type genericType,
                       Annotation[] annotations,
                       MediaType mediaType,
                       MultivaluedMap<String, Object> httpHeaders,
                       OutputStream outputStream) throws IOException
   {
      try {
         JAXBContext ctx = JAXBContext.newInstance(type);
         Marshaller m = ctx.createMarshaller();

         boolean pretty = false;
         for (Annotation ann : annotations) {
            if (ann.annotationType().equals(Pretty.class)) {
                pretty = true;
                break;
            }
         }
         if (pretty) {
            marshaller.setProperty(Marshaller.JAXB_FORMATTED_OUTPUT, true);
         }

         m.marshal(target, outputStream);
      } catch (JAXBException ex) {
        throw new RuntimeException(ex);
      }
   }
```


Here, we iterate over the **annotations** parameter to see if any of them are the **@Pretty** annotation. If **@Pretty** has been set, we set the **JAXB_FORMATTED_OUTPUT** property on the **Marshaller** so that it will format the XML with line breaks and indentation strings.



#### Pluggable JAXBContexts using ContextResolvers


Earlier in this chapter, we saw how you could plug in your own **JAXBContext** using the **ContextResolver** interface. Let’s look at how we can add this functionality to our **JAXBMarshaller** class.


First, we need a way to locate a **ContextResolver** that can provide a custom **JAXBContext**. We do this through the **javax.ws.rs.ext.Providers** interface:


```Java
public interface Providers {

   <T> ContextResolver<T> getContextResolver(Class<T> contextType,
                                             MediaType mediaType);

   <T> MessageBodyReader<T>
   getMessageBodyReader(Class<T> type, Type genericType,
                     Annotation annotations[], MediaType mediaType);


   <T> MessageBodyWriter<T>
   getMessageBodyWriter(Class<T> type, Type genericType,
                     Annotation annotations[], MediaType mediaType);


   <T extends Throwable> ExceptionMapper<T>
   getExceptionMapper(Class<T> type);

}
```


We use the **Providers.getContextResolver()** method to find a **ContextResolver**. We inject a reference to a **Providers** object using the **@Context** annotation. Let’s modify our **JAXBMarshaller** class to add this new functionality:


```Java
@Context
protected Providers providers;

public void writeTo(Object target,
                       Class<?> type,
                       Type genericType,
                       Annotation[] annotations,
                       MediaType mediaType,
                       MultivaluedMap<String, Object> httpHeaders,
                       OutputStream outputStream) throws IOException
{
   try {
      JAXBContext ctx = null;
      ContextResolver<JAXBContext> resolver =
           providers.getContextResolver(JAXBContext.class, mediaType);
      if (resolver != null) {
        ctx = resolver.getContext(type);
      }
      if (ctx == null) {
          // create one ourselves
          ctx = JAXBContext.newInstance(type);
      }
      ctx.createMarshaller().marshal(target, outputStream);
   } catch (JAXBException ex) {
     throw new RuntimeException(ex);
   }
}
```


In our **writeTo()** method, we now use the **Providers** interface to find a **ContextResolver** that can give us a custom **JAXBContext**. If one exists, we call **resolver.getContext()**, passing in the type of the object we want a **JAXBContext** for.



The **ContextResolver** returned by **Providers.getContextResolver()** is actually a proxy that sits in front of a list of **ContextResolvers** that can provide **JAXBContext** instances. When **getContextResolver()** is invoked, the proxy iterates on this list, recalling **getContextResolver()** on each individual resolver in the list. If it returns a **JAXBContext** instance, it returns that to the original caller; otherwise, it tries the next resolver in this list.


### MessageBodyReader


Now that we have written a **MessageBodyWriter** to convert a Java object into XML and output it as the HTTP response body, let’s write an unmarshaller that knows how to convert HTTP XML request bodies back into a Java object. To do this, we need to use the **javax.ws.rs.ext.MessageBodyReader** interface:


```Java
public interface MessageBodyReader<T> {

   boolean isReadable(Class<?> type, Type genericType,
                      Annotation annotations[], MediaType mediaType);

   T readFrom(Class<T> type, Type genericType,
              Annotation annotations[], MediaType mediaType,
              MultivaluedMap<String, String> httpHeaders,
              InputStream entityStream)
                         throws IOException, WebApplicationException;

}
```


The **MessageBodyReader** interface has only two methods. The **isReadable()** method is called by the JAX-RS runtime when it is trying to find a **MessageBodyReader** to unmarshal the message body of an HTTP request. The **readFrom()** method is responsible for creating a Java object from the HTTP request body.


Implementing a **MessageBodyReader** is very similar to writing a **MessageBodyWriter**. Let’s look at how we would implement one:


```Java
@Provider
@Consumes("application/xml")
public class JAXBUnmarshaller implements MessageBodyReader {

   public boolean isReadable(Class<?> type, Type genericType,
                      Annotation annotations[], MediaType mediaType) {
      return type.isAnnotationPresent(XmlRootElement.class);
   }
```


Our **JAXBUnmarshaller** class is annotated with **@Provider** and **@Consumes**. The latter annotation tells the JAX-RS runtime which media types it can handle. The matching rules for finding a **MessageBodyReader** are the same as the rules for matching **MessageBodyWriter**. The difference is that the **@Consumes** annotation is used instead of the **@Produces** annotation to correlate media types.


Let’s now look at how we read and convert our HTTP message into a Java object:


```Java
  Object readFrom(Class<Object>, Type genericType,
                  Annotation annotations[], MediaType mediaType,
                  MultivaluedMap<String, String> httpHeaders,
                  InputStream entityStream)
                         throws IOException, WebApplicationException {

      try {
         JAXBContext ctx = JAXBContext.newInstance(type);
         return ctx.createUnmarshaller().unmarshal(entityStream);
      } catch (JAXBException ex) {
        throw new RuntimeException(ex);
      }
   }
```


The **readFrom()** method gives us access to the HTTP headers of the incoming request as well as a **java.io.InputStream** that represents the request message body. Here, we just create a **JAXBContext** based on the Java type we want to create and use a **javax.xml.bind.Unmarshaller** to extract it from the stream.



### Life Cycle and Environment


By default, only one instance of each **MessageBodyReader**, **MessageBodyWriter**, or **ContextResolver** is created per application. If JAX-RS is allocating instances of these components (see [Chapter 14](../chapter14/configuration.md)), the classes of these components must provide a public constructor for which the JAX-RS runtime can provide all the parameter values. A public constructor may only include parameters annotated with the **@Context** annotation. For example:


```Java
@Provider
@Consumes("application/json")
public class MyJsonReader implements MessageBodyReader {

   public MyJsonReader(@Context Providers providers) {
      this.providers = providers;
   }
}
```


Whether or not the JAX-RS runtime is allocating the component instance, JAX-RS will perform injection into properly annotated fields and setter methods. Again, you can only inject JAX-RS objects that are found using the **@Context** annotation.





