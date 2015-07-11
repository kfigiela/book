# The type system

A type is a label telling which category of objects the expression result fits. You should think about a type in Luna like about a set of values with similar properties and behaviors. We can for example tell that `"Hello!"` belongs to the category of all possible sentences – `String`. We can use arithmetic operations on numbers, so it is sensible to group them in a common category. The numbers like `5` or `17` belong to both the category of integer numbers `Int` as well as category of real numbers `Real`. In fact every integer number belongs to `Real` at the same time. Such facts are represented in Luna by the type operator `::`, which should be read as "belongs to" or "is subset of". For example `Int :: Real` should be read as "Int is subset of Real". Unfortunetally this relation is not alway true, especially when we are talking about how numbers are stored in a computer memory. Nevertheless the type system in Luna does not suffer from this fact and such relation is defined in the standard library.

The Luna's type system has the following characteristics:

####Static type system
Luna is a statically typed language, which means that the type of every expression is known at compile time. The type safety leads to safer code and is the foundation for the creation of robust, maintainable systems. It is much better approach than relying on run-time errors, like many languages do nowadays, because a lot of possible errors are caught at compile time. You can be sure that if a program compiles, it does not contain any type based errors, like adding a number and a cake (unless you provide any sensible description how to add 5 to a bun)!

####Dependent types
Luna is a dependent type language. It means, that types and values live in the same space, can be mixed and that types can depend on the values. For example Luna understands how many elements a list stories or what keys are available in a dictionary. In fact Luna understand such relations for every user defined datatype without any explicit code. Luna was designed to understand as much as possible to provide best in class machinery proving that a particular program will work correctly under given conditions. Dependent types are a wide research subject and we are constantly working on extending the possibilities in Luna as much as possible.

####Honesty
Luna's type system is an honest one. This informal term means that the type system does not cheats you! In other words, Luna's type system provides mechanisms for describing the detailed system behavior, including the function purity and possible error occurrence. In Luna you can use type system to indicate that a particular function is pure and does not communicate with any resources external to the program, like all the I/O operations. Such functionality enables Luna to allow many uncommon features, like automatic data parallelism on multicore CPUs and GPUs or support for Software Transactional Memory (STM). There is a great video by Erik Meijer about modern functional language features and the type system honesty: https://www.youtube.com/watch?t=678&v=z0N1aZ6SnBk.

# Values and types
Luna type system provides an inference mechanism. It means that you don't have to explicitly label every piece of code with a type information because the type system can intelligently figure it out for you. In practice you do not have to provide any type information in vast majority of cases, but sometimes it is just handy to do so.

You can ask the compiler about a type of a particular expression using either the `:t` shell command or the `type of` function.

```ruby
λ: type of "Hello World"
   "Hello World" :: "Hello World" :: String
λ: :t 1 == 2
   1 == 2 :: False :: Bool
```

As you can see, Luna infers the most specific type for every expression. The output means that `"Hello World"` is a variable of type `"Hello World"`, which belongs to the category `String`. The only value of such type is the string `"Hello World"` itself. In the same fashion, the type of expression `1 == 2` is `False`, which belongs to the category of boolean values `Bool`. It is not always possible to infer types so precisely, in particular when the value depends on some run-time operations, like reading a file from disk. We can easily declare a function, which type would be not so precise:

```ruby
λ: def cmpToHello a: "Hello" == a
λ: :t cmpToHello
   cmpToHello :: String -> Bool
```

The function `cmpToHello` takes single argument and compares it with our favorite string. Luna has inferred the type to be `String -> Bool`, because `a` could be any `String` and we can get as a result both `True` as well as `False`, which forms the category `Bool`.

In fact we can write the type signature for `cmpToHello` more precisely:

```ruby
cmpToHello :: "Hello" -> True | _ -> False
```
But even if we don't, Luna keeps such relation under the hood. It is important to note, that Luna will try to determine the most precise type when a particular function is used:

```ruby
λ: type of (cmpToHello "World")
   cmpToHello "World" :: False :: Bool
```

Even though the type of `cmpToHello` was defined as `String -> Bool`, the type of `cmpToHello "World"` is `False`. You can think about the function `cmpToHello` like about all possible functions from a particular sentence to a boolean value. There is infinite number of such functions, including `"Hello" -> True` or `"World" -> False`.

We can query Luna for all possible values from a particular category using the `values` method.

```ruby
λ: Bool.values
   {True, False}
```

We can also ask Luna if a particular value belongs to a given category using the `in` operator:

```ruby
λ: True in Bool
   True
λ: 2.5 in Int
   False
```

It is also possible to query the compiler about the defined categories of a particular value. The `categories of` function serves exactly this purpose.

