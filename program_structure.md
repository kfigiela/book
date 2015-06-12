# Program structure

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