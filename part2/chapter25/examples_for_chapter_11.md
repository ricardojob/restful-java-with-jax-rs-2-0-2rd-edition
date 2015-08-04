# Chapter 25. Examples for Chapter 11


In [Chapter 11](../../part1/chapter11/scaling_jax_rs_applications.md), you learned about HTTP caching techniques. Servers can tell HTTP clients if and how long they can cache retrieved resources. You can revalidate expired caches to avoid resending big messages by issuing conditional GET invocations. Conditional PUT operations can be invoked for safe concurrent updates.