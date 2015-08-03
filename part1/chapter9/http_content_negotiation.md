# Chapter 9. HTTP Content Negotiation


Within any meaningfully sized organization or on the Internet, SOA (service-oriented architecture) applications need to be flexible enough to handle and integrate with a variety of clients and platforms. RESTful services have an advantage in this area because most programming languages can communicate with the HTTP protocol. This is not enough, though. Different clients need different formats in order to run efficiently. Java clients might like their data within an XML format. Ajax clients work a lot better with JSON. Ruby clients prefer YAML. Clients may also want internationalized data so that they can provide translated information to their English, Chinese, Japanese, Spanish, or French users. Finally, as our RESTful applications evolve, older clients need a clean way to interact with newer versions of our web services.


HTTP does have facilities to help with these types of integration problems. One of its most powerful features is a client’s capability to specify to a server how it would like its responses formatted. The client can negotiate the content type of the message body, how it is encoded, and even which human language it wants the data translated into. This protocol is called HTTP Content Negotiation, or *conneg* for short. In this chapter, I’ll explain how conneg works, how JAX-RS supports it, and most importantly how you can leverage this feature of HTTP within your RESTful web services.

