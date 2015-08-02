# Deploying Filters and Interceptors


On the server side, filters and interceptors are deployed the same way any other **@Provider** is deployed. You either annotate it with **@Provider** and let it be scanned and automatically registered, or you add the filter or interceptor to the **Application** class’s classes or singletons list.


On the client side, you register filters and interceptors the same way you would register any other provider. There are a few components in the Client API that implement the **Configurable** interface. This interface has a **register()** method that allows you to pass in your filter or interceptor class or singleton instance. **ClientBuilder**, **Client**, and **WebTarget** all implement the **Configurable** interface. What’s interesting here is that you can have different filters and interceptors per **WebTarget**. For example, you may have different security requirements for different HTTP resources. For one **WebTarget** instance, you might register a Basic Auth filter. For another, you might register a token filter.


