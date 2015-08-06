# Chapter 8. JAX-RS Client API


One huge gaping hole in the first version of the JAX-RS specification was the lack of a client API. You could slog through the very difficult-to-use **java.net.URL** set of classes to invoke on remote RESTful services. Or you could use something like Apache HTTP Client, which is not JAX-RS aware, so you would have to do marshalling and unmarshalling of Java objects manually. Finally, you could opt to use one of the proprietary client APIs of one of the many JAX-RS implementations out there. This would, of course, lock you into that vendor’s implementation. JAX-RS 2.0 fixed this problem by introducing a new HTTP client API.


