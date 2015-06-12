#The glorious Luna 1.0 Beta documentation! 

Welcome to the Luna documentation. Please mind that this document is a mix of a tutorial and a formal specification and it often relies on examples and intuitive description. We strongly believe that tutorials are easier to read and understand that adequate formal documents.

###Current state of the art
Please mind that both the official Luna compiler as well as this documentation concern Luna first public release. We have done an enormous amount of work to bring Luna to the reality, however, by no means neither the tool-set nor documentation could be considered to be complete. Its foundations (as described below) are done, however there are many of minor details to be added in.

This specification describes the Luna specification, including syntactic and semantic rules and describes the current status of Luna compiler. You will find some language elements, that are not yet supported by the compiler -- they will be tagged with appropriate information. Please let us know what you like and what needs improvement. This will help us refine the language and decide on priorities of its evolution.

## About this document
This document covers the textual Luna representation and the official Luna compiler ecosystem. The visual representation can be used with the Luna Graphical Environment (LGE) and will be covered in a separate document.











#Types
A type is a label telling which category of objects the expression result fits. We can for example tell that `"Hello!"` is an object from the category containing all possible sentences named `String`.  `5` or `17` are objects in both the category of integer numbers `Int` as well as category of real numbers `Real`. You should think about type like about a set of all possible values under some constrains. Types can overlap, for example every `Int` is also `Real`, but not every `Real` can be used as `Int`.

The type system in Luna, however, was build on completely different basis than the type systems you can find in most (if not all) available languages. We call it the category type system. You will learn about it in detail in the following sections.

##The type system
###Static type system
Luna is a statically typed language, which means that the type of every expression is known at compile time. The type safety leads to safer code and is the foundation for the creation of robust, maintainable systems. It is much better approach than relying on run-time errors, like many languages do nowadays, because a lot of possible errors are caught at compile time. You can be sure that if a program compiles, it does not contain any type based errors, like adding a number and a cake (unless you provide any sensible description how to add 5 to a bun)!

###Dependent types
Luna is a dependent type language. It means, that types and values live in the same space, can be mixed and that types can depend on the values. For example Luna tries to understand how many elements a list stories or what keys are available in a dictionary. In fact Luna was designed to understand as much as possible to provide best in class machinery proving that a particular program will work correctly in any condition. Dependent types are a wide research subject and we are constantly working on extending the possibilities in Luna as much as possible.

###Honesty
Luna's type system is an honest one. This informal term means that the type system does not cheats you. In other words, Luna's type system provides mechanisms for describing the detailed system behavior, including the function purity and possible error occurrence. In Luna you can use type system to indicate that a particular function is pure and does not communicate with any resources external to the program, like all the I/O operations. Such functionality enables Luna to allow many uncommon features, like automatic data parallelism on multicore CPUs and GPUs or support for Software Transactional Memory (STM). There is a great video by Erik Meijer about modern functional language features and the type system honesty: https://www.youtube.com/watch?t=678&v=z0N1aZ6SnBk.

###Type inference
Luna type system provides an inference mechanism. It means that you don't have to explicitly label every piece of code with a type information because the type system can intelligently figure it out for you. In practice you do not have to provide any type information in vast majority of cases, but sometimes it is just handy to do so.

You can ask the compiler about a type of a particular expression using either the `:t` shell command or the `type of` function.

```ruby
>> type of "Hello World"
"Hello World" :: "Hello World"
>> :t 1 == 2
1 == 2 :: False
```

As you can see, Luna infers the most specific type for every expression. If you see something like `a :: "Hello World"`, it means that `a` is a variable of type `"Hello World"` and its only possible value is the string `"Hello World"` itself. In the same fashion, the type of expression `1 == 2` is `False`, which belongs to the category of boolean values `Bool`. It is not always possible to infer types so precisely, in particular when the value depends on some run-time operations, like reading a file from disk. We can easily declare a function, which type would be not so precise:

```ruby
>> def cmpToHello a: "Hello" == a
>> :t cmpToHello
cmpToHello :: String -> Bool
```

The function `cmpToHello` takes single argument and compares it with our favorite string. Luna has inferred the type to be `String -> Bool`, because `a` could be any `String` and we can get as a result both `True` as well as `False`, which forms the category `Bool`.It is important to note, that Luna will try to determine the most precise type when a particular function is used:

```ruby
:t cmpToHello "World"
cmpToHello "World" :: False
```

Even though the type of `cmpToHello` is `String -> Bool`, the type of `cmpToHello "World"` is `False`. You can think about the function `cmpToHello` like about all possible functions from a particular sentence to a boolean value. There is infinite number of such functions, including `"Hello" -> True` or `"World" -> False`.


We can query Luna for all possible values from a particular category using the `variants` method.

```ruby
>> Bool.variants
{True, False}
```

We can also ask Luna if a particular value belongs to a given category using the `in` operator:

```ruby
>> True in Bool
True
>> 2.5 in Int
False
```
It is also possible to ask the compiler about possible value categories. The `categories of` function serves exactly this purpose.

```ruby
>> categories of 2
{Integer, Real, Int, Float}
``` 
Finally, you can query for the "default" category of a particular value:

```ruby

```

You can ask the compiler about a type of a particular expression and use the `:v` shell command or `variant of` function for that reason. Luna provides also the `:t` command and `type of` function, but the way how it works would not be clear enough for now. You will learn about it in the section about variant types.

