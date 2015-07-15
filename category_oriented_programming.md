# Category Oriented Programming

We introduce the Category Oriented Programming paradigm. You can think of it like about a generalization of Object Oriented Programming mixed with purely Functional one in a world of dependent types.

Category Programming paradigm bases on the concepts of `objects` and `classes`, but there is more differences than similarities between these concepts and the ones known from Object Oriented Programming. The former are immutable data structures, while the later are used to classify objects based on their properties and behaviors. Objects and categories allow expressing composite type relations including all forms of algebraic data-types. Formally objects are product types, while categories can be used to build sum types. Category Programming paradigm disallows any form of inheritance, using composition as its basic data building mechanism.

The Category Programming paradigm is the basic paradigm in the Luna language. Categories in Luna are named `classes`, but they serve different purpose than classes known from Object Oriented Programming. In fact both words `category` and `class` are used interchangeable across this book.

###Objects
The concept of objects is analogous to real world objects. Object describes an entity that has a specific properties and behaviors, like a car, a lamp or a book. And rather than interacting with them at a very low level, you interact with them at a very high level of abstraction. You can for example tell a taxi cab object to go to Los Angeles and you do not have to provide detailed information how to do it, because the object knows it. Objects encapsulate complexity, and the interfaces to that complexity are high level.

Formally, objects are data structures containing data in the form of fields. They are associated with methods – functions describing their behaviors. In contrast to classical Object Oriented paradigm, methods are not properties of objects. They are properties of classes while objects can inherit them as needed. Objects in Luna are immutable, which means that after they are created, you cannot change them. You can though create new, updated ones.Everything in Luna is an object, including values, data structures, functions, types, modules and even previously mentioned code blocks.

###Classes
Classes are used to classify objects based on their properties and behaviors. Formally classes derive from Category theory. Each class is a category containing objects and their methods, formally called arrows. You can think of a class like about a set of objects and related transformations. This set can be defined explicitly, like all the even integer numbers or implicitly by behaviors, like all the objects, which can be mapped by a given function. Classes can be composed together to create a new class of objects with common behaviors.

Each object can belong to many classes at the same time. In fact every object in Luna belongs to an infinite number of classes. Some of them are explicitly labeled like `Int`, but some are not like all the even integer numbers. In particular every object belongs to the so called unit-class – a class containing that single object only.

In general, there is an infinite universe of different objects, often with unrelated behaviors. If we do not know anything about a particular object, we cannot use it in any sensible way. Let's take the taxi cab as an example. If we do not know that this particular object is a taxi cab, we cannot ask it to drive us home. We could do another thing instead. We can tell it – if you are a taxi cab, drive us home. This behavior is formally named the structural sub-typing and it is allowed by the type system in Luna.

####Sub-classes and super-classes
There is also a notion of sub-classes and super-classes in Luna. They are just like regular classes, but they operate respectively on sub-sets and super-sets of objects of a given class. Sub-class cannot modify the meaning of the super-class methods, it can only define more precise and additional ones. There is always an infinite amount of super-classes of a given class. Its worth noting, that every class is a super-class and sub-class of itself at the same time. Sub-class relations are represented in Luna by the sub-class operator `::`, which should be read as `belongs to` or `is subclass of`.

A good example are the numbers! In most programming languages `7` would be considered a value of either `Int` type or a polymorphic one, determined by the compiler from a specific usage scenario. In the Category Oriented Programming `7` is both a value as well as the type. It belongs to various classes, including the class of integer numbers `Int` or the class of real numbers `Real`. In fact every integer number belongs to `Real` at the same time, so `Int` is a sub-class of `Real`. This sub-class relation keeps all the behaviors of `Real` numbers in the `Int` class. Let's consider the multiplication! The multiplication of real numbers is described as:

```ruby
* :: Real -> Real -> Real
```

The signature means that the function takes two real numbers and outputs another real one. The above signature describes a class of functions as well. The multiplication over integer values is a subclass of that function, because every time you multiply `4` by `5` you get `20`, no matter if the factors were originally considered real or integer numbers. The `Int` class defines more specific version of multiplication, which given two integer numbers results in an integer one:

```ruby
* :: Int -> Int -> Int
```

However, it does not affect the result in any way, only its classification. We can express these relation as:

```ruby
Int :: Real
Int -> Int -> Int :: Real -> Real -> Real
```

The former one should be read as `Int is subclass of Real`. Unfortunately these relations are not alway true, especially when we are talking about how numbers are stored in a computer memory. Nevertheless the type system in Luna does not suffer from this fact and such relations are defined in the Luna's standard library.


###Types
A class an object inherits behavior from is called its type. Sub- and super-classes of that class are called sub- and super-types respectively. Every object has a single, specific type. Luna allows to use a particular value whenever a super-type is required, for example you can always use an integer number when a real one is needed.


