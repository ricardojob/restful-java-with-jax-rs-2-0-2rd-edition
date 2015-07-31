# JAX-RS and Conneg


The JAX-RS specification has a few facilities that help you manage conneg. It does method dispatching based on **Accept** header values. It allows you to view this content information directly. It also has complex negotiation APIs that allow you to deal with multiple decision points. Let’s look into each of these.


### Method Dispatching


In previous chapters, we saw how the **@Produces** annotation denotes which media type a JAX-RS method should respond with. JAX-RS also uses this information to dispatch requests to the appropriate Java method. It matches the preferred media types listed in the **Accept** header of the incoming request to the metadata specified in **@Produces** annotations. Let’s look at a simple example:


```Java
@Path("/customers")
public class CustomerResource {

   @GET
   @Path("{id}")
   @Produces("application/xml")
   public Customer getCustomerXml(@PathParam("id") int id) {...}

   @GET
   @Path("{id}")
   @Produces("text/plain")
   public String getCustomerText(@PathParam("id") int id) {...}

   @GET
   @Path("{id}")
   @Produces("application/json")
   public Customer getCustomerJson(@PathParam("id") int id) {...}
}
```

Here, we have three methods that all service the same URI but produce different data formats. JAX-RS can pick one of these methods based on what is in the **Accept** header. For example, let’s say a client made this request:


```
GET http://example.com/customers/1
Accept: application/json;q=1.0, application/xml;q=0.5
```


The JAX-RS provider would dispatch this request to the **getCustomerJson()** method.


### Leveraging Conneg with JAXB


In [Chapter 6](../chapter6/jax_rs_content_handlers.md), I showed you how to use JAXB annotations to map Java objects to and from XML and JSON. If you leverage JAX-RS integration with conneg, you can implement one Java method that can service both formats. This can save you from writing a whole lot of boilerplate code:


```Java
@Path("/service")
public class MyService {

   @GET
   @Produces({"application/xml", "application/json"})
   public Customer getCustomer(@PathParam("id") int id) {...}
}
```

In this example, our **getCustomer()** method produces either XML or JSON, as denoted by the **@Produces** annotation applied to it. The returned object is an instance of a Java class, Customer, which is annotated with JAXB annotations. Since most JAX-RS implementations support using JAXB to convert to XML or JSON, the information contained within our Accept header can pick which **MessageBodyWriter** to use to marshal the returned Java object.


### Complex Negotiation


Sometimes simple matching of the **Accept** header with a JAX-RS method’s **@Produces** annotation is not enough. Different JAX-RS methods that service the same URI may be able to deal with different sets of media types, languages, and encodings. Unfortunately, JAX-RS does not have the notion of either an **@ProduceLanguages** or **@ProduceEncodings** annotation. Instead, you must code this yourself by looking at header values directly or by using the JAX-RS API for managing complex conneg. Let’s look at both.


#### Viewing Accept headers


In [Chapter 5](../chapter5/jax_rs_injection.md), you were introduced to **javax.ws.rs.core.HttpHeaders**, the JAX-RS utility interface. This interface contains some preprocessed conneg information about the incoming HTTP request:


```Java
public interface HttpHeaders {

   public List<MediaType> getAcceptableMediaTypes();

   public List<Locale> getAcceptableLanguages();
...
}
```

The **getAcceptableMediaTypes()** method contains a list of media types defined in the HTTP request’s **Accept** header. It is preparsed and represented as a **javax.ws.rs.core.MediaType**. The returned list is also sorted based on the “q” values (explicit or implicit) of the preferred media types, with the most desired listed first.


The **getAcceptableLanguages()** method processes the HTTP request’s **Accept-Language** header. It is preparsed and represented as a list of **java.util.Locale** objects. As with **getAcceptableMediaTypes()**, the returned list is sorted based on the “q” values of the preferred languages, with the most desired listed first.


You inject a reference to **HttpHeaders** using the **@javax.ws.rs.core.Context** annotation. Here’s how your code might look:


```Java
@Path("/myservice")
public class MyService {

   @GET
   public Response get(@Context HttpHeaders headers) {

      MediaType type = headers.getAcceptableMediaTypes().get(0);
      Locale language = headers.getAcceptableLanguages().get(0);

      Object responseObject = ...;

      Response.ResponseBuilder builder = Response.ok(responseObject, type);
      builder.language(language);
      return builder.build();
   }
}
```

