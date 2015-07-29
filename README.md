# Foreword

**Marc Hadley**

JAX-RS 1.0 Specification Lead


REST is an architectural style that defines a set of constraints that, when applied to the architecture of a distributed system, induce desirable properties like loose coupling and horizontal scalability. RESTful web services are the result of applying these constraints to services that utilize web standards such as URIs, HTTP, XML, and JSON. Such services become part of the fabric of the Web and can take advantage of years of web engineering to satisfy their clients’ needs.


The Java API for RESTful web services (JAX-RS) is a new API that aims to make development of RESTful web services in Java simple and intuitive. The initial impetus for the API came from the observation that existing Java Web APIs were generally either:

* Very low level, leaving the developer to do a lot of repetitive and error-prone work such as URI parsing and content negotiation, or
* Rather high level and proscriptive, making it easy to build services that conform to a particular pattern but lacking the necessary flexibility to tackle more general problems.


A Java Specification Request (JSR 311) was filed with the Java Community Process (JCP) in January 2007 and approved unanimously in February. The expert group began work in April 2007 with the charter to design an API that was flexible and easy to use, and that encouraged developers to follow the REST style. The resulting API, finalized in October 2008, has already seen a remarkable level of adoption, and we were fortunate to have multiple implementations of the API under way throughout the development of JAX-RS. The combination of implementation experience and feedback from users of those implementations was invaluable and allowed us to refine the specification, clarify edge cases, and reduce API friction.


JAX-RS is one of the latest generations of Java APIs that make use of Java annotations to reduce the need for standard base classes, implementing required interfaces, and out-of-band configuration files. Annotations are used to route client requests to matching Java class methods and declaratively map request data to the parameters of those methods. Annotations are also used to provide static metadata to create responses. JAX-RS also provides more traditional classes and interfaces for dynamic access to request data and for customizing responses.


Bill Burke led the development of one of the JAX-RS implementations mentioned earlier (RESTEasy) and was an active and attentive member of the expert group. His contributions to expert group discussions are too numerous to list, but a few of the areas where his input was instrumental include rules for annotation inheritance, use of regular expressions for matching request URIs, annotation-driven support for cookies and form data, and support for streamed output.


This book, RESTful Java with JAX-RS 2.0, provides an in-depth tutorial on JAX-RS and shows how to get the most from this new API while adhering to the REST architectural style. I hope you enjoy the book and working with JAX-RS.