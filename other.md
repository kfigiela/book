 --------
There is however one important thing to keep in mind. Luna will reject any explicit type signature, that is not as specific as possible:
```ruby
>> lst = [] :: x
**ERROR**
Luna interactive 1:12: Provided type `x` is not as specific as possible: `[]`.
```

--- opisac maybe



# Other

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