```ruby
>> variant of "Hello World"
"Hello World" :: String
>> :v 1 == 2
1 == 2 :: Bool
```

So now we see, that the expression `"Hello world"` belongs to the category of `String` while the expression `1 == 2` is `Bool`.


##Type signatures
Explicit type signatures could be useful in few situations, like type level programming, debugging, type inspection, increasing code clarity or in order to limit the range of possible values. Explicit type information are provided after the type annotation operator `::`, which should be read as "belongs to". For example `1 :: Int` should be read as "one belongs to Int". 

Types in Luna are very expressive and they help to create self-documenting code, thus we strongly encourage to use explicit type signatures when defining top-level function or class method. If such signature is missing, Luna warns about it during the compilation. You can turn off the warnings by using the compiler option `- warn.topLevelSignatures`. 

Luna will reject any program with ill-typed expressions:
```ruby
>> a = "word" :: Int
**ERROR** 
Luna interactive 2:13: Couldn't match "word" :: String with expected type Int.
```

The error tells us that the value `"word"`does not belong to the category `Int`, but it belongs to a category `String` instead.


###Partial type signatures
Luna allows to provide partial type information, leaving type checker the inference of the missing part. You can use the wildcard operator `_` to omit information about a particular type in an explicit type signature. A good example would be to annotate a variable being a `Vector` of `Ints`:

```ruby
v1 = Vector 1 2 3 :: Vector Int # ok.
v2 = Vector 1 2 3 :: Vector _   # ok, type inferred.
```

In the second case the Luna compiler would automatically infer the missing part, but not as specific as in the former example. Values `1`, `2` and `3` belong to many categories, so the explicit type signature of `Vector Int` tells Luna, that we mean the `Int` category here.

###Type holes
Sometimes it is helpful to query the compiler about the inferred type of a particular expression. You can use the type holes operator `??` for that purpose:

```ruby
>> "test" :: ??
Luna interactive:1:9:
    Found a type hole with the signature of "test" :: String
```

In the above example Luna inferred the type of `"test"` to be `"test"`, which belongs to the default category of `String`.

####Type aliases 
Sometimes it is just inconvenient to use full type names. Lets consider following situation — we are creating a geometry library and want to store point coordinates in a tuple. If we want to write explicit signature for a point, it would be much easier to write `p :: Point Real` than `p :: (Real,Real,Real)`. In the same fashion it is just easier to write `"Hello World" :: String` than `"Hello World" :: [Char]`. Type aliases are used exactly for this purpose, allowing us to define other names for existing types. To create a type alias, you can use the `typedef` keyword. In fact type definitions allow for defining type level functions, providing much more powerful features than ordinary type alias definitions. They are described in the Type level programming chapter. The standard library defines `String` as follows:
```ruby
def String: [Char]
```

We can use parameterized type aliases, like the mentioned `Point` one:

```ruby
def Point a: (a,a,a)
```

You can use type aliases like regular types, for example:
```ruby
>> def TwoInts: [Int, Int]
>> lst = [1,2] :: TwoInts
```




#Classes and objects

Luna is an Object Oriented language, which means that it bases on the concept of "objects". Please keep in mind, that all the object-related concepts in Luna differ significantly from the classical Object Oriented paradigm.

Object is a collection of attributes and methods. The former contain data, while the later define the object's possible behavior. Everything in Luna is an object, including data structures, functions, types, modules and even previously mentioned code blocks. Objects with similar properties are grouped into classes and are defined by variants of a particular class.


##Algebraic data types
Classes in Luna derive formally from algebraic data types. Algebraic data type is a kind of composite type -- a type formed by combining other types. Two common sorts of algebraic data type are product and sum types, also called records and variants. Both forms are supported in Luna. Algebraic data types can be analyzed and deconstructed using pattern match mechanism.

Although algebraic data types are proven to be a very powerful tool, they are uncommon among programming languages nowadays. This situation is changing however, mainly trough recent increase of interest in functional programming paradigm, where algebraic data types originate from. The formal ideas are very simple, however both the implementation as well as the derivable features differ significantly among programming languages. In fact algebraic data types in Luna are much more flexible and easier to use, than those available for example in Haskell.

##Introduction to classes
Class is a category of objects sharing some properties and behaviors. In the simplest form you can define a new class using the `class` keyword followed by its name and body definition. The following code defines a new `Student` class, containing three fields: `name`, `surname` and `age`.

```ruby
class Student:
    name    :: String
    surname :: String
    age     :: Int

    def sayHello:
        print "Hello! My name is $name."
```
We can can create a new `Student` object the following way:
```ruby
>> s = Student "John" "Doe" 26
>> print s
Student "John" "Doe" 26
>> s.sayHello
"Hello! My name is John."
```

##Variants
Variant is an object definition. But wait a moment! If a class is a category of objects sharing some properties and behaviors, where are the variants of the `Student` class defined? In the example, there is single object definition only. In such situation Luna generates the default object description, so called the default variant. The above code is in fact translated to the following one:

```ruby
class Student:
    Student:
        name    :: String
        surname :: String
        age     :: Int

        def sayHello:
            print "Hello! My name is $name."
```

As you can see, if a class consist of single variant only, Luna allows shorter definition syntax. Every field and method declared in the class level is just copied to the variant definition. If you declare many variants within a single class, Luna copies the class-level definitions to each of them. 

