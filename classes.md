# Classes

Classes are used to classify objects based on their properties and behaviors. You can learn more about the concept in the [Category Oriented Programming](category_oriented_programming.md) chapter. 

Class definition is very similar to an object type definition. Classes describe properties and behaviors an object must provide in order to be considered the class member. Every object that meets the requirements belongs to the class even if there is no explicit statement about it in the code.

A new class can be defined using the `class` function. Lets create our first, brand new, shiny class! 

```ruby
class XY:
    x y :: Real
```

The `XY` class describes all objects containing both `x` and `y` fields of type `Real`. Every object that provides such fields belongs to the class, including all the instances of the `Point` object type from previous chapter.

### Object type classes
Object types and classes are similar in many ways. Each object type defines in fact a class of variants. Object type is just a different way of declaring the class – instead of describing requirements an object has to meet, it explicitly enumerates all of its members. In contrast to classes however, object types are able to define new data structures.

Let's take another look at the `Shape` object type definition:

```ruby
object Shape:
    Circle:
        pos    :: Point = new Point 0 0
        radius :: Float

    Rectangle:
        pos    :: Point = new Point 0 0
        width  :: Float
        height :: Float
```

The `Shape` object type is just a class containing the `Circle` and `Rectangle` variant instances.

### Polymorphism
Luna incorporates an easy to use, yet very powerful polymorphism mechanisms. All lower-case identifiers in a class or object type signature denote type variables. Type variables are just like regular variables, but they live in the type-level and can be used to express type relations.

The parameters are white space separated identifiers placed after the class or object type name. The following code creates a new `Point` object type containing two fields `x` and `y` of the same parametric type `a`.

```python
class Point a:
    x y :: a
```

Now we can use this definition to create various `Point` objects:

```ruby
λ: p = Point 0.3 0.4
λ: :t p
   p :: Point.Default 0.3 0.4 :: Point Real
```

```ruby
λ: p = Point 1 2
λ: :t p
   p :: Point.Default 1 2 :: Point Int
```

It is worth noting that the the types of `1` and `2` are the variant types `1` and `2` respectively. They are subtypes of the `Int` class, so Luna is allowed to display the type of `p` as both a very concrete `Point.Default 1 2` as well as more general `Point Int`. In fact Luna is wise enough to give you the concise, uncluttered output, sometimes omitting or simplifying parts of the signature, but never changing its meaning. There is for example an intermediate type signature that was omitted in the output:

```ruby
p :: Point (1 | 2)
```

The signature means, that `p` is a value of type `Point`, which first type argument is either `1` or `2`. If we would like to express the detailed type dependency chain, we could write the following statement:

```ruby
p :: Point.Default 1 2 
  :: Point (1 | 2) 
  :: Point Int 
  :: Point Real 
  :: Point Complex 
  :: Point Number 
  :: Point * 
  :: *
```




### Class composition
It is possible to construct such class manually in Luna using the class composition operator `|`:

```ruby
MyShape = Circle Point Float | Rectangle Point Float Float
```

The class composition forms a category of all objects and behaviors of its components. Such composition forms a class as well. You can compose several classes using the class composition operator `|`. Lets define a funny function!

```ruby
def foo x:
    if x < 0 then 'oh no!'
             else x
```

The above code defines a `foo` function, which takes an argument `x`, compares it to `0` and returns either string `'oh no!'` or the value of `x`. The function results thus in a value of a type `Int | 'oh no!'`, which is a subtype of a more general one `Int | String`. Such value inherits all the common behaviors of elements from both categories.

The composed class allows for using any sensible method from all the component classes. Lets investigate this behavior using the Luna shell! 

```ruby
λ: lst = [2, 'test']
λ: type of lst
   lst :: [2, 'test'] :: List (Int | String)
λ: lst.each el: el + 1
   [3, 'test1']
λ: lst.each el: el * 2
   [4, 'testtest']
λ: lst.each el: el/2
*: Unsupported operand / for type String
```

As we can see, we can use both `+` as well as `*` operator on elements of the list, but we cannot use the `/` one, because it does not have any sense for the `String` type. If we use a method, which will result in the same type, Luna will unify the result type accordingly:

```ruby
λ: ulst = lst.each el: el.to String
λ: print ulst
   ['2', 'test']
λ: type of ulst
   ulst :: ['2', 'test'] :: List String
```

The method `to` takes a type as argument and converts the object to that type if possible.

### Types as values

Luna is a dependent type language. It means that values can be used in the type level and vice-versa – types can be used as values. The syntax for both value and type level operations is always the same. A good example is the expression `2*8 :: 2^4`, which is an integer number `16` of type `16`.

Every object in Luna belongs to an infinite number of classes, especially to the unit class – a class defined by the object itself. An example unit classes are `1` or `'test'`, which can be used to describe `1` and `'test'` values respectively. Each class is a subset of other class, in particular itself. It is valid in Luna to tell that `Int` is a subset of `Int`, like `Int :: Int`. In fact, if a function expects an argument of type `Int` you can provide the `Int` object itself:

```ruby
λ: print 'test' + 1
   'test1'
λ: print 1.show
   '1'
λ: print String + Int
   String
λ: print Int.show
   String 
```
This is a very powerful mechanism. It can be used for both type level programming as well as to bulk-process all values from a specific category. Lets assume we've got a heterogeneous dictionary and we want to obtain all the values, whose key is an integer:

```ruby
λ: dict = {'foo':'bar', 1:2, 3:4}
λ: dict.lookup Int . values
   [3,4]
```
----

Classes are just categories of objects and we have met some of them already, like the integer numbers. Each number is a distinct object with a distinct type, like `1::1`, `2::2`, `3::3` etc. They are defined in the standard library. Every time you write `7`, you are using the constructor named `7` of the data type `7` being a variant of the `Int` class.

OPISAC Int + Int === Int






TODO napisac o dualizmie typow 2::2 oraz 2::Int, przy czym Int::Int
napisac o explicite data type decls

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
