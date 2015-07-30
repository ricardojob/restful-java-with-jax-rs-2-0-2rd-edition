# The Object Model


The object model of our order entry system is very simple. Each order in the system represents a single transaction or purchase and is associated with a particular customer. Orders are made up of one or more line items. Line items represent the type and number of each product purchased.



Based on this description of our system, we can deduce that the objects in our model are **Order**, **Customer**, **LineItem**, and **Product**. Each data object in our model has a unique identifier, which is the integer id property. Figure 2-1 shows a UML diagram of our object model.

![Figure 2-1](Figure 2-1.png "Order entry system object model")

