# Language Negotiation


HTTP Content Negotiation also has a simple protocol for negotiating the desired human language of the data sent back to the client. Clients use the **Accept-Language** header to specify which human language they would like to receive. For example:


```
GET http://example.com/stuff
Accept-Language: en-us, es, fr
```


Here, the client is asking for a response in English, Spanish, or French. The **Accept-Language** header uses a coded format. Two digits represent a language identified by the ISO-639 standard.[^8] You can further specialize the code by following the two-character language code with an ISO-3166 two-character country code.[^9] In the previous example, **en-us** represents US English.


The **Accept-Language** header also supports preference qualifiers:


```
GET http://example.com/stuff
Accept-Language: fr;q=1.0, es;q=1.0, en=0.1
```

Here, the client prefers French or Spanish, but would accept English as the default translation.


Clients and servers use the **Content-Language** header to specify the human language for message body translation.

---

[^8] For more information, see [the w3 website](http://www.w3.org/wai/er/ig/ert/iso639.htm).

[^9] For more information, see the ISO website.


