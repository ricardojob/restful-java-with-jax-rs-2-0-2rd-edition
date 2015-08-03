# OAuth 2.0


OAuth 2.0 is an authentication protocol that allows an entity to gain access to a user’s data in a secure manner without having to know the user’s credentials.[^15] A typical example is a news site like cnn.com. You’re reading an interesting political editorial and want to voice your opinion on the article in its comment section. To do this, though, you have to tell CNN who you are and what your email address is. It gives you the option of logging in via your Google or Facebook account. You are forwarded to Google and log in there. You grant CNN permission to ask Google who you are and what your email address is, and then you are forwarded back to cnn.com so that you can enter in your comment. Through this interaction CNN is granted an access token, which it then uses to obtain information about you via a seperate HTTP request.


Here’s how it works:

1. The CNN website redirects your browser to Google’s login page. This redirect sets a special cnn.com session cookie that contains a randomly generated value. The redirect URL contains **client_id**, **state**, and **redirect_uri**. The **client_id** is the Google username CNN has registered with Google.com. The **state** parameter is the same value that was set in the session cookie. The **redirect_uri** is a URL you want Google to redirect the browser back to after authentication. A possible redirect URL in this scenario thus would be **http://googleapis.com/oauth?client_id=cnn&state=23423423123412352314&redirect_uri=http%3A%2F%2Fcnn.com**.

2. You enter your username and password on Google’s login page. You then are asked if you will grant CNN access to your personal information.

3. If you say yes, Google generates an access code and remembers the **client_id** and **redirect_uri** that was sent in the original browser redirect.

4. Google redirects back to CNN.com using the **redirect_uri** sent by CNN’s initial redirect. The redirect URL contains the original **state** parameter you forwarded along with a **code** parameter that contains the access code: **http://cnn.com/state=23423423123412352314&code=0002222**.

5. With this redirection, CNN will also get the value of the special cookie that it set in step 1. It checks the value of this cookie with the **state** query parameter to see if they match. It does this check to make sure that it initiated the request and not some rogue site.

6. The CNN server then extracts the **code** query parameter from the redirect URL. In a separate authenticated HTTP request to Google, it posts this access code. Google.com authenticates that CNN is sending the request and looks up the access code that was sent. If everything matches up, it sends back an access token in the HTTP response.

7. CNN can now make HTTP requests to other Google services to obtain information it wants. It does this by passing the token in an **Authorization** header with a value of **Bearer** plus the access token. For example:


```
GET /contacts?user=billburke
Host: contacts.google.com
Authorization: Bearer 2a2345234236122342341bc234123612341234123412adf
```


In reality, sites like Google, Facebook, and Twitter don’t use this protocol exactly. They all put their own spin on it and all have a little bit different way of implementing this protocol. The same is true of OAuth libraries. While the core of what they do will follow the protocol, there will be many custom attributes to each library. This is because the OAuth specification is more a set of detailed guidelines rather than a specific protocol set in stone. It leaves out details like how a user or OAuth client authenticates or what additional parameters must be sent. So using OAuth may take a bunch of integration work on your part.


There are many different Java frameworks out there that can help you turn your applications into OAuth providers or help you integrate with servers that support OAuth authentication. This is where I make my own personal plug. In 2013, I started a new project at Red Hat called Keycloak. It is a complete end-to-end solution for OAuth and SSO. It can also act as a social broker with social media sites like Google and Facebook to make leveraging social media easier. Please check us out at http://www.keycloak.org.

---
[^15] For more information, see [the OAuth 2.0 Authorization Framework](http://tools.ietf.org/html/rfc6749).