```ruby
λ: categories of 's'
   {Char, String}

λ: categories of 'str'
   {String}
```

In Luna every value is associated with a concrete type. Even if the value belongs to many categories one of them defines the behavior of that particular value. Such category is called the type. Luna allows values to change types in a safe and easy way.

The composition of several categories forms a category as well. Such combination could be used as a type, for example a value `a` of type `a :: Int | String` can be either an integer number or a sentence. Such value inherits all the common behaviors of elements from both categories.

## Type universes and the star type
Everything in Luna has a type. Even types have types and the types of types also have types. In fact every type has an infinite chain of other types, that it belongs to. Some of them are explicitly labeled, like `String`, but some are not, like all the even days of the week. Of course you can define such type by yourself! Lets investigate it using the Luna Shell!

```ruby
λ: type of 2
   2 :: 2 :: Int

λ: type of Int
   Int :: Real

λ: type of Real
   Real :: Number

λ: type of Number
   Number :: Complex

λ: type of Complex
   Complex :: *

λ: type of (*)
   * :: *
```

Oh, that is really interesting! As we can see, each type has it's own type and this type hierarchy can be defined recursively. We can display the whole dependency line using the `types of` function:

```ruby
λ: types of 2
   2 :: 2 :: Int :: Real :: Number :: * :: ..
```

This type means, that every `2` has type `2`. This type has type `Int`, which subsequently has type `Real`. Every `Real` value is also a `Number`. There are other numbers than `Real`, like the `Imaginary` numbers. The `Complex` type contains both the real as well as imaginary numbers and its type is simply `Number`. The `*` is so called the "star type". It is just a category containing all possible objects and types.

The star type is a very unique one. As we already know, all the objects of a given category have to share similar properties. Because the star type is a category containing all possible objects, the simmilar property set is empty, so we cannot use any methods on objects of the star type. You can explicitly state that a value is of the star type, but Luna will track its most precise type under the hood. In fact such statement tells only that a particular object is any object, so it is always true.

```ruby
λ: a = 7 :: *
λ: type of a
   a :: 7 :: Int
```

There is yet another type operator in Luna, that is worth mentioning. It is the strict type operator `:::`. It is like `::`, but disallows omiting parts of the type line. Yo can thus write both `2 :: Int` and `2 :: *`, but you cannot `2 ::: *`, because `*` is not the most concrete type of `2`. The strict type operator is very rarely used, but it serves a great tool when debugging the code.

##Explicit type signatures
Explicit type signatures could be useful in few situations, like type level programming, debugging, type inspection, increasing code clarity or in order to limit the range of possible values.

Types in Luna are very expressive and they help to create self-documenting code, thus we strongly encourage to use explicit type signatures when defining top-level function or class method. If such signature is missing, Luna warns about it during the compilation. You can turn off the warnings by using the compiler options `--type.sig.missing.top none` and `--type.sig.missing.class none`. You can also change the behavior to discard programs if any top-level function is not provided explicitly by changing the option to `--type.sig.missing.top error`, but we consider this behavior to restrictive for everyday use.

Luna will reject any program with ill-typed expression:
```erlang
λ: a = "word" :: Int
*: Couldn't match actual type String with expected type Int.
```

The error tells us that the value `"word"` does not belong to the category `Int`, but it belongs to a category `String` instead.


###Partial type signatures
Luna allows to provide partial type information, leaving type checker the inference of the missing part. You can use the wildcard operator `_` to omit information about a particular type in an explicit type signature. A good example would be to annotate a variable being a `Vector` of `Ints`:

```erlang
λ: v1 = Vector 1 2 3 :: Vector Int
λ: v2 = Vector 1 2 3 :: Vector _
λ: type of v1 == type of v2
   True
```

###Type holes
Sometimes it is helpful to query the compiler about the inferred type of a particular expression. You can use the type hole operator `??` exactly for that purpose:

```ruby
λ: a = 'str' + 7
λ: a :: ??
*: Found a type hole with the signature of a :: 'str' :: String
```

In the above example Luna inferred the type of `a` to be `'str'`, which belongs to the `String` category.

Type holes are treated as compilation errors by default. You can change this behavior by using the compiler option `--type.hole warning` to report the type holes as warnings instead.

#Classes and objects

Luna is not an Object Oriented language in a strict meaning. It shares many concepts with classical classical OOP paradigm, but differs significantly in many ways. Everything in Luna is an object, including values, data structures, functions, types, modules and even previously mentioned code blocks.

Object is a collection of attributes. It is also described by a specific behavior in a given category of objects. This behavior is expressed by methods – functions associated with each object. The important fact is that methods are not properties of objects, they are always defined in a context of a category.

