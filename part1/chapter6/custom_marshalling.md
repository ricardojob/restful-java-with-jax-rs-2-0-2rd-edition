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










