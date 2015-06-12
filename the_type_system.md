# The type system

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