Here, we create a **Response** with the **ResponseBuilder** interface, using the desired media type and language pulled directly from the **HttpHeaders** injected object.


#### Variant processing


JAX-RS also has an API to dealwith situations in which you have multiple sets of media types, languages, and encodings you have to match against. You can use the interface **javax.ws.rs.core.Request** and the class **javax.ws.rs.core.Variant** to perform these complex mappings. Let’s look at the **Variant** class first:


```Java
package javax.ws.rs.core.Variant

public class Variant {

   public Variant(MediaType mediaType, Locale language, String encoding) {...}

   public Locale getLanguage() {...}

   public MediaType getMediaType() {...}

   public String getEncoding() {...}
}
```


The **Variant** class is a simple structure that contains one media type, one language, and one encoding. It represents a single set that your JAX-RS resource method supports. You build a list of these objects to interact with the **Request** interface:


```Java
package javax.ws.rs.core.Request

public interface Request {

   Variant selectVariant(List<Variant> variants) throws IllegalArgumentException;
...
}
```


The **selectVariant()** method takes in a list of **Variant** objects that your JAX-RS method supports. It examines the **Accept**, **Accept-Language**, and **Accept-Encoding** headers of the incoming HTTP request and compares them to the **Variant** list you provide to it. It picks the variant that best matches the request. More explicit instances are chosen before less explicit ones. The method will return null if none of the listed variants matches the incoming accept headers. Here’s an example of using this API:


```Java
@Path("/myservice")
public class MyService {

   @GET
   Response getSomething(@Context Request request) {

      List<Variant> variants = new ArrayList<Variant>();
      variants.add(new Variant(
                      MediaType.APPLICATION_XML_TYPE,
                      "en", "deflate"));

      variants.add(new Variant(
                      MediaType.APPLICATION_XML_TYPE,
                      "es", "deflate"));
      variants.add(new Variant(
                      MediaType.APPLICATION_JSON_TYPE,
                      "en", "deflate"));

      variants.add(new Variant(
                      MediaType.APPLICATION_JSON_TYPE,
                      "es", "deflate"));
      variants.add(new Variant(
                      MediaType.APPLICATION_XML_TYPE,
                      "en", "gzip"));

      variants.add(new Variant(
                      MediaType.APPLICATION_XML_TYPE,
                      "es", "gzip"));
      variants.add(new Variant(
                      MediaType.APPLICATION_JSON_TYPE,
                      "en", "gzip"));

      variants.add(new Variant(
                      MediaType.APPLICATION_JSON_TYPE,
                      "es", "gzip"));

      // Pick the variant
      Variant v = request.selectVariant(variants);
      Object entity = ...; // get the object you want to return

      ResponseBuilder builder = Response.ok(entity);
      builder.type(v.getMediaType())
             .language(v.getLanguage())
             .header("Content-Encoding", v.getEncoding());

      return builder.build();
   }
```


That’s a lot of code to say that the **getSomething()** JAX-RS method supports XML, JSON, English, Spanish, deflated, and GZIP encodings. You’re almost better off not using the **selectVariant()** API and doing the selection manually. Luckily, JAX-RS offers the **javax.ws.rs.core.Variant.VariantListBuilder** class to make writing these complex selections easier:


```Java
public static abstract class VariantListBuilder {

      public static VariantListBuilder newInstance() {...}

      public abstract VariantListBuilder mediaTypes(MediaType... mediaTypes);

      public abstract VariantListBuilder languages(Locale... languages);

      public abstract VariantListBuilder encodings(String... encodings);

      public abstract List<Variant> build();

      public abstract VariantListBuilder add();
   }
```


The **VariantListBuilder** class allows you to add a series of media types, languages, and encodings to it. It will then automatically create a list of variants that contains every possible combination of these objects. Let’s rewrite our previous example using a **VariantListBuilder**:


```Java
@Path("/myservice")
public class MyService {

   @GET
   Response getSomething(@Context Request request) {

      Variant.VariantListBuilder vb = Variant.VariantListBuilder.newInstance();
      vb.mediaTypes(MediaType.APPLICATION_XML_TYPE,
                       MediaType.APPLICATION_JSON_TYPE)
        .languages(new Locale("en"), new Locale("es"))
        .encodings("deflate", "gzip").add();

      List<Variant> variants = vb.build();

      // Pick the variant
      Variant v = request.selectVariant(variants);
      Object entity = ...; // get the object you want to return

      ResponseBuilder builder = Response.ok(entity);
      builder.type(v.getMediaType())
             .language(v.getLanguage())
             .header("Content-Encoding", v.getEncoding());

      return builder.build();
   }
```


