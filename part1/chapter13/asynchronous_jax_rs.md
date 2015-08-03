# Chapter 13. Asynchronous JAX-RS


Another interesting new feature introduced in JAX-RS 2.0 is asynchronous request and response processing both on the client and server side. If you are mashing together a lot of data from different websites or you have something like a stock quote application that needs to push events to hundreds or thousands of idle blocking clients, then the JAX-RS 2.0 asynchronous APIs are worth looking into.
