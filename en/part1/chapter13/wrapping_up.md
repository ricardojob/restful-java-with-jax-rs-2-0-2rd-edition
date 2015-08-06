# Wrapping Up


In this chapter, we discussed how you can use JAX-RS asynchronously both on the client and server side. On the client, you can execute one or more requests in the background and either poll for their response, or receive a callback. On the server, we saw that you can suspend requests so that a different thread can handle response processing. This is a great way to scale specific kinds of applications. [Chapter 27](../../part2/chapter27/example_ex13_1_chat_rest_interface.md) walks you through a bunch of code examples that show most of these features in action.
