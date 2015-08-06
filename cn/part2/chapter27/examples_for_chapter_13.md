# Chapter 27. Examples for Chapter 13


In [Chapter 13](../../part1/chapter13/asynchronous_jax_rs.md), you learned how clients can invoke HTTP requests in the background. You also learned how the server side can detach response processing from the original calling thread with an **AsyncResponse**. In this chapter, we’ll use both of these features to implement a customer chat service.