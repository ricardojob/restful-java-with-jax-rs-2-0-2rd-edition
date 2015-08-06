# Example ex09_2: Conneg via URL Patterns


[Chapter 9](../../chapter9/http_content_negotiation.md) discussed how some clients, particularly browsers, are unable to use the **Accept** header to request different formats from the same URL. To solve this problem, many JAX-RS implementations allow you to map a filename suffix (*.html*, *.xml*, *.txt*) to a particular media type. RESTEasy has this capability. We’re going to illustrate this using your browser as a client along with a slightly modified version of *ex09_1*.


### The Server Code


A few minor things have changed on the server side. First, we add **getCustomerHtml()** method to our **CustomerResource** class:


```Java
   @GET
   @Path("{id}")
   @Produces("text/html")
   public String getCustomerHtml(@PathParam("id") int id)
   {
      return "<h1>Customer As HTML</h1><pre>"
                   + getCustomer(id).toString() + "</pre>";
   }
```


Since you’re going to be interacting with this service through your browser, it might be nice if the example outputs HTML in addition to text, XML, and JSON.


The only other change to the server side is in the configuration for RESTEasy:


```xml:src/main/webapp/WEB-INF/web.xml
<web-app>

    <context-param>
        <param-name>resteasy.media.type.mappings</param-name>
        <param-value>
                html : text/html,
                txt : text/plain,
                xml : application/xml
        </param-value>
    </context-param>
...
</web-app>
```


The **resteasy.media.type.mappings** content parameter is added to define a mapping between various file suffixes and the media types they map to. A comma separates each entry. The suffix string makes up the first half of the entry and the colon character delimits the suffix from the media type it maps to. Here, we’ve defined mappings between *.html* and **text/html**, *.txt* and **text/plain**, and *.xml* and **application/xml**.



### Build and Run the Example Program


Perform the following steps:

1. Open a command prompt or shell terminal and change to the *ex09_2* directory of the workbook example code.
2. Make sure your PATH is set up to include both the JDK and Maven, as described in [Chapter 17](../chapter17/workbook_introduction.md).
3. Perform the build and run the example by typing **maven jetty:run**.


The **jetty:run** target will run the servlet container so that you can make browser invocations on it. Now, open up your browser and visit *http://localhost:8080/customers/1*.


Doing so will show you which default media type your browser requests. Each browser may be different. For me, Firefox 3.x prefers HTML, and Safari prefers XML.


Next, browse each of the following URLs: *http://localhost:8080/customers/1.html*, *http://localhost:8080/customers/1.txt*, and *http://localhost:8080/customers/1.xml*. You should see a different representation for each. 