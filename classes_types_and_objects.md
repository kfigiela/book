# The type system
The Luna's type system has the following characteristics:

* **Static type system**<br/>
  Luna is a statically typed language, which means that the type of every expression is known at compile time. The type safety leads to safer code and is the foundation for the creation of robust, maintainable systems. It is much better approach than relying on run-time errors, like many languages do nowadays, because a lot of possible errors are caught at compile time. You can be sure that if a program compiles, it does not contain any type based errors, like adding a number and a cake (unless you provide any sensible description how to add 5 to a bun)!

* **Dependent types** <br/>
  Luna is a dependent type language. It means, that types and values live in the same space, can be mixed and that types can depend on the values. For example Luna understands how many elements a list stories or what keys are available in a dictionary. In fact Luna understand such relations for every user defined datatype without any explicit code. Luna was designed to understand as much as possible to provide best in class machinery proving that a particular program will work correctly under given conditions. Dependent types are a wide research subject and we are constantly working on extending the possibilities in Luna as much as possible.

* **Honesty** <br/>
  Luna's type system is an honest one. This informal term means that the type system does not cheats you! In other words, Luna's type system provides mechanisms for describing the detailed system behavior, including the function purity and possible error occurrence. In Luna you can use type system to indicate that a particular function is pure and does not communicate with any resources external to the program, like all the I/O operations. Such functionality enables Luna to allow many uncommon features, like automatic data parallelism on multicore CPUs and GPUs or support for Software Transactional Memory (STM). There is a great video by Erik Meijer about modern functional language features and the type system honesty: https://www.youtube.com/watch?t=678&v=z0N1aZ6SnBk.


###The type inference

Luna's type system provides an inference mechanism. It means that you don't have to explicitly label every piece of code with a type information because the type system can intelligently figure it out for you. In practice you do not have to provide any type information in vast majority of cases, but sometimes it is just handy to do so.

```ruby
λ: :t 'Hello World'
   'Hello World' :: 'Hello World' :: String
```

Luna always infers the most specific type for every expression. The output means that `'Hello World'` is a variable of the unit-class `'Hello World'`, which is a subtype of class `String`. The only value of such type is the string `'Hello World'` itself. The `:t` command prints both the unit-class as well as the superclass if the unit-class is available. It is very usefull especially when inspecting the types.

```ruby
λ: :t 1 == 2
   1 == 2 :: False :: Bool
```

In the same fashion, the type of expression `1 == 2` is `False`, which belongs to the class of boolean values `Bool`. 

It is not always possible to infer types so precisely, in particular when the value depends on some run-time operations, like reading a file from disk. We can easily declare a function, which inferred type would be more general:

```ruby
λ: def cmpToHello s:
       'Hello' == s
```

The function `cmpToHello` takes single argument and compares it with our favorite string. The simplified inferred type of `cmpToHello` is `a -> Bool`. It means that `s` could be of any type and the result is either `True` or `False`, which forms the category `Bool`. Unlike many languages, Luna allows comparison of objects of different types. It's an obvious fact taking in concideration that even `'foo'` and `'bar'` can be values of variables having distinct types and can belong to arbitrary set of classes.

It is important to note, that Luna will try to determine the most precise type when a particular function is used:

```ruby
λ: :t cmpToHello "World"
   cmpToHello "World" :: False :: Bool
```

Even though the type of `cmpToHello` was inferred as `a -> Bool`, the type of `cmpToHello "World"` is `False`. You can think about the function `cmpToHello` like about all possible functions from a particular sentence to a boolean value. There is infinite number of such functions, including `'Hello' -> True` or `'World' -> False`.

In fact we can write the type signature for `cmpToHello` more precisely:

```ruby
cmpToHello :: "Hello" -> True | _ -> False
```
But even if we don't, Luna keeps such relation under the hood. The pipe operator `|` is used to compose classes together. You will learn more about it later in this book.

We can query Luna for all possible values from a particular class using the `values` method.

```ruby
λ: Bool.values
   {True, False}
```

We can also ask Luna if a particular value belongs to a given class using the `in` operator:

```ruby
λ: True in Bool
   True
λ: 2.5 in Int
   False
```

It is also possible to query the compiler about all the the defined classes a particular value belongs to. The `classes` object method serves exactly this purpose.

```ruby
λ: 's'.classes
   {Char, String}

λ: 'str'.classes
   {String}
```


###Explicit type signatures
Explicit type signatures could be useful in few situations, like type level programming, debugging, type inspection, increasing code clarity or in order to limit the range of possible values.

Types in Luna are very expressive and they help to create self-documenting code, thus we strongly encourage to use explicit type signatures when defining top-level functions and class methods. If such signature is missing, Luna warns about it during compilation. You can turn off the warnings by using the compiler options `--type.sig.missing.top none` and `--type.sig.missing.class none`. You can also change the behavior to discard programs if any top-level function is not provided explicitly by changing the option to `--type.sig.missing.top error`, but we consider this behavior too restrictive for everyday use.

Luna will reject any program with ill-typed expression:
```erlang
λ: a = 'word' :: Int
*: The value 'word' doesn't belong to the Int class.
```

####Partial type signatures
Luna allows to provide partial type information, leaving type checker the inference of the missing part. You can use the wildcard operator `_` to omit information about a particular type in an explicit type signature. A good example would be to annotate a variable being a `Vector` of `Ints`:

```erlang
λ: v1 = Vector 1 2 3 :: Vector Int
λ: v2 = Vector 1 2 3 :: Vector _
```

####Type holes
Sometimes it is helpful to query the compiler about the inferred type of a particular expression. You can use the type hole operator `??` exactly for that purpose:

```ruby
λ: a = 'str' + 7
λ: a :: ??
*: Found a type hole with the signature of a :: 'str' :: String
```

In the above example Luna inferred the type of `a` to be `'str'`, which belongs to the `String` category.

Type holes are treated as compilation errors by default. You can change this behavior by using the compiler option `--type.hole warning` to report them as warnings instead.



