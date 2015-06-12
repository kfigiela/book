# Diving into Luna

## Your first program
As the sacred tradition obliges, we start with the "Hello world" example.

```ruby
def main:
    print "Hello world!"
```

###Ready, set, go!
Save the source code from the snippet above as the `Main.luna` file. During the compilation, Luna looks for `main` function and treats it as the program entry point. Luna source code files should be UTF-8 encoded.

Each Luna source file needs to have a name starting with an upper-case letter, followed by the `.luna` extension. Why the upper-case? This is because each source file defines a Luna module. Modules are data types, and Luna requires all type names to start with an upper-case character.

The language is bundled with a command line tool `luna`, which is an interface for the family of tools from Luna's ecosystem, including compiler, interactive interpreter, repository manager, code assistance or documentation manager.

To build the program, type `luna Main.luna` in your shell. The command executes the compiler, which firstly compiles Luna source code and links the resulting program with the Luna's standard library. The result is saved in an executable named `Main` (or `Main.exe` on Windows).

To run the program simply type `./Main` (or just `Main` on Windows). As a result you should get output as shown below:
```Ruby
"Hello world!"
```

Oh, wait a minute! Why the output is enclosed in quotes? For now, just remember that the `print` function works this way — you will learn about why later. If you want to output the text without the quotes, use `puts` instead.

The `luna Main.luna` command is just a shortcut for more verbose one -- `luna build Main.luna`. The compiler provides a wide set of commands, including `run`, `build`, `install` or `check`. Each command could provide relevant sub-commands and options. To examine them you can use the `--help` option:

```bash
$ luna build --help
-o, --output OUTPUT           Save the output into OUTPUT
-O, --optimization   [0-2]    Optimization level (default = 2)
-v, --verbose        [0-5]    Verbosity level (default = 3)
-h, --help                    Display detailed help message
[...]
```

You can also use the `run` command to evaluate the program without producing an executable: `luna run Main.luna`.


##Baby's first functions
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



#Program structure

##Entities
Luna programs are built out of five entities:

Modules
: Module is a collection of definitions in an environment created by a set of imports. Modules in Luna are reflecting the source files hierarchy. To learn more about modules, read the [Modules](#modules) chapter.

Definitions
: Definition describes one of function, data type, interface or type alias. Definitions can be placed in modules or nested as sub-definitions.

Expressions
: Expression allows describing algorithms by providing information about data dependencies and necessary transformations. TODO

Universes
: Luna is a dependent typed language, allowing mixing the values and types together. Luna provides notion of universes - higher order type families. Type information in Luna can be omitted in vast majority of cases, but it can be used to express 
Types of types are called kinds, types of kinds Type describes what kind of value an expression evaluates to. Type information in Luna can be omitted in vast majority of cases, but they can be used in expression to provide explicit type annotation and as a part of polymorphic class and interface declaration.
universe-level programming.

Patterns
: Pattern provides mechanism for datatypes decomposition. They can be used in function header declarations and both pattern binding as well as case expressions.


##Code layout
Luna code is build out of nested code blocks. There are two types of blocks -- indentation sensitive colon-blocks and indentation insensitive braces-blocks. Former are defined after the colon `:`, while later between braces `{` `}`. Below you can find an example function definition using both methods.

```ruby
def main:
    print "I'm using the colon-block layout!"
```

```ruby
def main{
    print "I'm using the braces-block layout!";
}
```

Code blocks, as everything in Luna
> **Note:**

> Both colon- and braces- code layouts have advantages and disadvantages and are usable in different scenarios. We strongly encourage you not to mix the styles within one source code. The code can easily become unreadable. In fact Luna dissallows mixing different code layouts within single source file, unless the `MixedLayout` extension is enabled.  This rule does not affect the top-level program layout. To learn more about compiler extensions read the [Language extensions](#language_extensions) section.

TODO: napisac ze code blocks sa obiektami w sekcji o expewssion

### Colon-blocks
TODO: proposal check

Colon blocks were designed to maximally increase code readability and layout flexibility. There are three rules describing how the colon-blocks work:

1. Each block is associated with an indentation level defined as its first expression's indentation level;
2. The first expression in a block should have bigger indentation level then the parent's block one;
3. Expressions can span over multiple lines, but all the spanned lines have to be indented more than the expression itself.

Here are some examples of different usages of colon-blocks:

```ruby
# an inline block
class Vector a: x,y,z :: a

def checkVector v:
    # multiline expression:
    if v.x > v.y then True
                 else False

# standard function declaration with indented colon-block
def main:
    v = Vector 1 2 3
    print $ checkVector v
```

But the following examples are **invalid**:

```ruby
def checkVector v:
    # ERROR! Multiline expression is not properly indented 
    if v.x > v.y then True
    else False

# ERROR! Indentation level does not match the first expression's one
def main: v = Vector 1 2 3
    print $ checkVector v
```


###Braces-blocks
The braces-blocks can be used to minimize the code and eliminate the indentation sensitive syntax. Braces-blocks require all the statements but the last one to be terminated using the semicolon terminator `;`. It is also possible to use the braces block as top-level program layout by enclosing the whole source code in braces:
```ruby
{def main{print "everything is braced!"}; def foo a{a+1}}
```

##Scoping
Scopes define when a particular name is accessible in a program. Each code block creates a distinct name scope, inheriting the names from the parent one. All the names of entities defined in a scope cannot be accessed from any parent scope. It is valid to have different entities with the same name available within a single scope, but it is invalid to use them. Such situation can happen when importing functions from different modules - sometimes the same name can be imported. In such case a compilation error about the name ambiguity will be raised when trying to use such name. The Luna compiler warns about ambigous names within single scope even if they are not used anywhere in the code.

#Naming

##Values and types
Every expression in Luna evaluates to a value and has a static type. Values and types are mixed in Luna.

##Naming rules {#naming_rules}
Luna incorporates few simple naming conventions. There are two identifier types:

Lowercase variable identifiers
: Lowercase identifiers can be used as variable or function names. Lowercase identifiers constitute any nonempty sequence of letters, starting with a lowercase one. Both characters `_` and `?` are considered lowercase letters.

Uppercase type identifiers
: Uppercase identifiers can be used as both concrete type names as well as type constructor names. Uppercase identifiers constitute any nonempty sequence of letters, starting with an uppercase one.

##Operators
Luna allows some names to be declared as operators. Operator is just like a regular function, but it is implicitly parsed as infix one if not used with qualified name. To declare an operator you can declare a function as an operator or construct the name using any nonenmpty sequence of the following character set: `!#$%&*+/<=>\\^|-~`.


##Multi-segment names
Luna supports multi-segment names. It means, that you can use multiple identifiers as your variable or type name as long as they obey the naming convention rules. A good example is the multi-segment function `if then else`, which can be used the following way: `if a < b then 1 else 2`.

You can access each multi-segment name just by surrounding it with backticks, like in the following example: ``` `if then else` (a<b) 1 2 ```, which is equivalent to the previous one.

