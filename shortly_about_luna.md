# Shortly about Luna

Luna is intended to be a production language, not a research one, incorporating at the same time many recent innovations in programming language design. It is the world's first high performance, general purpose, hybrid visual-textual programming language. 

Luna provides a source and target independent language specification bundled with set of open-source, reusable compiler and tool-chain technologies.

###Purely functional, multi-paradigm language
Luna was designed from the ground up to be a high-performance, robust, enterprise solution, allowing easy creation of secure and scalable applications. Luna combines many aspects of Object-Oriented, Data-Flow and purely Functional programming into a new Category Programming paradigm. It provides higher-order functions, non-strict semantics, user-defined algebraic data types, structural subtyping, advanced type inferencer, a rich set of primitive datatypes and support for dependent type programming. Moreover Luna introduces few new concepts, including variant type system, data-flow error handling or automatic monad management.

###Lazy and strict evaluation

###Immutability
In contrast to vast majority of Object Oriented languages, objects are immutable in Luna. It means that you cannot modify the state of an object after it is created. Immutable objects are often useful because they are inherently thread-safe, simpler to understand and reason about, and offer higher security than mutable ones.

Such design decision could seem inefficient at first glance, but its subject for compilation-time optimization. In fact immutable data structures can perform as fast as mutable ones and the Luna compiler is able to optimize them this way.

###High performance
Luna was designed as a high-level, high-performance, easy to use programming language. Simple and concise syntax allows writing clean and elegant programs, while maintaining all the benefits of the static type system, like compile-time error detection and optimization opportunities. The resulted performance of a program depends on the optimizations introduced by the compiler, but not every language can be optimized the same way. Luna was designed to provide all the necessary information to allow drastic optimization.

###Multiple source representations
Luna supports multiple syntax representations, currently including text and visual one. Every program in the text form has its equivalent representation as a visual data-flow graph and vice-versa. Both forms are interchangeable and can be used simultaneously. There are plans to develop other source representations, including natural language one and sql-like queries.

###Multiple target platforms
We believe in a very simple vision behind Luna - write once, use everywhere. Luna specification is target-platform independent. It is possible to compile the same Luna sources to different output targets, currently including fast machine code and JavaScript. More backends will be supported in the future, like Python, Java or Erlang one.

You can use Luna to both develop applications from scratch as well as create reusable components to existing projects. The optimizations take place before target platform code generation, so Luna often produces faster code, than native language implementation.

###Incredible type system (Robust, etc)

###Well designed compilation framework
