# Wrapping Up


In this chapter we learned about client- and server-side filters and interceptors. Filters generally interact with HTTP message headers, while interceptors are exclusive to processing HTTP message bodies. Filters and interceptors are applied to all HTTP requests by default, but you can bind them to individual JAX-RS resource methods by using **DynamicFeature** or **@NameBinding**. [Chapter 26](../../part2/chapter26/examples_for_chapter_12.md) walks you through a bunch of code examples that show most of these component features in action.