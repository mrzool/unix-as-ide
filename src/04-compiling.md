# Compiling

There are a lot of tools available for compiling and interpreting code on the Unix platform, and they tend to be used in different ways. However, conceptually many of the steps are the same. Here I’ll discuss compiling C code with `gcc` from the GNU Compiler Collection, and briefly the use of `perl` as an example of an interpreter.

## GCC

[GCC](http://gcc.gnu.org/) is a very mature GPL-licensed collection of compilers, perhaps best-known for working with C and C++ programs. Its free software license and near ubiquity on free Unix-like systems like Linux and BSD has made it enduringly popular for these purposes, though more modern alternatives are available in compilers using the [LLVM](http://llvm.org/) infrastructure, such as [Clang](http://clang.llvm.org/).

The frontend binaries for GNU Compiler Collection are best thought of less as a set of complete compilers in their own right, and more as *drivers* for a set of discrete programming tools, performing parsing, compiling, and linking, among other steps. This means that while you can use GCC with a relatively simple command line to compile straight from C sources to a working binary, you can also inspect in more detail the steps it takes along the way and tweak it accordingly.

I won’t be discussing the use of `make` files here, though you’ll almost certainly be wanting them for any C project of more than one file; that will be discussed in the next article on build automation tools.

## Compiling and assembling object code

You can compile object code from a C source file like so:

    $ gcc -c example.c -o example.o

Assuming it’s a valid C program, this will generate an unlinked binary object file called `example.o` in the current directory, or tell you the reasons it can’t. You can inspect its assembler contents with the `objdump` tool:

    $ objdump -D example.o

Alternatively, you can get `gcc` to output the appropriate assembly code for the object directly with the `-S` parameter:

    $ gcc -c -S example.c -o example.s

This kind of assembly output can be particularly instructive, or at least interesting, when printed inline with the source code itself, which you can do with:

    $ gcc -c -g -Wa,-a,-ad example.c > example.lst

## Preprocessor

The C preprocessor `cpp` is generally used to include header files and define macros, among other things. It’s a normal part of `gcc` compilation, but you can view the C code it generates by invoking `cpp` directly:

    $ cpp example.c

This will print out the complete code as it would be compiled, with includes and relevant macros applied.

## Linking objects

One or more objects can be linked into appropriate binaries like so:

    $ gcc example.o -o example

In this example, GCC is not doing much more than abstracting a call to `ld`, the GNU linker. The command produces an executable binary called `example`.

## Compiling, assembling, and linking

All of the above can be done in one step with:

    $ gcc example.c -o example

This is a little simpler, but compiling objects independently turns out to have some practical performance benefits in not recompiling code unnecessarily, which I’ll discuss in the next article.

## Including and linking

C files and headers can be explicitly included in a compilation call with the `-I` parameter:

    $ gcc -I/usr/include/somelib.h example.c -o example

Similarly, if the code needs to be dynamically linked against a compiled system library available in common locations like `/lib` or `/usr/lib`, such as `ncurses`, that can be included with the `-l` parameter:

    $ gcc -lncurses example.c -o example

If you have a lot of necessary inclusions and links in your compilation process, it makes sense to put this into environment variables:

    $ export CFLAGS=-I/usr/include/somelib.h
    $ export CLIBS=-lncurses
    $ gcc $CFLAGS $CLIBS example.c -o example

This very common step is another thing that a `Makefile` is designed to abstract away for you.

## Compilation plan

To inspect in more detail what `gcc` is doing with any call, you can add the `-v` switch to prompt it to print its compilation plan on the standard error stream:

    $ gcc -v -c example.c -o example.o

If you don’t want it to actually generate object files or linked binaries, it’s sometimes tidier to use `-###` instead:

    $ gcc -### -c example.c -o example.o

This is mostly instructive to see what steps the `gcc` binary is abstracting away for you, but in specific cases it can be useful to identify steps the compiler is taking that you may not necessarily want it to.

## More verbose error checking

You can add the `-Wall` and/or `-pedantic` options to the `gcc` call to prompt it to warn you about things that may not necessarily be errors, but could be:

    $ gcc -Wall -pedantic -c example.c -o example.o

This is good for including in your `Makefile` or in your [`makeprg`](http://vim.wikia.com/wiki/Errorformat_and_makeprg) definition in Vim, as it works well with the quickfix window discussed in the previous article and will enable you to write more readable, compatible, and less error-prone code as it warns you more extensively about errors.

## Profiling compilation time

You can pass the flag `-time` to `gcc` to generate output showing how long each step is taking:

    $ gcc -time -c example.c -o example.o

## Optimisation

You can pass generic optimisation options to `gcc` to make it attempt to build more efficient object files and linked binaries, at the expense of compilation time. I find `-O2` is usually a happy medium for code going into production:

-   `gcc -O1`
-   `gcc -O2`
-   `gcc -O3`

Like any other Bash command, all of this can be [called from within Vim](http://blog.sanctum.geek.nz/unix-as-ide-editing/) by:

    :!gcc % -o example

## Interpreters

The approach to interpreted code on Unix-like systems is very different. In these examples I’ll use Perl, but most of these principles will be applicable to interpreted Python or Ruby code, for example.

## Inline

You can run a string of Perl code directly into the interpreter in any one of the following ways, in this case printing the single line “Hello, world.” to the screen, with a linebreak following. The first one is perhaps the tidiest and most standard way to work with Perl; the second uses a [heredoc](http://tldp.org/LDP/abs/html/here-docs.html) string, and the third a classic Unix shell pipe.

    $ perl -e 'print "Hello world.\n";'
    $ perl <<<'print "Hello world.\n";'
    $ echo 'print "Hello world.\n";' | perl

Of course, it’s more typical to keep the code in a file, which can be run directly:

    $ perl hello.pl

In either case, you can check the syntax of the code without actually running it with the `-c` switch:

    $ perl -c hello.pl

But to use the script as a *logical binary*, so you can invoke it directly without knowing or caring what the script is, you can add a special first line to the file called the “shebang” that does some magic to specify the interpreter through which the file should be run.

    #!/usr/bin/env perl
    print "Hello, world.\n";

The script then needs to be made executable with a `chmod` call. It’s also good practice to rename it to remove the extension, since it is now taking the shape of a logic binary:

    $ mv hello{.pl,}
    $ chmod +x hello

And can thereafter be invoked directly, as if it were a compiled binary:

    $ ./hello

This works so well that many of the common utilities on modern Linux systems, such as the `adduser` frontend to `useradd`, are actually Perl or even Python scripts.

In the next post, I’ll describe the use of `make` for defining and automating building projects in a manner comparable to IDEs, with a nod to newer takes on the same idea with Ruby’s `rake`.

