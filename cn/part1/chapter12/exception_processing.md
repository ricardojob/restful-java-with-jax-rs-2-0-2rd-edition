# Exception Processing


So what happens if a filter or interceptor throws an exception? On the server side, the JAX-RS runtime will process exceptions in the same way as if an exception were thrown in a JAX-RS method. It will try to find an **ExceptionMapper** for the exception and then run it. If an exception is thrown by a **ContainerRequestFilter** or **ReaderInterceptor** and mapped by an **ExceptionMapper**, then any bound **ContainerResponseFilter** must be invoked. The JAX-RS runtime ensures that at most one **ExceptionMapper** will be invoked in a single request processing cycle. This avoids infinite loops.


On the client side, if the exception thrown is an instance of **WebApplicationException**, then the runtime will propagate it back to application code. Otherwise, the exception is wrapped in a **javax.ws.rs.client.ProcessingException** if it is thrown before the request goes over the wire. The exception is wrapped in a **javax.ws.rs.client.ResponseProcessingException** when processing a response.


