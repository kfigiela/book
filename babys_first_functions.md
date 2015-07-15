# Baby's first functions

In the previous section we got a basic feel for calling a function, now it's time to learn how to create one. The detailed function evaluation model is described in following chapters. For now, just assume that in most cases you do not need any parentheses in Luna expressions, but sometimes it is just more convenient to use ones.


####Luna interactive shell
Let's run a new interactive Luna session by invoking `luna shell` or just `luna` without any parameters:

```ruby
$ luna
Luna Shell 1.0 Alpha
Type "help" to get more info

  1 λ:
```

The Luna shell runs the Luna interpreter allowing interactive code evaluation and inspection. At start the shell displays a greeting message and an input prompt. The prompt displays the current line number and lambda sign `λ` indicating the start of user input. The line numbers are often omitted across this book unless they are needed as further reference.

####Objects and methods
Let me take a closer look at the following session:

```ruby
$ luna
Luna Shell 1.0 Alpha
Type "help" to get more info

  1 λ: print 5.inc
       6
  2 λ: print 5.dec
       4
```

Luna uses the dot-notation to access methods associated with values. As you can see, the functions `inc` and `dec` are members of the `5` object. We can provide arguments to functions simply separating them by spaces, like in the following code snippet:

```ruby
λ: print 5.+ 6
   11
```

One thing to note here is the usage of the `print` function. It is declared with a very low precedence level, thus you do not have to use any parenthesis in the expression. This way the code `print 5.+ 6` is the same as `print (5.+ 6)`.

Let's look at the code one more time… you are right! The name of the function in this example is `+`! As was explained in the [Naming conventions](#naming_conventions) section, the `+` is just an operator name and it is used with explicit qualification in this example, so it is not treated as an infix call. We can of course write the following expression instead:

```ruby
λ: print 5 + 6
   11
```

but this time we call a function `+` available in the current scope instead of the method `+` of the object `5`. In fact, the `+` function is provided by the standard library and has the following definition:

```ruby
def a + b:
    a.+ b
```

All other arithmetic operators like `+`, `-`, `*`, `/`, etc. are defined in the same fashion in Luna. New data types may use operators by simply declaring appropriate methods.


###Function evaluation

The result of a function is always the value of the last statement. You should keep in mind that there is no such keyword in Luna like `return`, which allows immediately returning a value from function. There is though a design proposal introducing such functionality, so it might appear in a future Luna release. For now the `return` name is reserved for further compatibility.

####Function composition

Lets take a look at a little more complicated example:

```ruby
1 λ: [1..].each random . take 3
     [7917908265643496962,-2493721835987381530,5541392136091291592]
```

There are two things to note in the code. The first one is how the `each` and `take` functions were combined using the dot accessor. Luna is a white-space-aware language and allows writing clean and elegant code denoting some of the operators association precedence by using white-spaces.

The above code could also be written as:

```ruby
2 λ: ([1..].each random).take 3
     [7917908265643496962,-2493721835987381530,5541392136091291592]
```

but not as:

```ruby
3 λ: [1..].each random.take 3
  *: Function `random` does not provide member `take`.
  *: Function `List.each` should take only one argument, but is provided with two.
```

As you can see, prefixing the dot accessor with whitespace characters lowers its precedence level. Any name followed immediately by dot has a very high one instead. The above code was parsed by Luna the following way:

```ruby
[1..].each (random.take) 3
```

####Lazy evaluation

Let's go back to our original example.

```ruby
λ: [1..].each random . take 3
   [7917908265643496962,-2493721835987381530,5541392136091291592]
```

Because Luna is a lazy language, we can create and process infinite structures, being sure that only the needed data will be computed. The above function creates infinite list of ascending numbers, feeding them as random seeds into `random` function and then discarding all but first 3 results. The function `List.each` takes a function as an argument and maps it to every element of the list.

Let's leave functions for now. We will learn more about them after we dig into Luna type system later in this document.