#### Type universes
Everything in Luna has a type. Even types have types and the types of types also have types. In fact every type has an infinite chain of other types, that it belongs to.  Of course you can define such type by yourself!

Lets investigate it using the Luna Shell! You can query the compiler for a type of a particular expression using either the `type of` function or the more verbose `:t` shell command.

```ruby
λ: :t 2
   2 :: 2 :: Int

λ: :t Int
   Int :: Real

λ: :t Real
   Real :: Complex

λ: :t Complex
   Complex :: Number

λ: :t Number
   Number :: *

λ: :t (*)
   * :: *
```

Oh, that is really interesting! As we can see, each type has it's own type and this type hierarchy can be defined recursively. We can display the whole type dependency line using the `types of` function:

```ruby
λ: types of 2
   2 :: Int :: Real :: Complex :: Quaternion :: Octonion :: Sedenion :: Number :: * :: ..
```

This type means that every `2` has type `2`. This type has type `Int`, which subsequently has type `Real`. Every `Real` value is also a `Complex` number. The `Complex` type contains both the real as well as imaginary numbers and its type is simply `Number`. The `*` is so called the "star type". It is just a category containing all possible objects and types.

#####The star type
The star type is a very unique one. As we already know, all the objects of a given category have to share similar properties. Because the star type is a category containing all possible objects, the similar property set is empty, so we cannot use any methods on objects of the star type. You can explicitly state that a value is of the star type, but Luna will track its most precise type under the hood. In fact such statement tells only that a particular object is any object, so it is always true.

```ruby
λ: a = 7 :: *
λ: :t a
   a :: 7 :: Int
```

###Dependent types
Luna is a dependent type language. It means that values can be used in the type level and vice-versa – types can be used as values. The syntax for both value- and type-level operations is always the same and you are allowed to use exactly the same functions and objects in both the value and type expressions. Lets consider our favorite number `5`. The following statements are all true:

```ruby
5 :: 5
5 :: sqrt 25
5 :: 2 + 3
5 :: Nat
5 :: Int
5 :: Real
...
```

In the same fashion it is valid in Luna to tell that `Int` is a subset of `Int` – `Int :: Int`. In fact, if a function expects an argument of type `Int` you can provide the `Int` object itself. It means that instead of values, you are running the computations on a class-level. You can think about it like running the computations on all integer numbers at the same time. Luna is wise enough not to naively loop over all the numbers and collect the results, it returns a class of the result instead:

```ruby
λ: 'test' + 1
   'test1'
λ: 1.show
   '1'
λ: String + Int
   String
λ: Int.show
   String
```
This is a very powerful mechanism. It can be used for both type level programming as well as to bulk-process all values from a specific category. Lets assume we've got a heterogeneous dictionary and we want to obtain all the values, whose key is an integer. We can accomplish it by using the `values` method available in every category:

```ruby
λ: dict = {'foo':'bar', 1:2, 3:4}
λ: dict.lookup Int . values
   [2,4]
```

It is also possible to treat a concrete value like its unit-class, so the following code behaves in a similar fashion to the previous one:

```ruby
λ: dict = {'foo':'bar', 1:2, 3:4}
λ: dict.lookup 1 . values
   [2]
```

KF: tu sie pojawia syntactic sugar dla slownikow i tablic – powinien byc gdzies wyjasniony.

You will learn more about dependent types in later chapters of this book. That was just an introduction to the way Category Oriented Programming treats the values and their types.

###The strict type operator
There is yet another type operator in Luna, that is worth mentioning. It is the strict type operator `:::`. It is like `::`, but disallows omitting parts of the type line. Yo can thus write both `2 :: Int` and `2 :: *`, but you cannot `2 ::: *`, because `*` is not the most concrete type of `2`. The strict type operator is very rarely used, but it serves a great tool when debugging the code.

###Comparison to Object Oriented Programming

The classical Object Oriented paradigm was widely criticized for a number of reasons, including not meeting its stated goals of re-usability and modularity. In classical OOP the inheritance mechanism can be used to arrange objects in a hierarchy that represents "is-a-type-of" relationships. This relationship cannot be easily extended and leads to complex and hard-to-maintain code. Moreover, it introduces ambiguity in situations such as the well known "diamond problem", where it may be ambiguous as to which parent class a particular feature is inherited from.

On the other hand, Object Oriented languages often provide mechanism called interfaces, allowing grouping objects by some common properties and behaviors. While the idea behind interfaces seems similar to the core concept of Category Programming Paradigm, interfaces in classical meaning are much more restricted and do not provide sufficient mechanism to completely replace the inheritance.

Category Oriented Programming, on the other hand, introduces much simpler and more general object classification concept, allowing writing clean, elegant and maintainable code easily.
