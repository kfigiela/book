# Your first program

As the sacred tradition obliges, we start with the "Hello world" example.

```ruby
def main:
    print "Hello world!"
```

###Ready, set, go!
Save the source code from the snippet above as the `Main.luna` file. During the compilation, Luna looks for `main` function and treats it as the program entry point. Luna source code files should be UTF-8 encoded.

Each Luna source file needs to have a name starting with an upper-case letter, followed by the `.luna` extension. Why the upper-case? This is because each source file defines a Luna module. Modules are data types, and Luna requires all type names to start with an upper-case character.

The language is bundled with a command line tool `luna`, which is an interface for the family of utilities from Luna's ecosystem including compiler, interactive interpreter, repository manager, code assistance or documentation manager.

####Building the sources
To build the program, type `luna Main` in your shell. The file extension can be omited when building the program. The command executes the Luna compiler, which reads the sources, compiles the code into target platfrom and links the resulting program with the Luna's standard library.  To The result is saved in an executable named `Main` (or `Main.exe` on Windows).

To run the program simply type `./Main` (or just `Main` on Windows). As a result you should get output as shown below:
```Ruby
"Hello world!"
```

Oh, wait a minute! Why the output is enclosed in quotes? For now, just remember that the `print` function works this way — you will learn about why later. If you want to output the text without the quotes, use `puts` instead.

Luna chooses the `native` target platform unless stated otherwise. You can choose different platform using the `--target` option. In order to run the example in a web browser, you can compile it to JavaScript using `luna --target js Main` instead.

####Compilation options

The `luna Main` command is just a shortcut for more verbose one – `luna build Main.luna`. The compiler provides a wide set of commands, including `run`, `build`, `install` or `check`. Each command could provide relevant sub-commands and options. To examine them you can use the `--help` option:

```bash
$ luna build --help
-o, --output OUTPUT   Save the output into OUTPUT.
-O, --optimization    Optimization level (default = 2).
-v, --verbose         Verbosity level (default = 3).
-t, --target          Target platform.
-h, --help            Display detailed command help message.
[...]
```

You can also use the `run` command to evaluate the program without producing an executable, like `luna run Main`.