You interact with **VariantListBuilder** instances by calling the **mediaTypes()**, **languages()**, and **encodings()** methods. When you are done adding items, you invoke the build() method and it generates a Variant list containing all the possible combinations of items you built it with.


You might have the case where you want to build two or more different combinations of variants. The **VariantListBuilder.add()** method allows you to delimit and differentiate between the combinatorial sets you are trying to build. When invoked, it generates a **Variant** list internally based on the current set of items added to it. It also clears its builder state so that new things added to the builder do not combine with the original set of data. Let’s look at another example:


```Java
Variant.VariantListBuilder vb = Variant.VariantListBuilder.newInstance();
vb.mediaTypes(MediaType.APPLICATION_XML_TYPE,
                   MediaType.APPLICATION_JSON_TYPE)
  .languages(new Locale("en"), new Locale("es"))
  .encodings("deflate", "gzip")
  .add()
  .mediaTypes(MediaType.TEXT_PLAIN_TYPE)
  .languages(new Locale("en"), new Locale("es"), new Locale("fr"))
  .encodings("compress");
```


In this example, we want to add another set of variants that our JAX-RS method supports. Our JAX-RS resource method will now also support **text/plain** with English, Spanish, or French, but only the compress encoding. The **add()** method delineates between our original set and our new one.


You’re not going to find a lot of use for the **Request.selectVariant()** API in the real world. First of all, content encodings are not something you’re going to be able to easily work with in JAX-RS. If you wanted to deal with content encodings portably, you’d have to do all the streaming yourself. Most JAX-RS implementations have automatic support for encodings like GZIP anyway, and you don’t have to write any code for this.


Second, most JAX-RS services pick the response media type automatically based on the **@Produces** annotation and **Accept** header. I have never seen a case in which a given language is not supported for a particular media type. In most cases, you’re solely interested in the language desired by the client. You can obtain this information easily through the **HttpHeaders.getAcceptableLanguages()** method.


### Negotiation by URI Patterns


Conneg is a powerful feature of HTTP. The problem is that some clients, specifically browsers, do not support it. For example, the Firefox browser hardcodes the **Accept** header it sends to the web server it connects to as follows:


```
text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
```


If you wanted to view a JSON representation of a specific URI through your browser, you would not be able to if JSON is not one of the preferred formats that your browser is hardcoded to accept.


A common pattern to support such clients is to embed conneg information within the URI instead of passing it along within an **Accept** header. Two examples are:


```
/customers/en-US/xml/3323
/customers/3323.xml.en-US
```


The content information is embedded within separate paths of the URI or as filename suffixes. In these examples, the client is asking for XML translated into English. You could model this within your JAX-RS resource methods by creating simple path parameter patterns within your **@Path** expressions. For example:


```Java
@Path("/customers/{id}.{type}.{language}")
@GET
public Customer getCustomer(@PathParam("id") int id,
                  @PathParam("type") String type,
                  @PathParam("language") String language) {...}
```


Before the JAX-RS specification went final, a facility revolving around the filename suffix pattern was actually defined as part of the specification. Unfortunately, the expert group could not agree on the full semantics of the feature, so it was removed. Many JAX-RS implementations still support this feature, so I think it is important to go over how it works.


The way the specification worked and the way many JAX-RS implementations now work is that you define a mapping between file suffixes, media types, and languages. An **xml** suffix maps to **application/xml**. An **en** suffix maps to **en-US**. When a request comes in, the JAX-RS implementation extracts the suffix and uses that information as the conneg data instead of any incoming **Accept** or **Accept-Language** header. Consider this JAX-RS resource class:


```Java
@Path("/customers")
public class CustomerResource {

   @GET
   @Produces("application/xml")
   public Customer getXml() {...}

   @GET
   @Produces("application/json")
   public Customer getJson() {...}
}
```

For this **CustomerService** JAX-RS resource class, if a request of **GET /customers.json** came in, the JAX-RS implementation would extract the **.json** suffix and remove it from the request path. It would then look in its media type mappings for a media type that matched **json**. In this case, let’s say json mapped to **application/json**. It would use this information instead of the **Accept** header and dispatch this request to the **getJson()** method.



