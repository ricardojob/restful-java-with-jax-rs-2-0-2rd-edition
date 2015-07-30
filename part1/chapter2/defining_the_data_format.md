# Defining the Data Format


One of the most important things we have to do when defining a RESTful interface is determine how our resources will be represented over the wire to our clients. XML is perhaps one of the most popular formats on the Web and can be processed by most modern languages, so let’s choose that. JSON is also a popular format, as it is more condensed and JavaScript can interpret it directly (great for Ajax applications), but let’s stick to XML for now.


Generally, you would define an XML schema for each representation you want to send across the wire. An XML schema defines the grammar of a data format. It defines the rules about how a document can be put together. I do find, though, that when explaining things within an article (or a book), providing examples rather than schema makes things much easier to read and understand.


### Read and Update Format