So what does it mean that class contains objects sharing similar properties? To answer that question, lets look at the `Bool` definition from Luna's standard library:

```ruby
class Bool:
    True
    False
```

`True` and `False` are variants, completely distinct object definitions, but they share a common property - they are the only possible values of any Boolean expression. The objects does not carry any information, so their definition is quite simple.

Much more interesting example would be to create custom rendering engine of some primitive shapes. Our renderer would support only two shapes for now, but we can easily extend it later:

```ruby
class Shape:
    Rectangle: width height x=0 y=0 :: Float
    Circle:    radius       x=0 y=0 :: Float
```

We have defined the `Shape` class, that contains all the shapes we are able to render in our toy engine for now. The shapes are `Rectangle` and `Circle` and their object definitions are provided in the code. It is worth noting, that both variants share some position information, but different geometry properties. The position information, in the form of `x` and `y` fields, is provided with default values. All variants should be defined before any other class definitions.

###Constructors
Each variant defines a constructor function. Constructor, like its name suggests, facilitates the construction of objects of that particular variant. In contrast to regular functions, constructors are always generated by the compiler and share their names with corresponding variants. Constructors are the only functions, which names start with an upper-case letter.

For each variant field, there is a distinct input argument required by the constructor. If a field is provided with a default value, the corresponding constructor argument is optional. Let's investigate the type of the `Rectangle` constructor:

```ruby
>> :t Rectangle
Rectangle :: Float Float Float=0 Float=0 => Rectangle
```

The `Rectangle` constructor takes two mandatory and two optional `Float` values and produces a new `Rectangle` object.

```ruby
>> r = Rectangle 200 100
>> print r
Rectangle 200 100 0 0
```


##Classes
In contrast to classical Object Oriented paradigm, classes in Luna do not explicitly shape the relations between objects. You can think about it like about a generalization of OOP model, where there is infinite amount of relations between objects and we choose the ones, that are useful at the moment. The relation between `Rectangle` and `Circle` is not defined anywhere in the code and there is infinite amount of possible relations to choose from, including the fact that both have properties of geometric figures or that they could be faster rasterized on GPU by our brand new renderer.

Luna does not support inheritance, but it allows overlapping classes instead. It means, that objects can belong to many classes at the same time. Classes are just categories of objects in Luna. You have met some of them already, like the integer numbers. Each number is a distinct object, so the standard library defines somehow variants of `1`, `2`, `3`. Every time you write `7`, you are using the constructor named `7` of the variant `7`! There is also a class `Int`, which all the integer numbers belong to. But there is also another class `Real` containing the `Int` one! This is a great example of overlapping categories - every `Int` belongs to `Real` but not vice versa.

Lets go back to our rendering engine. We want to provide some animation support in our library, so our objects could travel a predefined path. The path can be either a rectangle, circle or a segment between two points. We could try to define `Path` the following way:

```ruby
class Path:
    Rectangle: width height x=0 y=0 :: Float
    Circle:    radius       x=0 y=0 :: Float
    Segment:   sx sy ex ey          :: Float
```

Luna will reject the above code, because both `Rectangle` as well as `Circle` were defined earlier in this module, while variants always create new types of objects. But there is a simple way we can tell Luna to use the earlier defined ones:
```ruby
class Path: 
    @Rectangle
    @Circle
    Segment: sx sy ex ey :: Float
```

Now both `Rectangle` as well as `Circle` have the type of both `Shape` and `Path` at the same time. What is even more interesting, we can declare the `Path` using different, more explicit notion. In fact the above definition is automatically converted during the compilation time to the following one:
```ruby
class Segment: sx sy ex ey :: Float
alias Path = Rectangle | Circle | Segment
```

The pipe operator `|` is used to combine classes together. Each object belongs to one or more class and for each object there is implicit declaration of its base class - the one containing objects of this particular variant only. The following code should be read the following way -- `Path` is a class defined as combination of classes `Rectangle`, `Circle` and `Segment`. In other words, `Path` is either a `Rectangle` or `Circle` or `Segment`. The whole example could be easy written using this notion exclusively:

```python
class Rectangle: width height x=0 y=0 :: Float
class Circle:    radius       x=0 y=0 :: Float
class Segment:   sx sy ex ey          :: Float
alias Shape = Rectangle | Circle
alias Path  = Rectangle | Circle | Segment
```

You have met this pattern much earlier in this document. Below you can see a hypothetical `Int` definition:

```ruby
class 0
class 1
...
alias Int = ... | -2 | -1 | 0 | 1 | 2 | ...
```
In fact it is not so far away from the real implementation from the standard library. The `Real` class is just a class of all `Int` and non-integer numbers:
```ruby
alias Real = Int | 0.1 | 0.5 | 1.7 | <other real numbers>
```

Such design is much better than the classical OOP, where we have to define a priori the class hierarchy, which often tends to be hard to refactor and maintain.

###The Class type
The earlier mentioned pipe operator `|` is just a type level function for joining classes together. The resulting combination of possible elements is called the `Class` and is of course an object. The following code snippets are equivalent:
```ruby
alias Path  = Rectangle | Circle | Segment
```
```ruby
alias Path  = Class {Rectangle, Circle, Segment}
```

The `{Rectangle, Circle, Segment}` is a set of types and is used to define the new class of values.

