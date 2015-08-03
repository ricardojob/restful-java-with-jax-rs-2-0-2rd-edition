# Chapter 12. Filters and Interceptors


Filters and interceptors are objects that are able to interpose themselves on client or server request processing. They allow you to encapsulate common behavior that cuts across large parts of your application. This behavior is usually infrastructure- or protocol-related code that you don’t want to pollute your business logic with. While most JAX-RS features are applied by application developers, filters and interceptors are targeted more toward middleware and systems developers. They are also often used to write portable extensions to the JAX-RS API. This chapter teaches you how to write filters and interceptors using real-world examples.

