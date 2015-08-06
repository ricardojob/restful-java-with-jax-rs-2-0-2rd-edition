# Reader and Writer Interceptors


While filters modify request or response headers, reader and writer interceptors deal with message bodies. They work in conjunction with a **MessageBodyReader** or **MessageBodyWriter** and are usable on both the client and server. Reader interceptors implement the **ReaderInterceptor** interface. Writer interceptors implement the **WriterInterceptor** interface.


```Java
package javax.ws.rs.ext;

public interface ReaderInterceptor {
    public Object aroundReadFrom(ReaderInterceptorContext context)
            throws java.io.IOException, javax.ws.rs.WebApplicationException;
}

public interface WriterInterceptor {
    void aroundWriteTo(WriterInterceptorContext context)
            throws java.io.IOException, javax.ws.rs.WebApplicationException;
}
```


These interceptors are only triggered when a **MessageBodyReader** or **MessageBodyWriter** is needed to unmarshal or marshal a Java object to and from the HTTP message body. They also are invoked in the same Java call stack. In other words, a **ReaderInterceptor** wraps around the invocation of **MessageBodyReader.readFrom()** and a **WriterInterceptor** wraps around the invocation of **MessageBodyWWriter.writeTo()**.


A simple example that illustrates these interfaces in action is adding compression to your input and output streams through content encoding. While most JAX-RS implementations support GZIP encoding, let’s look at how you might add support for it using a **ReaderInterceptor** and **WriterInterceptor**:



```Java
@Provider
public class GZIPEncoder implements WriterInterceptor {

   public void aroundWriteTo(WriterInterceptorContext ctx)
                    throws IOException, WebApplicationException {
      GZIPOutputStream os = new GZIPOutputStream(ctx.getOutputStream());
      ctx.getHeaders().putSingle("Content-Encoding", "gzip");
      ctx.setOutputStream(os);
      ctx.proceed();
      return;
   }
}
```



The **WriterInterceptorContext** parameter allows you to view and modify the HTTP headers associated with this invocation. Since interceptors can be used on both the client and server side, these headers represent either a client request or a server response. In the example, our **aroundWriteTo()** method uses the **WriterInterceptorContext** to get and replace the **OutputStream** of the HTTP message body with a **GZipOutputStream**. We also use it to add a **Content-Encoding** header. The call to **WriterInterceptorContext.proceed()** will either invoke the next registered **WriterInterceptor**, or if there aren’t any, invoke the underlying **MessageBodyWriter.writeTo()** method.


Let’s now implement the **ReaderInterceptor** counterpart to this encoding example:


```Java
@Provider
public class GZIPDecoder implements ReaderInterceptor {
   public Object aroundReadFrom(ReaderInterceptorContext ctx)
                                throws IOException {
      String encoding = ctx.getHeaders().getFirst("Content-Encoding");
      if (!"gzip".equalsIgnoreCase(encoding)) {
         return ctx.proceed();
      }
      GZipInputStream is = new GZipInputStream(ctx.getInputStream());
      ctx.setInputStream(is);
      return ctx.proceed(is);
   }
}
```



The **ReaderInterceptorContext** parameter allows you to view and modify the HTTP headers associated with this invocation. Since interceptors can be used on both the client and server side, these headers represent either a client response or a server request. In the example, our **aroundReadFrom()** method uses the **ReaderInterceptorContext** to first check to see if the message body is GZIP encoded. If not, it returns with a call to **ReaderInterceptorContext.proceed()**. The **ReaderInterceptorContext** is also used to get and replace the **InputStream** of the HTTP message body with a **GZipInputStream**. The call to **ReaderInterceptorContext.proceed()** will either invoke the next registered **ReaderInterceptor**, or if there aren’t any, invoke the underlying **MessageBodyReader.readFrom()** method. The value returned by **proceed()** is whatever was returned by **MessageBodyReader.readFrom()**. You can change this value if you want, by returning a different value from your **aroundReadFrom()** method.



There’s a lot of other use cases for interceptors that I’m not going to go into detail with. For example, the RESTEasy project uses interceptors to digitally sign and/or encrypt message bodies into a variety of Internet formats. You could also use a **WriterInterceptor** to add a JSONP wrapper to your JSON content. A **ReaderInterceptor** could augment the unmarshalled Java object with additional data pulled from the request or response. The rest is up to your imagination.