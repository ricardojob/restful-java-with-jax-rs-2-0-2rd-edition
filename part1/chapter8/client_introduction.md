# Client Introduction


Before I dive into the Client API, let’s look at a simple code example that illustrates the basics of the API:


```Java
Client client = ClientBuilder.newClient();

WebTarget target = client.target("http://commerce.com/customers");

Response response = target.post(Entity.xml(new Customer("Bill", "Burke)));
response.close();

Customer customer = target.queryParam("name", "Bill Burke")
                          .request()
                          .get(Customer.class);
client.close();
```


This example invokes GET and POST requests on a target URL to create and view a **Customer** object that is represented by XML over the wire. Let’s now pull this code apart and examine each of its components in detail.