###The Base Class
Each variant has its own so called Base Class. It is a class that accepts objects of that particular variant only.

kazdy variant deklaruje swoj typ i swoja base cateogry zawierajaca tylko wartosci bedace nim samym lub innymi jego wartosciami. (np do Inta mozemy przekzac 1,2,3,.. lub Inta). Przeniesc przyklady i kawalki opisow z kolejnego czapteru
TODO TODO TODO TODO TODO TODO TODO TODO TODO TODO TODO TODO 

##Existential types
Each variant defines a distinct type. That type describes a class of objects of that particular variant type. Luna tracks the usages of every variant and uses this information for better reasoning about the data flow. This is why if you ask Luna about a type of a well defined object, you often get a type that looks exactly or very similar to the investigated expression:
```ruby
:t 5
5 :: 5
:t "hello"
"hello" :: "hello"
:t Rectangle 0 0 200 100
Rectangle 0 0 200 100 :: Rectangle
:t 5
5 :: 5
```

However it is easy to break this by define such sample function:
```ruby
def area shape:
   case shape of
       Rectangle _ _ w h: w * h
       Circle    _ _ r  : pi * r^2
```
If we ask Luna for the inferred type, we will see `area :: Shape -> Float`, because it cannot be further simplified. We do not know yet if the argument will be `Rectangle` or `Circle`, but we know could get any `Float` as a result. In fact we could use smarter types in our shape definition like `Positive Int`, because the diameters cannot be negative. In such case, the type inferencer would deduce that the shape results are non-negative too.

The funny part is, that this mechanism works with any combination of variants, even not related explicitly so far. Lets look at the following function definition:
```ruby
def func x::Int :
    if x>0 then x
           else "oh no"
```
If we ask Luna about its type, we get:
```ruby
:t func
func :: Int -> Int | "oh no"
```
We should read it as follow -- function foo takes a single argument that belongs to the category of `Int` and result could be either any int or the sentence `"oh no"`. Because each possible sentence belongs to the `String` class and is defined by a distinct variant, it has also a distinct type. The type-level `"oh no"`means a class accepting a single objects only -- the string `"oh no"` itself. In fact the type checker could do a better job here, deducing the signature to be `func :: Int -> Positive Int | "oh no"`, but the inference possibilities are strongly compiler implementation dependent and doesn't affect the language features.

The type inference for categories is basically straightforward. If we are provided with a variable and we know that the possible values belong to class `Class {A, B}`, using a method `foo` on this object, we get a result that belongs to a category `Class {A.foo, B.foo}`. Because `{...}` means a set in Luna, if both the classes `A.foo` and `B.foo` would be further inferred to return values from the same category, like for example `String`, the resulting type would be simplified to just `String`.

Some other languages like Haskell allow similar functionality, but much less expressive -- it is usually named `existential types`. It allows working on arbitrary values, that implement a common, predefined interface. The idea is very powerful, but requires explicit interface specification, which is often either unclear or is often changing during a project development and adds some significant maintenance cost. In Luna on the other hand, the type system bases on the idea of categories of related objects, where the relations do not have to be expressed in the explicit way, so there is no difference between casual and existential types in Luna.


##Interfaces
Class variants do not only define new object definitions, they are also some kind of interfaces of some object properties. There could be other objects, that do not belong to this variant, that would have very similar behavior. Sometimes it is desirable to be able to pass to a function any object, that implements some specification. For this reason Luna provides the interface mechanism

```ruby
class Foo:
    def ala: show + "!"
alias Fooable = interface Foo # interface jest przeciazona funkcja mogaca pobierac klasy oraz bloki deklarujace
```



####Constructor types
Each constructor defines a distinct Luna type, while class defines a variant type of all of its constructors. You will learn about variants in the following chapters.

Because each constructor produces objects with distinct types, Luna can track their usage and check for any improper attributes access. A good example would be to create earlier declared `Rectangle` object and query it for the attribute `radius`:

```ruby
>> rec = Rectangle 0 0 4 5
>> :t rec
rec :: Rectangle
>> rec.radius
**ERROR**
Luna interactive 4:13: Rectangle has no attribute `radius`.
```

Please note, that Luna reports a compile-time error about missing or misspelled attributes. It is possible because of the underlying variant type system. In many languages however, such behavior would be treated as a run-time error. A good example would be Haskell, where constructors create instances of the same, basic type.


##Polymorphism
It is possible to declare polymorphic classes by providing class parameters. Parameters are type variables, that can be referenced inside of the class body definition. You can think of polymorphic classes like about class templates. They can contain fields and methods, whose types depend on the parametrization.

Parameters are white space separated type variable identifiers placed after the class name. The following code creates a new class definition `Vector` containing three fields `x`, `y` and `z` of the same parametric type `a`.
```ruby
class Vector a:
    x :: a
    y :: a
    z :: a
```
Luna provides a shorter syntax for declarations of parameters of the same type. You can just write a comma separated list of names provided with a type information:

```ruby
class Vector a:
    x,y,z :: a
```

 Now you can create instances of this record
```ruby
def main:
    v = Vector 1 2 3
    print variant of v
```

The above code produces the following output:

```ruby
Vector Int
```

