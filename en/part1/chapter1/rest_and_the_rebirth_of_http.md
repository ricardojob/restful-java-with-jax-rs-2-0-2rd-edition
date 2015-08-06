# REST and the Rebirth of HTTP


REST isn’t protocol-specific, but when people talk about REST, they usually mean REST over HTTP. Learning about REST was as much of a rediscovery and reappreciation of the HTTP protocol for me as learning a new style of distributed application development. Browser-based web applications see only a tiny fraction of the features of HTTP. Non-RESTful technologies like SOAP and WS-* use HTTP strictly as a transport protocol and thus use a very small subset of its capabilities. Many would say that SOAP and WS-* use HTTP solely to tunnel through firewalls. HTTP is actually a very rich application protocol that provides a multitude of interesting and useful capabilities for application developers. You will need a good understanding of HTTP in order to write RESTful web services.


HTTP is a synchronous request/response-based application network protocol used for distributed, collaborative, document-based systems. It is the primary protocol used on the Web, in particular by browsers such as Firefox, MS Internet Explorer, Safari, and Netscape. The protocol is very simple: the client sends a request message made up of the HTTP method being invoked, the location of the resource you are interested in invoking, a variable set of headers, and an optional message body that can basically be anything you want, including HTML, plain text, XML, JSON, and even binary data. Here’s an example:

```shell
GET /resteasy HTTP/1.1
Host: jboss.org
User-Agent: Mozilla/5.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language:      en-us,en;q=0.5
Accept-Encoding:     gzip,deflate
```


Your browser would send this request if you wanted to look at [http://jboss.org/resteasy](http://jboss.org/resteasy). GET is the method we are invoking on the server. /resteasy is the object we are interested in. HTTP/1.1 is the version of the protocol. Host, User-Agent, Accept, Accept-Language, and Accept-Encoding are all message headers. There is no request body, as we are querying information from the server.


The response message from the server is very similar. It contains the version of HTTP we are using, a response code, a short message that explains the response code, a variable set of optional headers, and an optional message body. Here’s the message the server might respond with using the previous GET query:

```html
HTTP/1.1 200 OK
X-Powered-By: Servlet 2.4; JBoss-4.2.2.GA
Content-Type: text/html

<head>
<title>JBoss RESTEasy Project</title>
</head>
<body>
<h1>JBoss RESTEasy</h1>
<p>JBoss RESTEasy is an open source implementation of the JAX-RS specification...
```


The response code of this message is 200, and the status message is “OK.” This code means that the request was processed successfully and that the client is receiving the information it requested. HTTP has a large set of response codes. They can be informational codes like 200, “OK,” or error codes like 500, “Internal Server Error.” Visit the w3c’s website for a more complete and verbose listing of these codes.


This response message also has a message body that is a chunk of HTML. We know it is HTML by the **Content-Type** header.