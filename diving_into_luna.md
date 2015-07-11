# Diving into Luna



#Values and types
Every expression in Luna evaluates to a value and has a static type. Values and types are mixed in Luna.


#Naming rules
Luna incorporates few simple naming conventions. There are two identifier types:

* **Lowercase variable identifiers** <br/>
Lowercase identifiers can be used as variable or function names. They constitute any nonempty sequence of letters, starting with a lowercase one. Both characters `_` and `?` are considered lowercase letters.

* **Uppercase type identifiers** <br/>
Uppercase identifiers can be used as both concrete type names as well as type constructor names. Uppercase identifiers constitute any nonempty sequence of letters, starting with an uppercase one.

##Operators
Luna allows some names to be declared as operators. Operator is just like a regular function, but it is implicitly parsed as infix one if not used with qualified name. To declare an operator you can declare a function as an operator or construct the name using any nonenmpty sequence of the following character set: `$ % & * + / < = > \ \ ^ | - ~`.


##Multi-segment names
Luna supports multi-segment names. It means, that you can use multiple identifiers as your variable or type name as long as they obey the naming convention rules. A good example is the multi-segment function `if then else`, which can be used the following way: `if a < b then 1 else 2`.

You can access each multi-segment name just by surrounding it with backticks, like in the following example: ``` `if then else` (a<b) 1 2 ```, which is equivalent to the previous one.