A category can be defined in Luna using the `class` function. In contrast to other languages, `class` is a function, not a keyword in Luna. At first glance, classes look very familiar! The following code defines a new `Student` class, containing three fields – `name`, `surname`, `age` and a method `greet`.

```ruby
class Student:
    name    :: String
    surname :: String
    age     :: Int

    def greet:
        print 'Hello! My name is $name.'
```

The field declaration consist always of its name followed by explicit type information. It is possible to declare more than one field within a single line, but then the type signature is applied to every field in that line. So the above definition can be written shorter:

```ruby
class Student:
    name surname :: String
    age          :: Int

    def greet: print 'Hello! My name is $name'

```

###Constructors
Class `Student` declares a new data type. Each data type is associated with so called constructor function. Constructor, like its name suggests, facilitates the construction of objects of that particular variant. In contrast to regular functions, constructors are always generated by the compiler and share their names with corresponding data types. Moreover, constructors are the only functions, which names start with an upper-case letter.

For each data type field, there is a distinct input argument required by the constructor. If a field is provided with a default value, the corresponding constructor argument is optional. Let's investigate the type of the `Student` constructor:

```ruby
λ: type of Student
   Student :: String -> String -> Int -> Student
```

This is a function type. You will learn more about function types later in this book. For now just remember that it means, that we have to provide three arguments of types `String`, `String` and `Int` in order to get object of type `Student`. We can can create a new `Student` objects using the constructor the following way:

```ruby
λ: s = Student "John" "Doe" 26
λ: print s
   Student "John" "Doe" 26
λ: s.greet
   "Hello! My name is John."
```


###Algebraic data types
Classes in Luna derive formally from algebraic data types. Algebraic data type is a kind of composite type – a type formed by combining other types. Two common sorts of algebraic data type are product and sum types, also called records and variants. Both forms are supported in Luna. Algebraic data types can be analyzed and deconstructed using pattern match mechanism.

Although algebraic data types are proven to be a very powerful tool, they are uncommon among programming languages nowadays. This situation is changing however, mainly trough recent increase of interest in functional programming paradigm, where algebraic data types originate from. The formal ideas are very simple, however both the implementation as well as the derivable features differ significantly among programming languages. In fact algebraic data types in Luna are much more flexible and easier to use, than those available for example in Haskell.


###Records and variants
Record defines the object structure. But wait a moment! If a class is a category of objects sharing some properties and behaviors, where is the record definition of the `Student` class? In the example, there is single object definition only. In such situation Luna generates the default record description, so called the default class variant. Each record description of a particular category is called its variant. The previous code is in fact translated to the following one:

```ruby
class Student:
    Student:
        name    :: String
        surname :: String
        age     :: Int

    def sayHello:
        print "Hello! My name is $name."
```

All class variants are just upper-case names placed within the class definition body. They could provide field definitions, but do not have to. Otherwise they are just data types that cannot carry any additional information.

As you can see, if a class consist of single variant only, Luna allows shorter definition syntax. Every field declared in the class level is just copied to the beginning of every variant definition. The methods are defined for all objects of a particular category, that is all instances of the class variants. The constructors of every variant are exported to the class declaration scope, so you can use them without any qualification, as we have seen previously.

So what does it mean that class contains objects sharing similar properties? To answer that question, lets look at the `Bool` definition from Luna's standard library:

```ruby
class Bool:
    True
    False
```

`True` and `False` are variants of the `Bool` class, completely distinct object definitions. But they share a common property - they are the only possible values of any boolean expression. The objects does not have any fields, so their definition is quite simple.

Much more interesting example would be to create custom rendering engine of some primitive shapes. Our renderer would support only two shapes for now, but we can easily extend it later:

```ruby
class Point:
    x y :: Float

class BasicShape:
    Circle:
        pos    :: Point = Point 0 0
        radius :: Float

    Rectangle:
        pos    :: Point = Point 0 0
        width  :: Float
        height :: Float

```

The above definition can be written much shorter in Luna:

```ruby
class BasicShape:
    pos :: Point = Point 0 0

    Circle:    radius       :: Float
    Rectangle: width height :: Float
```

We have defined the `BasicShape` class, that contains all the shapes we are able to render in our toy engine for now. The shapes are defined by `Rectangle` and `Circle` variants. It is worth noting, that both variants share some position information, but different geometry properties. The position information is provided with default value. All variants should be defined before any other class definition.

### Categories
In contrast to classical Object Oriented paradigm, classes in Luna do not explicitly shape the relations between objects. You can think about it like about a generalization of OOP model, where there is infinite amount of relations between objects and we choose the ones, that are useful at the moment. The relation between `Rectangle` and `Circle` is not defined anywhere in the code and there is infinite amount of possible relations to choose from, including the fact that both share properties of geometric figures or that they could be faster rasterized on GPU by our brand new renderer.

