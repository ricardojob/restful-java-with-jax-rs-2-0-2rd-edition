# Configuration Scopes


If you look at the declarations of **ClientBuilder**, **Client**, **WebTarget**, **Invocation**, and **Invocation.Builder**, you’ll notice that they all implement the **Configurable** interface. Each one of these interfaces has its own scope of properties and registered components that it inherits from wherever it was created from. You can also override or add registered components or properties for each one of these components. For example:


```Java
Client client = ClientBuilder.newBuilder()
                             .property("authentication.mode", "Basic")
                             .property("username", "bburke")
                             .property("password", "geheim")
                             .build();

WebTarget target1 = client.target("http://facebook.com");
WebTarget target2 = client.target("http://google.com")
                          .property("username", "wburke")
                          .register(JacksonJsonProvider.class);
```


If you viewed the properties of **target1** you’d find the same properties as those defined on the **client** instances, as **WebTargets** inherit their configuration from the **Client** or **WebTarget** they were created from. The **target2** variable overrides the username property and registers a provider specifically for **target2**. So, **target2**’s configuration properties and registered components will be a little bit different than **target1**’s. This way of configuration scoping makes it much easier for you to share initialization code so you can avoid creating a lot of extra objects you don’t need and reduce the amount of code you have to write.


