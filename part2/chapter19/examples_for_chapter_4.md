# Chapter 19. Examples for Chapter 4


[Chapter 4](../../part1/chapter4/http_method_and_uri_matching.md) discussed three things. First, it mentioned how the **@javax.ws.rs.HttpMethod** annotation works and how to define and bind Java methods to new HTTP methods. Next, it talked about the intricacies of the **@Path** annotation, and explained how you can use complex regular expressions to define your application’s published URIs. Finally, the chapter went over the concept of subresource locators.


This chapter walks you through three different example programs that you can build and run to illustrate the concepts in [Chapter 4](../../part1/chapter4/http_method_and_uri_matching.md). The first example uses **@HttpMethod** to define a new HTTP method called PATCH. The second example expands on the customer service database example from [Chapter 18](../chapter18/build_and_run_the_example_program.md) by adding some funky regular expression mappings with **@Path**. The third example implements the subresource locator example shown in Full Dynamic Dispatching in [Chapter 4](../../part1/chapter4/http_method_and_uri_matching.md).


