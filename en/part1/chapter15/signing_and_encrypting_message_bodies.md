# Signing and Encrypting Message Bodies


Sometimes you have RESTful clients or services that may have to send or receive HTTP messages from unknown or untrusted intermediaries. A great example of an intermediary is Twitter. You post tweets to Twitter through the Twitter REST API, and one or more people can receive these tweets via Twitter. What if a tweet receiver wanted to verify that the tweet originator is who he says he is? Or what if you wanted to post encrypted tweets through Twitter that only trusted receivers could decode and read? This interaction is different from HTTPS in that HTTPS is a trusted SSL socket connection between one client and one server. For the Twitter example, we’re sending a representation that is retransmitted via a totally different HTTP request involving different clients and servers. Digitally signing or encrypting the representation gives you the protection you need in this retransmission scenario.


### Digital Signatures


Java developers are intimately familiar with the **HashMap** class. The way maps work is that a semi-unique hash code is generated for the key you are storing in the map. The key’s hash code is used as an array index to quickly look up a value in the map. Under the covers, a digital signature is simply an encrypted hash code of the piece of data you are transmitting.


While a shared secret can be used to generate and verify a digital signature, the best approach is to use an asymmetric key pair: in other words, a private and public key. The signer creates the digital signature of the message using its private key. It then publishes its public key to the world. The receiver of the message uses the public key to verify the signature. If you use the right hash and encryption algorithms, it is virtually impossible to derive the private key of the sender or fake the signatures. I’m going to go over two methods you can use to leverage digital signatures in your RESTful web services.


#### DKIM/DOSETA


DomainKeys Identified Mail (DKIM)[^16].] is a digital signature protocol that was designed for email. Work is also being done to apply this header to protocols other than email (e.g., HTTP) through the DOSETA[^17] specifications. DKIM is simply a request or response header that contains a digital signature of one or more headers of the message and the content. What’s nice about DKIM is that its header is self-contained and not part of the transmitted representation. So if the receiver of an HTTP message doesn’t care about digital signatures, it can just ignore the header.


The format of a DKIM header is a semicolon-delimited list of name/value pairs. Here’s an example:



```
DKIM-Signature: v=1;
                a=rsa-sha256;
                d=example.com;
                s=burke;
                c=simple/simple;
                h=Content-Type;
                x=0023423111111;
                bh=2342322111;
                b=M232234=
```



While it’s not that important to know the structure of the header, here’s an explanation of each parameter:


* **v**

	Protocol version. Always 1.

* **a**

	Algorithm used to hash and sign the message. RSA signing and SHA256 hashing is the only supported algorithm at the moment by RESTEasy.

* **d**

	Domain of the signer. This is used to identify the signer as well as discover the public key to use to verify the signature.

* **s**

	Selector of the domain. Also used to identify the signer and discover the public key.

* **c**

	Canonical algorithm. Only simple/simple is supported at the moment. Basically, this allows you to transform the message body before calculating the hash.

* **h**

	Semicolon-delimited list of headers that are included in the signature calculation.

* **x**

	When the signature expires. This is a numeric long value of the time in seconds since epoch. Allows the signer to control when a signed message’s signature expires.

* **t**

	Timestamp of signature. Numeric long value of the time in seconds since epoch. Allows the verifier to control when a signature expires.

* **bh**

	Base 64–encoded hash of the message body.

* **b**

	Base 64–encoded signature.



What’s nice about DKIM is that you can include individual headers within your digital signature of the message. Usually **Content-Type** is included.



To verify a signature, you need a public key. DKIM uses DNS text records to discover a public key. To find a public key, the verifier concatenates the selector (**s** parameter) with the domain (**d** parameter):



```
<selector>._domainKey.<domain>
```


It then takes that string and does a DNS request to retrieve a **TXT** record under that entry. In our previous example, **burke._domainKey.example.com** would be used as the lookup string.



This is a very interesting way to publish public keys. For one, it becomes very easy for verifiers to find public keys, as there’s no real central store that is needed. Second, DNS is an infrastructure IT knows how to deploy. Third, signature verifiers can choose which domains they allow requests from. If you do not want to be dependent on DNS, most DKIM frameworks allow you to define your own mechanisms for discovering public keys.


Right now, support for DKIM in the Java world is quite limited. The RESTEasy framework does have an API, though, if you’re interested in using it.


#### JOSE JWS


JOSE JSON Web Signature is a self-contained signature format that contains both the message you want to sign as well as the digital signature of the message.[^18] The format is completely text-based and very compact. It consists of three Base 64–encoded strings delimited by a **.** character. The three encoded strings in the JOSE JWS format are a JSON header describing the message, the actual message that is being transmitted, and finally the digital signature of the message. The media type for JOSE JWS is **application/jose+json**. Here’s what a full HTTP response containing JWS might look like:



```
HTTP/1.1 200 OK
Content-Type: application/jose+json


eyJhbGciOiJSUzI1NiJ9
.
eyJpc3MiOiJqb2UiLA0KICJleHAiOjEzMDA4MTkzODAsDQogImh0dHA6Ly9leGFt
cGxlLmNvbS9pc19yb290Ijp0cnVlfQ
.
cC4hiUPoj9Eetdgtv3hF80EGrhuB__dzERat0XF9g2VtQgr9PJbu3XOiZj5RZmh7
AAuHIm4Bh-0Qc_lF5YKt_O8W2Fp5jujGbds9uJdbF9CUAr7t1dnZcAcQjbKBYNX4
BAynRFdiuB--f_nZLgrnbyTyWzO75vRK5h6xBArLIARNPvkSjtQBMHlb1L07Qe7K
0GarZRmB_eSN9383LcOLn6_dO--xi12jzDwusC-eOkHWEsqtFZESc6BfI7noOPqv
hJ1phCnvWh6IeYI2w9QOYEUipUTI8np6LbgGY9Fs98rqVt5AXLIhWkWywlVmtVrB
p0igcN_IoypGlUPQGe77Rw
```


Let’s break down how an encoded JWS is created. The first encoded part of the format is a JSON header document that describes the message. Minimally, it has an **alg** value that describes the algorithm used to sign the message. It also often has a **cty** header that describes the **Content-Type** of the message signed. For example:


```json
{
   "alg" : "RS256",
   "cty" : "application/xml"
}
```


The second encoded part of the JWS format is the actual content you are sending. It can be anything you want, like a simple text mesage, a JSON or XML document, or even an image or audio file; really, it can be any set of bytes or formats you want to transmit.


Finally, the third encoded part of the JWS format is the encoded digital signature of the content. The algorithm used to create this signature should match what was described in the header part of the JWS message.


What I like about JOSE JWS is that it is HTTP-header-friendly. Since it is a simple ASCII string, you can include it within HTTP header values. This allows you to send JSON or even binary values within an HTTP header quite easily.



### Encrypting Representations


While you can rely on HTTPS to encrypt your HTTP requests and responses, I noted earlier that you may have some scenarios where you want to encrypt the HTTP message body of your requests and responses. 


Specifically, consider scenarios where you are sending messages to a public or untrusted intermediary. While there are a few standard ways to encrypt your representations, my favorite is JOSE JSON Web Encryption.[^19]


JWE is a compact text format. It consists of five Base 64–encoded strings delimited by a **.** character. The first encoded string is a JSON header describing what is being transmitted. The second encoded string is an encrypted key used to encrypt the message. The third is the initialization vector used to encrypt the first block of bytes. The fourth is the actual encrypted messsage. And finally, the fifth is some extra metadata used to validate the message. The media type for JOSE JWE is **application/jose+json**. So here’s what a full an HTTP response containing JWE might look like:


```
HTTP/1.1 200 OK
Content-Type: application/jose+json

eyJhbGciOiJSU0ExXzUiLCJlbmMiOiJBMTI4Q0JDLUhTMjU2In0.
UGhIOguC7IuEvf_NPVaXsGMoLOmwvc1GyqlIKOK1nN94nHPoltGRhWhw7Zx0-kFm
1NJn8LE9XShH59_i8J0PH5ZZyNfGy2xGdULU7sHNF6Gp2vPLgNZ__deLKxGHZ7Pc
HALUzoOegEI-8E66jX2E4zyJKx-YxzZIItRzC5hlRirb6Y5Cl_p-ko3YvkkysZIF
NPccxRU7qve1WYPxqbb2Yw8kZqa2rMWI5ng8OtvzlV7elprCbuPhcCdZ6XDP0_F8
rkXds2vE4X-ncOIM8hAYHHi29NX0mcKiRaD0-D-ljQTP-cFPgwCp6X-nZZd9OHBv
-B3oWh2TbqmScqXMR4gp_A.
AxY8DCtDaGlsbGljb3RoZQ.
KDlTtXchhZTGufMYmOYGS4HffxPSUrfmqCHXaI9wOGY.
9hH0vgRfYgPnAHOd8stkvw
```


Like JSON Web Signatures, the encoded header for JWE is a simple JSON document that describes the message. Minimally, it has an **alg** value that describes the algorithm used to encrypt the message and a **enc** value that describes the encryption method. It often has a **cty** header that describes the **Content-Type** of the message signed. For example:


```json
{
   "alg":"RSA1_5",
   "enc":"A128CBC-HS256",
   "cty" : "application/xml"
}
```


The algorithms you can use for encryption come in two flavors. You can use a shared secret (i.e., a password) to encrypt the data, or you can use an asymmetric key pair (i.e., a public and private key).


As for the other encoded parts of the JWE format, these are really specific to the algorithm you are using and something I’m not going to go over.


As with JWS, the reason I like JWE is that it is HTTP-header-friendly. If you want to encrypt an HTTP header value, JWE works quite nicely.

---
[^16] For more information, see http://dkim.org

[^17] For more information, see the DomainKeys Security Tagging.

[^18] For more information, see the JSON Web Signature.

[^19] For more information, see [the JSON Web Encryption](http://tools.ietf.org/html/draft-ietf-jose-json-web-encryption-14).