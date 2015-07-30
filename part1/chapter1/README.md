# Chapter 1. Introduction to REST

For those of us with computers, the World Wide Web is an intricate part of our lives. We use it to read the newspaper in the morning, pay our bills, perform stock trades, and buy goods and services, all through the browser, all over the network. “Googling” has become a part of our daily vocabulary as we use search engines to do research for school, find what time a movie is playing, or just search for information on old friends. Door-to-door encyclopedia salesmen have gone the way of the dinosaur as Wikipedia has become the summarized source of human knowledge. People even socialize over the network using sites like Facebook and Google+. Professional social networks are sprouting up in all industries as doctors, lawyers, and all sorts of professionals use them to collaborate. The Web is an intricate part of our daily jobs as programmers. We search for and download open source libraries to help us develop applications and frameworks for our companies. We build web-enabled applications so that anybody on the Internet or intranet can use a browser to interact with our systems.


Really, most of us take the Web for granted. Have you, as a programmer, sat down and tried to understand why the Web has been so successful? How has it grown from a simple network of researchers and academics to an interconnected worldwide community? What properties of the Web make it so viral?

One man, Roy Fielding, did ask these questions in his doctoral thesis, “Architectural Styles and the Design of Network-based Software Architectures.”[1] In it, he identifies specific architectural principles that answer the following questions:

* Why is the Web so prevalent and ubiquitous? 
* What makes the Web scale? 
* How can I apply the architecture of the Web to my own applications?


The set of these architectural principles is called REpresentational State Transfer (REST) and is defined as:

Addressable resources : 
The key abstraction of information and data in REST is a resource, and each resource must be addressable via a URI (Uniform Resource Identifier).

A uniform, constrained interface : 
Use a small set of well-defined methods to manipulate your resources.

Representation-oriented : You interact with services using representations of that service. A resource referenced by one URI can have different formats. Different platforms need different formats. For example, browsers need HTML, JavaScript needs JSON (JavaScript Object Notation), and a Java application may need XML.


Communicate statelessly : Stateless applications are easier to scale.

Hypermedia As The Engine Of Application State (HATEOAS) : Let your data formats drive state transitions in your applications.



For a PhD thesis, Fielding’s paper is actually very readable and, thankfully, not very long. It, along with Leonard Richardson and Sam Ruby’s book RESTful Web APIs (O’Reilly), is an excellent reference for understanding REST. I will give a much briefer introduction to REST and the Internet protocol it uses (HTTP) within this chapter.