Luna does not support inheritance, but it allows overlapping classes instead. It means, that objects can belong to many classes at the same time. Classes are just categories of objects and we have met some of them already, like the integer numbers. Each number is a distinct object with a distinct type, like `1::1`, `2::2`, `3::3` etc. They are defined in the standard library. Every time you write `7`, you are using the constructor named `7` of the data type `7` being a variant of the `Int` class.

Lets go back to our rendering engine. We want to provide some animation support in our library, so our objects could travel a predefined path. The path can be either a rectangle, circle or a segment between two points. We could try to define `Path` the following way:

```ruby
class Path:
    Rectangle: width height x=0 y=0 :: Float
    Circle:    radius       x=0 y=0 :: Float
    Segment:   sx sy ex ey          :: Float
```

Luna will reject the above code, because both data types – `Rectangle` and `Circle` were defined earlier in this module. Class variant definitions declare always new data types, which names cannot be the same across one module. But there is a simple way we can tell Luna to use the earlier defined data types! If we enumerate the concrete objects in an `include` block, they will be included in the category of objects described by that particular class.

```ruby
class Path:
    include: Rectangle
             Circle
    Segment: start end :: Point
```

Now both `Rectangle` as well as `Circle` belong to `Shape` and `Path` classes at the same time. An important fact to note is that even if the data types belong to various categories, their type is `BasicShape`. So if you create a new `Rectangle` object, it will behave like every element of the `BasicShape` category unless stated otherwise. You will learn about transferring objects between categories later.

###Polymorphism
Luna incorporates an easy to use, yet very powerful polymorphism mechanisms. All lower-case identifiers in a type signature denote type variables. Type variables are just like normal variables, but they live in the type-level space and can be used to express type relations.

Class parameters are white space separated type variable identifiers placed after the class name. The following code creates a new class `Vector` containing three fields `x`, `y` and `z` of the same parametric type `a`.

```ruby
class Vector a:
    x y z :: a
```

Now we can use this definition to create various `Vector` objects:

```erlang
λ: v = Vector 1 2 3
λ: type of v
   v :: Vector.Vector 1 2 3 :: Vector Int
```

```erlang
λ: v = Vector 'a' 'ab' 'abc'
λ: type of v
   v :: Vector.Vector 'a' 'ab' 'abc' :: Vector String
```

####The name scopes
There are two important things to note here – the qualified reference to `Vector.Vector` and its parametrization. As we have learned so far, the `Vector` class defines the `Vector` data type, which in turn is associated with the `Vector` constructor. It is possible because the names live in distinct name-spaces and can be accessed in an easy and clear way.

There are some rules regarding how you can access both the class object as well as the constructor function across your code:

* If you define a class containing a variant of the same name, the constructor name will override the class name in the expressions scope. A special case for this situation is when class does not provide explicit variant definition.
```ruby
λ: class Vector a:
       x y z :: a
λ: v = Vector 1 2 3
λ: c = v.cls
```
The `Vector` identifier was used in the example as a constructor of the `Vector` class. You can always access the class definition object using the `cls` property.

* If the class name is different from every variant name, both names are accessible in the expresions scope.
```ruby
λ: class Bool:
       True
       False
λ: a = True
λ: c = Bool
```
The expression `c = Bool` assings the class object `Bool` to the variable `c` and it is the same as `a.cls`.

* The type operator `::` allows accessing the type scope, where the constructor names are not promoted if they overlap with class names.
```ruby
λ: v = Vector 1 2 3
λ: v :: Vector
λ: v :: Vector.Vector 1 2 3
```
In the first line we create new object using the `Vector` constructor. In the second line the identifier `Vector` was used in the type level and it references the `Vector` class. The third line shows how we can access the variant name from within the type scope.

TODO napisac o dualizmie typow 2::2 oraz 2::Int, przy czym Int::Int
napisac o explicite data type decls
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

####Explicit category deinitions
What is even more interesting, we can declare the `Path` using different, more explicit notion. The `class` function is designed in such way, toeliminate the need for any boilerplate. Under the hood it is using other, more explicit functions. In fact the above definition is automatically converted during the compilation time to the following one:

```ruby
class Segment: start end :: Point
alias Path = Rectangle | Circle | Segment
```

```ruby
data Segment: start end :: Point
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



TODO TODO TODO

TODO: dokonczyc, opisac w jakich sytuacjach mozemy podawac typy ogolniesjsze a potem opisac datatpyy konstruowane przez "data"






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