##Variant types
Sum types are uncommon in popular programming language nowadays. You can find some languages supporting this f The values of a sum type are grouped into several variants. A value of a variant type can be created using a variant constructor. Each variant has its own constructor, which takes a specified number of arguments with specified types. 
Lets start with a very simple sum type and see, how the `Bool` type is defined in Luna's standard library.
```ruby
class Bool: True
            False
```

That's it! The `True` and `False` are the constructors of the type `Bool`. You can think about constructors just like about functions with the difference, that their names are always upper-case. If the constructors are just like ordinary functions, we can use them the same way. So we can now construct the `Bool` type!
```ruby
>> print True
Ture
>> True && False
False
```

Wait! How the `&&` function works? It is also defined in the standard library just like:
```ruby
def a && b: a.&& b
```
You know that pattern, doesn't you? Tthe code `True && False` is evaluated to `True.&& False`, so there must be defined Bool's method `&&` somewhere! It is, in the standard library. In fact, the `Bool` class defines some methods. Let's look at it!
```ruby
class Bool:
    True, False
    
    def && v:
        if self == v then True
                     else False
```

Simple, isn't it? You have just learned the `if then else` function, be happy! And yes, `if then else` is a function, you will learn more about it in the following chapters.
Default constructors
So how the `Vector` class works? We haven't defined any constructor, yet we were able to construct the data type. If we do not provide any explicit constructor, Luna creates default, implicit one with the same name as the type name. According to our example, the following code snippets are equal:
```ruby
class Vector a:
    x,y,z :: a

class Vector a:
    Vector: x,y,z :: a
```




##Polymorphism
Luna incorporates an easy to use, yet very powerful polymorphism mechanisms. All lower-case identifiers in a type signature denote type variables. Type variables are just like normal variables, but they live in the type-level space and can be assigned with any type-level values, affecting the possible values.

We can observe polymorphic types using for example the `Maybe a` type. `Maybe a` type can have only two values `Just a` and `Nothing`. It could be useful when declaring a value that can not be available in run-time.

```ruby
x = Nothing
>> :v x
x :: Maybe a
```

```ruby
>> y = Just 5
>> :v y
y :: Maybe Int
```

The variable `x` is assigned with `Nothing`, so the compiler infers that its type is `Maybe a`. The `a` is a polymorphic type variable, which could be evaluated in later expressions. The `Just 5` value, on the other hand, can be fully inferred with `Maybe Int` type.

You should be careful when digging into Luna, not to confuse variables and type variables in a type signature. Basing on our previous example:

```ruby
>> a = Nothing
>> :t a
a :: Maybe a
```
Let's focus on the last line: `a :: Maybe a`. On the left side of type annotation operator there is an `a` variable. On the other hand, there is also type variable `a`  on the right side of the operator, but keep in mind that these are completely distinct variables. The former one indicates a value, while the later a type. 

Luna is a dependent type language, so it is possible to mix variables and type variables. You will learn how to use this machinery in later chapters.
 
 --------
There is however one important thing to keep in mind. Luna will reject any explicit type signature, that is not as specific as possible:
```ruby
>> lst = [] :: x
**ERROR**
Luna interactive 1:12: Provided type `x` is not as specific as possible: `[]`.
```

###Type contexts
It is not always possible to express all possible type signatures using only concrete types and type variables. Imagine a function, which takes a parameters of type `a` and calls a method `foo` on it. It is not sufficient to tell, that the signature of this function is `func :: a -> b`, because we know more about the type relation. For this purpose Luna introduces a mechanisms called type contexts. A type context is a comma separated list of dependencies between type variables, called constraints. You can use the context operator `|` to introduce explicit constrains. Type contexts are a very powerful mechanism allowing type-level programming and providing a detailed debugging machinery.

Luna supports two types of contexts -- the primary and secondary one. The former introduces user defined constrains, while the later describes all the type variables dependencies resulting directly from the particular function definition. In most cases you will only need to provide the primary context if you want to introduce some additional type relations, while the secondary one would be automatically inferred from the definition. If omitted, Luna always infers the primary context to be empty. The general pattern for a type signature looks like `<signature> | <primary context> | <secondary context>`. Both contexts can be omitted in a type signature. Depending on the context a particular constrain is defined in, we call it a primary or secondary constrain. Luna will reject both any primary constrain that is directly implied by the definition as well as any secondary context that is not.

Lets consider the following function definition:
```haskell
def foo a:
    a.bar 2
```

The full signature for the `foo`function is `foo :: a -> b || b = a.bar Int`. As you can see, the primary context is inferred to be empty, while the secondary consist of all the type dependencies resulting from the implementation.

The following sections describe all the building rules for contexts. You will learn more about their usage in various places across the document, including sections concerning type functions, interfaces or monads to name few.

####Constrains
Luna supports two types of context constrains -- equality and application one. You can combine them to express any possible type relation.

######Equality constrains
Equality constrains allow constraining values of two type expressions to the same type. In the most primitive form, we can set equality constrain between two type variables using the equality constrain operator `=`, for example to constrain elements of tuple to have the same type, we can use the following type description: `t :: (a,b) | a = b`. Such simple use case does not make much sense though, because it is the same as writing `t :: (a,a)`. 

#####Application constraints
Application constrains allow expressing that two types depend on each other, because one is a result of a method call on the other. The application constrain syntax is very simmilar to normal function calls in Luna. Lets focus on the example from the beginning of this section. We can express the full type of `foo`  using explicit secondary context as follow `func :: a -> b || a.foo = b`. It means, that this function takes any value of type `a` and results value of type `b`, where `b` is a type of value resulted from calling method `foo` from class of type `a`.

