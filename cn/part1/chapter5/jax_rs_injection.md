# Chapter 5. JAX-RS Injection


A lot of JAX-RS is pulling information from an HTTP request and injecting it into a Java method. You may be interested in a fragment of the incoming URI. You might be interested in a URI query string value. The client might be sending critical HTTP headers or cookie values that your service needs to process the request. JAX-RS lets you grab this information à la carte, as you need it, through a set of injection annotations and APIs.

