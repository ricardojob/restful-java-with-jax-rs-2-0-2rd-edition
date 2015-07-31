# Common Functionality


Each of these injection annotations has a common set of functionality and attributes. Some can automatically be converted from their string representation within an HTTP request into a specific Java type. You can also define default values for an injection parameter when an item does not exist in the request. Finally, you can work with encoded strings directly, rather than having JAX-RS automatically decode values for you. Let’s look into a few of these.


### Automatic Java Type Conversion


All the injection annotations described in this chapter reference various parts of an HTTP request. These parts are represented as a string of characters within the HTTP request. You are not limited to manipulating strings within your Java code, though. JAX-RS can convert this string data into any Java type that you want, provided that it matches one of the following criteria:


1. It is a primitive type. The **int**, **short**, **float**, **double**, **byte**, **char**, and **boolean** types all fit into this category. 
2. It is a Java class that has a constructor with a single **String** parameter. 
3. It is a Java class that has a static method named **valueOf()** that takes a single **String** argument and returns an instance of the class. 
4. It is a **java.util.List&lt;T&gt;**, **java.util.Set&lt;T&gt;**, or **java.util.SortedSet&lt;T&gt;**, where **T** is a type that satisfies criteria 2 or 3 or is a **String**. Examples are **List&lt;Double&gt;**, **Set&lt;String&gt;**, or **SortedSet&lt;Integer&gt;**. 