# Chapter 6. JAX-RS Content Handlers


In the last chapter, we focused on injecting information from the header of an HTTP request. In this chapter, we will focus on the message body of an HTTP request and response. In the examples in previous chapters, we used low-level streaming to read in requests and write out responses. To make things easier, JAX-RS also allows you to marshal message bodies to and from specific Java types. It has a number of built-in providers, but you can also write and plug in your own providers. Let’s look at them all.


