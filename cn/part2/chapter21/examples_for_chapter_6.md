# Chapter 21. Examples for Chapter 6


In [Chapter 3](../../part1/chapter3/your_first_jax_rs_service.md), you saw a quick overview on how to write a simple JAX-RS service. You might have noticed that we needed a lot of code to process incoming and outgoing XML data. In [Chapter 6](../../part1/chapter6/jax_rs_content_handlers.md), you learned that all this handcoded marshalling code is unnecessary. JAX-RS has a number of built-in content handlers that can do the processing for you. You also learned that if these prepackaged providers do not meet your requirements, you can write your own content handler.


There are two examples in this chapter. The first rewrites the *ex03_1* example to use JAXB instead of handcoded XML marshalling. The second example implements a custom content handler that can send serialized Java objects over HTTP.