###The undefined value
Luna standard library provides a very unique value `undefined`. If we ask about its type, we can see, that it can be used as any type in general.

```ruby
>> typeOf undefined
undefined :: a
```

Formally, we call it the bottom value. If Luna program evaluates the undefined value, it results in run-time error. The `undefined` value can be very usefull while prototyping some functionality. You can for example create a function with a specific signature and use `undefined` to define its body, if you plan to define it later. You can think about `undefined` as about a placeholder, which gives exception when evaluated. In fact `undefined` is implemented using the `except` function.

Because Luna is a lazy language, you can use undefined values in computations, that does not evaluate them:
```ruby
>> lst = [undefined, undefined] 
>> print lst.length
2
>> print lst
**EXCEPTION**
Luna interactive 4:7:
   Undefined value
```

You should not use `undefined` in any final, production ready code. Exceptions in Luna are used very rarely and there exist much better way to handle errors. The detailed explanation and description of how to handle errors and exceptions is provided later in the chapter about exceptions and errors.



##Variant types
The type system in Luna hovewer was build on completely different basis than the type systems you can find in most (if not all) available languages. We call it the variant type system. It is a generalization of algebraic data types, existential types and compositable algebraic datatypes. 

##Type conversion
Luna provides safe value conversion utilities between ... TODO

##Higher type universes
Luna provides many advanced type level programming mechanisms. In fact types can be combined into type expressions in the same fashion as values can be combined into expressions.

###Type kinds
Kind is a type of a type. As with regular values, you can provide explicit kind signature for types using the kind annotation operator `::`. The most basic kind is `*`, which means that the type is a fully-applied one without any further restrictions. You can inspect the kind of any type using the Luna interactive shell and the `:kind` or `:k` command:
```ruby
>> :k Int
Int :: *
>> :k []
[] :: * -> *
```

The `* -> *` kind means, that `[]` is a type level function and needs one more argument to be a fully applied type. You can think about it like about a type constructor, that need some arguments to become a type. Type level functions are just like regular functions, but they are feed by types instead of values. Only fully applied types can describe values in Luna. You cannot have a value with type `[] :: * -> *`, but you can have one with type `[a] :: *` for example. 

Luna supports both type level as well as kind level programming. You can also use kinds in order to set some type level restrictions, they are called data kinds. You will learn more about it in the Type level programming chapter.

##Type defaulting

#The standard library
This chapter describes both the primitive types in Luna, which belongs to the specification of the language, as well as popular data types from the official Luna standard library implementation.
TODO ...

Luna ships with a rich set of basic data types, including lists, arrays, dictionaries, arbitrary and fixed precision integers, and a floating-point numbers.

###Primitive types
A primitive type is delivered, not implemented by Luna. This section is an overview for some common ones.

####Int
Int represents an fixed-precision integer value with at least the range $[-2^{29} ... 2^{29}]$. The exact range is not currently defined and depends on underlying machine architecture.

Luna provides several ways to express `Int` literals, following the common conventions from other languages:
```ruby
100 # one hundred
0xbad # hexadecimal
0777 #octal
0b10101011 #binary literal
```

####UnboundInt
`UnboundInt` is like `Int`, but it's not bounded, so can be used to represent even very big numbers. You should keep in mind, that `Int` is more efficient though.
```ruby
>> def factorial n::UnboundInt: 
>>     product [1..n]
>> factorial 50
30414093201713378043612608166064768844377641568960512000000000000  
```
####Real
`Real` represents real numbers in the best possible precision. It is defined just as a type alias to `Double`.

####Float
Single-precision floating point numbers. It is desirable that this type be at least equal in range and precision to the IEEE single-precision type.

####Double
Double-precision floating point numbers. It is desirable that this type be at least equal in range and precision to the IEEE double-precision type.

Float and double literals use the dot `.` as a decimal mark:
```ruby
0.250  #Float literal
```

####Char
Char type represent a single character  — a unicode code point.
The Char literal is typically expressed as a character enclosed in single quotes ' ', like `'a'`, `'ä'`, `'ф'` or `'漢'`. It is also allowed to escape several characters giving them special meaning:
```ruby
'\n' # new line (platform independent LF)
'\r' carriage return
'\t' #tabulator
'\"' #double quotes
'\'' single quotes
'\\' # the backslash itself
```

####Tuples
Tuple, similarly to a list, is a sequence of values. There are however two big differences:

- tuples can contain elements of different types
- tuples contains a fixed number of elements; that number is a part of tuple type

You can think of tuples like about type level lists. Tuple types are denoted by a comma separated list of types that is enclosed with parenthesis. Examples follow:

```ruby
()                  # an empty tuple
(Int)               # WARNING: This is not a tuple!
(Int, String, Int)  # tuple with Int, String, Int
([Int], String)     # tuple with a list of Int elements and a String
```

It is worth noting, that there are no single-element tuples in Luna. This design decision bases on the formal definition of tuples. Tuple is just a set of elements, which determine the tuple type. Single element tuple will have its type determined only by this element, so its type will be equal to type of that element. In other words, single element tuple is nothing more than just its element, which is concise with the syntax provided by Luna.

