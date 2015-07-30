# Model the URIs


The first thing we are going to do to create our distributed interface is define and name each of the distributed endpoints in our system. In a RESTful system, endpoints are usually referred to as *resources* and are identified using a URI. URIs satisfy the addressability requirements of a RESTful service.


In our object model, we will be interacting with **Orders**, **Customers**, and **Products**. These will be our main, top-level resources. We want to be able to obtain lists of each of these top-level items and to interact with individual items. **LineItems** are aggregated within **Order** objects so they will not be a top-level resource. We could expose them as a subresource under one particular **Order**, but for now, let’s assume they are hidden by the data format. Given this, here is a list of URIs that will be exposed in our system:

```shell
/orders
/orders/{id}
/products
/products/{id}
/customers
/customers/{id}
```


> #### NOTE

> You’ll notice that the nouns in our object model have been represented as URIs. URIs shouldn’t be used as mini-RPC mechanisms and should not identify operations. Instead, you should use a combination of HTTP methods and the data format to model unique operations in your distributed RESTful system.