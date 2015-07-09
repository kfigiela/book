# Category Oriented Programming

We introduce the Category Oriented Programming paradigm. You can think of it like about a generalization of Object Oriented Programming mixed with purely Functional one in a world of dependent types. 

Category Programming paradigm bases on the concepts of `objects` and `categories`. The former are immutable data structures, while the later are used to classify objects based on their properties and behaviors. Objects and categories allow expressing composite type relations including all forms of algebraic data-types. Formally objects are product types, while categories can be used to build sum types. Category Programming paradigm disallows any form of inheritance, using composition as its basic data building mechanism.

A good example are the integer numbers. In most programming languages `7` would be considered a value of either `Int` type or a polymorphic one, determined by the compiler from a specific usage scenario. In the Category Oriented Programming `7` is both a value as well as the type. It belongs to various categories, including the category of integer numbers `Int` or the category of real numbers `Real`. In fact `7` belongs to infinite number of categories, like a category of possible day numbers `DayNumber`.

The Category Programming paradigm is the basic paradigm in the Luna language. This documentation was written to describe and explain all the concepts in an easy and accessible way for wide range of people.

##Comparison to Object Oriented Programming

The classical OOP was widely criticized for a number of reasons, including not meeting its stated goals of re-usability and modularity.  In classical OOP the inheritance mechanism can be used to arrange objects in a hierarchy that represents "is-a-type-of" relationships. This relationship cannot be easily extended and easily leads to complex code. Morover it introduces ambiguity in situations such as the well known "diamond problem", where it may be ambiguous as to which parent class a particular feature is inherited from.

On the other hand, OOP languages often provide mechanism called interfaces, allowing grouping objects by some common properties and behaviors. While the idea behind interfaces seems similar to the core concept of Category Programming Paradigm, interfaces in classical meaning are much more restricted and do not provide sufficient mechanism to completely replace inheritance.