# Encoding Negotiation


Clients can also negotiate the encoding of a message body. To save on network bandwidth, encodings are generally used to compress messages before they are sent. The most common algorithm for encoding is GZIP compression. Clients use the **Accept-Encoding** header to specify which encodings they support. For example:


```
GET http://example.com/stuff
Accept-Encoding: gzip, deflate
```


Here, the client is saying that it wants its response either compressed using GZIP or uncompressed (**deflate**).


The **Accept-Encoding** header also supports preference qualifiers:


```
GET http://example.com/stuff
Accept-Encoding: gzip;q=1.0, compress;0.5; deflate;q=0.1
```


Here, **gzip** is desired first, then **compress**, followed by **deflate**. In practice, clients use the **Accept-Encoding** header to tell the server which encoding formats they support, and they really don’t care which one the server uses.


When a client or server encodes a message body, it must set the **Content-Encoding** header. This tells the receiver which encoding was used.



