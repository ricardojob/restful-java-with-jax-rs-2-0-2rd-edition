# Chapter 10. HATEOAS


The Internet is commonly referred to as “the Web” because information is connected together through a series of hyperlinks embedded within HTML documents. These links create threads between interrelated websites on the Internet. Because of this, humans can “surf” the Web for interesting tidbits of related information by clicking through these links with their browsers. Search engines can crawl these links and create huge indexes of searchable data. Without them, the Internet would never have scaled. There would have been no way to easily index information, and registering websites would have been a painful manual process.


Besides links, another key feature of the Internet is HTML . Sometimes a website wants you to fill out information to buy something or register for some service. The server is telling you, the client, what information it needs to complete an action described on the web page you are viewing. The browser renders the web page into a format that you can easily understand. You read the web page and fill out and submit the form. An HTML form is an interesting data format because it is a self-describing interaction between the client and server.


The architectural principle that describes linking and form submission is called HATEOAS. HATEOAS stands for Hypermedia As The Engine Of Application State. It is a bit of a weird name for a key architecture principle, but we’re stuck with it (my editor actually thought I was making the acronym up). The idea of HATEOAS is that your data format provides extra information on how to change the state of your application. On the Web, HTML links allow you to change the state of your browser. When you’re reading a web page, a link tells you which possible documents (states) you can view next. When you click a link, your browser’s state changes as it visits and renders a new web page. HTML forms, on the other hand, provide a way for you to change the state of a specific resource on your server. When you buy something on the Internet through an HTML form, you are creating two new resources on the server: a credit card transaction and an order entry.

