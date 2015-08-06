# Conneg Explained


The first part of HTTP Content Negotiation is that clients can request a specific media type they would like returned when querying a server for information. Clients set an Accept request header that is a comma-delimited list of preferred formats. For example:


```
GET http://example.com/stuff
Accept: application/xml, application/json
```


In this example request, the client is asking the server for **/stuff** formatted in either XML or JSON. If the server is unable to provide the desired format, it will respond with a status code of 406, “Not Acceptable.” Otherwise, the server chooses one of the media types and sends a response in that format back to the client.


Wildcards and media type properties can also be used within the **Accept** header listing. For example:


```
GET http://example.com/stuff
Accept: text/*, text/html;level=1
```

The **text/\*** media type means any text format.


### Preference Ordering


The protocol also has both implicit and explicit rules for choosing a media type to respond with. The implicit rule is that more specific media types take precedence over less specific ones. Take this example:


```
GET http://example.com/stuff
Accept: text/*, text/html;level=1, */*, application/xml
```

The server assumes that the client always wants a concrete media type over a wildcard one, so the server would interpret the client preference as follows:

1. **text/html;level=1**
2. **application/xml**
3. **text/\***
4. **\*/\***


The **text/html;level=1** type would come first because it is the most specific. The **application/xml** type would come next because it does not have any MIME type properties like **text/html;level=1** does. After this would come the wildcard types, with **text/\*** coming first because it is obviously more concrete than the match-all qualifier **\*/\***.


Clients can also be more specific on their preferences by using the q MIME type property. This property is a numeric value between 0.0 and 1.0, with 1.0 being the most preferred. For example:

```
GET http://example.com/stuff
Accept: text/*;q=0.9, */*;q=0.1, audio/mpeg, application/xml;q=0.5
```

If no **q** qualifier is given, then a value of **1.0** must be assumed. So, in our example request, the preference order is as follows:

1. **audio/mpeg**
2. **text/\***
3. **application/xml**
4. **\*/\***


The **audio/mpeg** type is chosen first because it has an implicit qualifier of **1.0**. Text types come next, as **text/\*** has a qualifier of **0.9**. Even though **application/xml** is more specific, it has a lower preference value than **text/\***, so it follows in the third spot. If none of those types matches the formats the server can offer, anything can be passed back to the client.

