# Baby's first functions

In the previous section we got a basic feel for calling a function, now it's time to learn how to create one!

The detailed function evaluation model is described in following chapters. For now, just assume that in most cases you do not need any parentheses in Luna expressions, but sometimes it is just more convenient to use ones.

Let's run a new interactive Luna session by invoking `luna shell` or just `luna` without any parameters. Let me take a closer look at the following session:

```Ruby
$ luna
Welcome to Luna Shell [Luna 1.0 Beta]
Use 'help' to get more info

>> print 5.inc 
6
>> print 5.dec
4
```

As you can see, the functions `inc` and `dec` are members of the `Int` class and we can access them using the dot operator. We can provide arguments to functions simply separating them by spaces, like here:

```Ruby
>> print 5.+ 6
11
```

One thing to note here is the usage of the `print` function. It is declared with a very low precedence level, thus you do not have to use any parenthesis in the expression. This way the code `print 5.+ 6` is the same as `print (5.+ 6)`.

Let's look at the code one more time ... you are right! The name of the function in this example is `+`! As was explained in the [Naming conventions](#naming_conventions) section, the `+` is just an operator name and it is used in this example as a qualified-named function, so its usage is not treated as infix call. We can of course write the following code instead:

```Ruby
>> print 5 + 6
11
```

but this time we call a function `+` available in the current scope instead of the method `+` of the object `5`. In fact, the `+` function is provided by the standard library, having the following definition:

```Ruby
def a + b:
    a.+ b
```

All other arithmetic operators like `+`, `-`, `*`, `/`, etc. are defined this way in Luna. It allows new data types to use operators simply by declaring appropriate methods.


###Function evaluation

You should keep in mind that there is no such keyword in Luna like `return`, which allows immediately returning a value from function. The result of a function is always the value of the last statement. There is though a design proposal introducing such functionality, so it might appear in the future Luna releases. For now the `return` name is reserved for further compatibility.

####Function composition

Lets take a look at a little more complicated example:

```
>> [1..].each random . take 3
[7917908265643496962,-2493721835987381530,5541392136091291592]
```

There are two things to note in the code. The first one is how the `each` and `take` functions were combined using the dot accessor. Luna is a white-space-aware language and allows writing clean and elegant code denoting some of the operators association precedence by using white-spaces.

The above code could also be written as:

```Ruby
>> ([1..].each random).take 3
[7917908265643496962,-2493721835987381530,5541392136091291592]
```

but not as:

```Ruby
>> [1..].each random.take 3
**Error** Function `random` does not provide member `take`.
**Error** Function `List.each` should take only one argument, but is provided with two.
          Couldn't match expected type `Int -> a` with `b, Int -> a`.
```

As you can see, prefixing the dot accessor with whitespace characters lowers its precedence level. Any name followed immediately by dot has a very high one instead. The above code was parsed by Luna the following way:

```Ruby
>> [1..].each (random.take) 3
```

> **Note:**
> There are few design proposals introducing different syntax for function call pipes, but they are not introduced yet.

####Lazy evaluation

Let's go back to our original example! 
```
>> [1..].each random . take 3
[7917908265643496962,-2493721835987381530,5541392136091291592]
```

Because of the fact that Luna is a lazy language, we can create and process infinite structures, being sure that only the needed data will be computed. The above function creates infinite list of ascending numbers, using each number as a random seed and then discarding all but first 3 results. The function `List.each` takes a function as an argument and maps it to every element of the list.

Let's leave functions for now. We will learn more about them after we dig into Luna type system later in this document.