Tuple literals are created similarly, just fill the parenthesis with values instead of type names:

```ruby
a = (5, 5.0, "five") :: (Int, Float, String)
```

###Basic types

####Bool
A boolean type. It can only have two values — `True` and `False`.

###Lists 
A list is a sequence of elements of a single type, called the element type. The number of elements is called the length -- it is representable by a value of type `Int` and is never negative.
List can be either finite or infinite. Luna provides support for the list literals -- they have a form of a comma-separated elements sequence enclosed with brackets.

```ruby
[] #an empty list
[1, 2, 3, 4, 5] :: [Int] #list of 5 elements of Int type
```

The double dot (`..`) operator allows expressing the linear sequence in lists:

```ruby
`[N..M]` means sequence
[N, N+1, …, M] when M > N
[N, N-1, …, M] otherwise
`[N..]` means infinite sequence [N, N+1, N+2, …]
`[N..M..]` means infinite sequence increasing by M-N
```

Here are few examples:
```ruby
[1..5]             # equivalent of [1,2,3,4,5]
[1..3, 11, 15..12] # equivalent of [1,2,3,11,15,14,13,12]
[1..3..]           # non-negative odd numbers [1,3,5,..]
[1,3..]            # [1,3,4,5,6,7 ...]
```

Two lists can be concatenated using the binary `+` operator. Appending an element to a list requires to traverse the whole list and it works in time `O(n)`.

```ruby
a = [1,2] + [3,4] + [5,6]
```

It's worth keeping in mind that `+` operator is right-associative, so the above concatenation will traverse all of the elements only once!

The binary `|` operator allows prepending element to the list and it is a very fast operation `O(1)`.

```ruby
0 | [1,2,3]   # equivalent to [0,1,2,3,4]
```


####Strings
A string is a list of Unicode characters. Strings can be spanned across multiple lines. All three popular line-endings (`CR`, `CRLF`, `LF`) are treated as a single `LF` byte, so multi-line strings behave always the same across the platforms. String is implemented using Luna list type, allowing at the same time all list-specific operations to be performed, including concatenation, length checking or index-based character accessing. Luna supports two type of string literals - regular and raw ones.

#####Regular String literal
Regular String literals are expressed as a char sequence enclosed in quotes `" "`. Escaping rules as described in Char section apply. 
```ruby
"Hello world!\n"
"Привет!"
"Lunaが最高！"
```

######String expressions
Regular String literals allow special syntax for accessing values from surrounding scope. You can either use the dollar operator `$` as a prefix to a variable name or use any Luna expression inside braces prefixed with dollar:
```ruby
>> myName = "Joe"
>> names = ["Tim", "John"]
>> txt = "Hello! My name is $myName. 
>> These ${names.length} guys are ${names.join "and"}.
>> print txt
"Hello! My name is Joe.
These 2 guys are Tim and John"
``` 

After evaluating every String expression, the `toString` method is called on the result before concatenating with the rest of the text.

#####Raw String literals
Raw string literals are character sequences between triple quotes `""" """`. Within the quotes, any character is legal except sequence of three quotes. The value of a raw string literal is the string composed of the uninterpreted (implicitly UTF-8-encoded) characters between the quotes; in particular, backslashes and dollars have no special meaning. 


#Class extensions
Luna allows us to modify an existing class definition in two forms, by adding a new method definition or by adding a new variant one. The extensions would be available within a module only if were declared inside or imported directly or indirectly as a module dependency. Class extensions could be useful in two scenarios:

 - If a library describes some class of objects and related abstractions, it could extend any existing object definitions to match the abstraction and be included in the class without the need to declare new data types.
 - If we want to add additional logic to a definition placed in external module that is either closed source or that we do not want to change for any other reason.

##Extension methods
Extensions methods 
Czy na pewno chcemy miec nowe metody pojawiajace sie w obiektach gdy cos importujemy? Troche jak classtypy w Haskellu - moze chcemy? W sumie cala stdlib na tym stoi.
Z drugiej storny mamy slaba kontrole przenikania definicji - moze warto wprowadzic obok slowka import slowko using?

```ruby
import Std: Int using Std Foo # Int will have access to extension methods from Std and Foo only
```

##Extension variants
moze niepotrzebny skoro to syntactic sugar:
```ruby
class Foo.Bar: x y z :: Int
# <=>
class Foo:
    @Foo
    Bar: x,y,z :: Int
```
dodatkowo dziala inaczej niz extenoiosn methods poniewaz nie podmienia definicji tylko tworzy nowa

#Functions
Functions in Luna are first class citizens. It means, that you can create unnamed functions, pass functions to other ones or return them as a result. Functions support many features including partial application, default values or named and unnamed arguments. 


##Function type
Functions in Luna also have types. Function type is written as follows: `Parameters -> ReturnType`. Parameters are comma separated list of types within braces. If function accepts only a single argument, braces can be omitted. In Luna every function returns a value. In particular this value can be an empty tuple, which is common pattern for functions that should not return any value. 

```ruby
{Int, Int} -> Int       # function taking two Ints and returning Int
(Int, Int) -> Int       # function taking single tuple of two Ints and returning Int
[String] -> (Int, Int)  # function taking list of Strings and returning tuple of two Ints
[a] -> a                # function taking list of elements of a type and returning a value of that type
[a -> a]                # list of functions taking and returning value of type a
c -> ()                 # function taking argument of any value and returning nothing (empty tuple)
```
The `->` operator has low precedence and is right-associative, taking in consideration a function returning other function, we can show its type like `a -> b -> c` or `a -> (b -> c)`.

