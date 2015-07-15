# Program structure

Luna programs are built out of six entities:

* **Types and type universes** <br/>
Type describes a valid set of values of a particular variable or expression. Type classifies objects by their properties and behaviors. Types in Luna are in fact values in a higher universe than objects, so every type has it's own type as well.

* **Modules** <br/>
Module is a collection of definitions in an environment created by a set of imports. Hierarchy of modules reflects the hierarchy of source files. You will learn more about modules in the [Modules](#modules) chapter.

* **Definitions** <br/>
Definition describes top level entity, such as a function or a data type. Definitions can be placed in modules or nested inside other definitions as their sub-definitions.

* **Expressions** <br/>
Expression describes data dependencies and the necessary transformations in order to compute desired value.

* **Patterns** <br/>
Pattern provides mechanism for data-type decomposition. They can be used in several placess across the code, including function headers and expressions.

* **Objects** <br/>
Object is a value with associated type. Objects are immutable and are build out of other values called fields and associated functionalities called methods.


##Code layout
Luna code is build out of nested code blocks. There are two types of blocks – indentation sensitive colon-blocks and indentation insensitive braces-blocks. They are defined after the colon `:` and between braces `{` `}` respectively. The following example shows a sample function definition  using both code layouts:

```ruby
def main:
    print "I'm using the colon-block layout!"
```

```ruby
def main {
    print "I'm using the braces-block layout!";
}
```

Both colon- and braces- code layouts have advantages and disadvantages and can be useful in different scenarios. We strongly encourage you not to mix the styles within single source file, because the code can easily become unreadable. In fact Luna dissallows mixing different code layouts within single source file, unless the `MixedLayout` pragma is enabled.


### Colon-blocks

Colon blocks were designed to maximally increase code readability and layout flexibility. There are three rules describing how the colon-blocks work:

1. Each block is associated with an indentation level defined as its first expression's indentation level;
2. Each nested code block should have bigger indentation level than the parent's one;
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
# ERROR: Multiline expression is not properly indented
def checkVector v:
    if v.x > v.y then True
    else False

# ERROR: Indentation level does not match the first expression's one
def main: v = Vector 1 2 3
    print $ checkVector v
```


###Braces-blocks
The braces-blocks can be used to minimize the code and eliminate the indentation sensitive syntax. Braces-blocks require all the statements but the last one to be terminated using the semicolon terminator `;`. It is also possible to use the braces block as top-level program layout by enclosing the whole source code in braces:
```ruby
{def main{print "everything is braced!"}; def foo a{a+1}}
```

##Scoping
Scopes define when a particular name is accessible in a program. Each code block creates a distinct name scope, inheriting the names from the parent one. All the names of entities defined in a scope cannot escape its boundaries, in particular they cannot be accessed from the parent scope.

It is valid to have different entities with the same name available within a single scope, but it is invalid to use them. Such situation can happen when importing functions from different modules – sometimes the same name can be imported. In such case a compilation error about the name ambiguity will be raised when trying to use such name. The Luna compiler warns about ambigous names within single scope even if they are not used anywhere in the code.
