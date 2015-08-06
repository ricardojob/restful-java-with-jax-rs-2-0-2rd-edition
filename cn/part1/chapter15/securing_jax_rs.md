# Chapter 15. Securing JAX-RS


Many RESTful web services will want secure access to the data and functionality they provide. This is especially true for services that will be performing updates. They will want to prevent sniffers on the network from reading their messages. They may also want to fine-tune which users are allowed to interact with a specific service and disallow certain actions for specific users. The Web and the umbrella specification for JAX-RS, Java EE, provide a core set of security services and protocols that you can leverage from within your RESTful web services. These include:


* Authentication

  Authentication is about validating the identity of a client that is trying to access your services. It usually involves checking to see if the client has provided an existing user with valid credentials, such as a password. The Web has a few standardized protocols you can use for authentication. Java EE, specifically your servlet container, has facilities to understand and configure these Internet security authentication protocols.

* Authorization

  Once a client is authenticated, it will want to interact with your RESTful web service. Authorization is about deciding whether or not a certain user is allowed to access and invoke on a specific URI. For example, you may want to allow write access (PUT/POST/DELETE operations) for one set of users and disallow it for others. Authorization is not part of any Internet protocol and is really the domain of your servlet container and Java EE.

* Encryption

  When a client is interacting with a RESTful web service, it is possible for hostile individuals to intercept network packets and read requests and responses if your HTTP connection is not secure. Sensitive data should be protected with cryptographic services like SSL. The Web defines the HTTPS protocol to leverage SSL and encryption.



JAX-RS has a small programmatic API for interacting with servlet and Java EE security, but enabling security in a JAX-RS environment is usually an exercise in configuration and applying annotation metadata.


Beyond Java EE, servlet, and JAX-RS security configuration and APIs, there’s a few areas these standards don’t cover. One area is digital signatures and encryption of the HTTP message body. Your representations may be passing through untrusted intermediaries and signing or encrypting the message body may add some extra protection for your data. There’s also advanced authentication protocols like OAuth, which allow you to make invocations on services on behalf of other users.


This chapter first focuses on the various web protocols used for authentication in a standard, vanilla Java EE, and servlet environment. You’ll learn how to configure your JAX-RS applications to use standard authentication, authorization, and encryption. Next you’ll learn about various formats you can use to digitally sign or encrypt message bodies. Finally, we’ll talk about the OAuth protocol and how you can use it within your applications.