If we consider only these functions, which operate on integers, the type of `+` function would be `{Int, Int} -> Int`, which means, that the function takes two arguments, each of type `Int` and results also in value of type `Int`. 

##Function definitions
Functions are defined using the `def` keyword. While defining a function, you can provide the type informations both externally as well as internally in a definition:
```ruby
diagonal :: {Real, Real} -> Real
def diagonal a b: sqrt $ a^2 + b^2
```

is equivalent to:
```ruby
def diagonal a::Real b::Real -> Real: sqrt $ a^2 + b^2
```

In both cases you can provide only partial type informations, like:

```ruby
>> def sum a::Real b -> ?: a + b
>> typeOf sum
sum :: {Real, IsReal a} -> Real
```

The `IsReal a` type is a interface type. It just tells that the second argument is anything that has a method `toReal` converting the value to `Real` type. The same way works the string concatenation function:
```ruby
>> def concat a::String b: a + b
>> typeOf concat
concat :: {String, IsString a} -> String
>> concat "foo" "bar"
"foobar"
>> concat "foo" 5
"foo5"
```

We were able to concat `"foo"` and `5` because Luna knows how to convert the `Int` to `String`. We will learn more about interface types in the future.


##Keyword arguments
##Default arguments
##Partial application
##Function piping
##Function composition



##Data wrappers
Data wrappers are just like type aliases, but they produce new types wrapping the underlying ones. They allow us to wrap values to create safer code. A good example could be the length wrapper:
```ruby
wrapper Length = Length Real
```
We can now create values with type `Length`. Having such value we can be sure, we will not for example multiply it with other `Real` value by mistake, because the `Length` type have no multiplication defined. Ok, so what can we do with such value? We can define methods for it using the extension methods mechanism or we can just "inherit" some methods implementations using the interfaces!
```ruby
wrapper Length = Length Real implements Add
```

You will learn more about interfaces in following chapters. Such wrapper definition is equivalent to the following class one:
```ruby
class Length implements Add:
    unwrap :: Real
```

and we can use it as follow:
```ruby
def main:
    width  = Length 15
    height = Length 10
    size   = width + width + height + height
    print size              # Length 50
    test   = width / height # error
```
    


#Expressions
##Pattern matching
Expressions are the basic bricks building the logic of an application.
Variables, aka pattern matching
Creating variables is pretty straight-forward:
```ruby
x = 10
y = x + 5 #y is now 15
```

Even though we haven’t used any types, all variables are statically typed. They receive a type inferred from the expression on the right-hand side of the = operator.
If we want to be sure that variable is of a specific type, we can explicitly declare its type
```ruby
z::Int = y #will compile iff type of y is Int
```

Actually, it is a part of a bigger mechanism. The left-hand side of = operator is called pattern and can be composed of:

- identifier to be bound with a value from the right-hand side
- an underscore character _ (wildcard) to discard value
- tuple of patterns — to match tuple as possibly extract values from it
- type constructor with pattern for each parameter — to extract values of fields from class

Pattern matching matches each pattern on the left hand-side to a value on the right-hand side.
```ruby
x = (1,2,"three")
(_, _, y) = x # y is now String "three"
```

More complex example with a sample type follows:
```ruby
class Point :
    x,y :: Int
 
def mousePos self:
    Point 50 75 #just a sample...
 
def main self :
Point _ yPos  = self.mousePos  #yPos is now 75
```

##Lambdas

#Interfaces
Interfaces allow to use structural subtyping in Luna. An interface is just a set of fields and methods with their optional default implementations.

---

Classes . In type theory classes are defined as implementations of some specific behavior interfaces. Luna follows that rule

##Definitions

Let's look at a very basic interface `Show`.
```ruby
interface Show:
    def show :: String
```

It means, that instances of `Show` should implement a method `show` resulting in a String. All objects in Luna implement `Show` interface by default. Because of that, we can use print function, defined as follows:
```ruby
print :: Show -> ()
def print a:
    puts $ a.show
```

As you can see, we have used the interface name as function argument's type name. It means that we can evaluate the function by providing any value, which implements the Show interface.

##Default methods
```ruby
interface ToString:
    def toString :: String
    # default implementations:
    def toString: self.show
```
The above interface defines default implementation for the method `toString`. The default methods are used only when classes explicitly declare which interfaces they define, so if we create a class `Foo`:
```ruby
>> class Foo
>> Foo.toString
*** Error: Type `Foo` does not have member named `toString`.
```

But if we declare it implements the interface:
```ruby
>> class Foo implements ToString
>> Foo.toString
Foo
```

Please note, that the expression `Foo.toString` constructs new `Foo` value using the `Foo` constructor and calls it's member function `toString`.


##Existential types

#Errors and exceptions
##Data-flow error system
##Error handling

#Monads
##IO / Pure
##Basic monads
###State monad

#Reflection

#Modules
**TODO** write about LUNAPATH and module structure
##Disk structure
##Imports
##Static values
##Module instances
##Meta-modules



#Memory management

#Luna ecosystem
##Components
##Compiler
###Architecture
###Luna RTS
##Repository manager
##Documentation manager
