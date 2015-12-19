# Building

Because compiling projects can be such a complicated and repetitive process, a good IDE provides a means to abstract, simplify, and even automate software builds. Unix and its descendents accomplish this process with a `Makefile`, a prescribed recipe in a standard format for generating executable files from source and object files, taking account of changes to only rebuild what’s necessary to prevent costly recompilation.

One interesting thing to note about `make` is that while it’s generally used for compiled software build automation and has many shortcuts to that effect, it can actually effectively be used for any situation in which it’s required to generate one set of files from another. One possible use is to generate web-friendly optimised graphics from source files for deployment for a website; another use is for generating static HTML pages from code, rather than generating pages on the fly. It’s on the basis of this more flexible understanding of software “building” that modern takes on the tool like [Ruby’s `rake`](http://rake.rubyforge.org/) have become popular, automating the general tasks for producing and installing code and files of all kinds.

## Anatomy of a `Makefile`

The general pattern of a `Makefile` is a list of variables and a list of *targets*, and the sources and/or objects used to provide them. Targets may not necessarily be linked binaries; they could also constitute actions to perform using the generated files, such as `install` to instate built files into the system, and `clean` to remove built files from the source tree.

It’s this flexibility of targets that enables `make` to automate any sort of task relevant to assembling a production build of software; not just the typical parsing, preprocessing, compiling proper and linking steps performed by the compiler, but also running tests (`make test`), compiling documentation source files into one or more appropriate formats, or automating deployment of code into production systems, for example, uploading to a website via a `git                                                                                                             push` or similar content-tracking method.

An example `Makefile` for a simple software project might look something like the below:

    all: example

    example: main.o example.o library.o
        gcc main.o example.o library.o -o example

    main.o: main.c
        gcc -c main.c -o main.o

    example.o: example.c
        gcc -c example.c -o example.o

    library.o: library.c
        gcc -c library.c -o library.o

    clean:
        rm *.o example

    install: example
        cp example /usr/bin

The above isn’t the most optimal `Makefile` possible for this project, but it provides a means to build and install a linked binary simply by typing `make`. Each *target* definition contains a list of the *dependencies* required for the command that follows; this means that the definitions can appear in any order, and the call to `make` will call the relevant commands in the appropriate order.

Much of the above is needlessly verbose or repetitive; for example, if an object file is built directly from a single C file of the same name, then we don’t need to include the target at all, and `make` will sort things out for us. Similarly, it would make sense to put some of the more repeated calls into variables so that we would not have to change them individually if our choice of compiler or flags changed. A more concise version might look like the following:

    CC = gcc
    OBJECTS = main.o example.o library.o
    BINARY = example

    all: example

    example: $(OBJECTS)
        $(CC) $(OBJECTS) -o $(BINARY)

    clean:
        rm -f $(BINARY) $(OBJECTS)

    install: example
        cp $(BINARY) /usr/bin

## More general uses of `make`

In the interests of automation, however, it’s instructive to think of this a bit more generally than just code compilation and linking. An example could be for a simple web project involving deploying PHP to a live webserver. This is not normally a task people associate with the use of `make`, but the principles are the same; with the source in place and ready to go, we have certain targets to meet for the build.

PHP files don’t require compilation, of course, but web assets often do. An example that will be familiar to web developers is the generation of scaled and optimised raster images from vector source files, for deployment to the web. You keep and version your original source file, and when it comes time to deploy, you generate a web-friendly version of it.

Let’s assume for this particular project that there’s a set of four icons used throughout the site, sized to 64 by 64 pixels. We have the source files to hand in SVG vector format, safely tucked away in version control, and now need to *generate* the smaller bitmaps for the site, ready for deployment. We could therefore define a target `icons`, set the dependencies, and type out the commands to perform. This is where command line tools in Unix really begin to shine in use with `Makefile` syntax:

    icons: create.png read.png update.png delete.png

    create.png: create.svg
        convert create.svg create.raw.png && \
        pngcrush create.raw.png create.png

    read.png: read.svg
        convert read.svg read.raw.png && \
        pngcrush read.raw.png read.png

    update.png: update.svg
        convert update.svg update.raw.png && \
        pngcrush update.raw.png update.png

    delete.png: delete.svg
        convert delete.svg delete.raw.png && \
        pngcrush delete.raw.png delete.png

With the above done, typing `make icons` will go through each of the source icons files in a Bash loop, convert them from SVG to PNG using ImageMagick’s `convert`, and optimise them with `pngcrush`, to produce images ready for upload.

A similar approach can be used for generating help files in various forms, for example, generating HTML files from Markdown source:

    docs: README.html credits.html

    README.html: README.md
        markdown README.md > README.html

    credits.html: credits.md
        markdown credits.md > credits.html

And perhaps finally deploying a website with `git push web`, but only *after* the icons are rasterized and the documents converted:

    deploy: icons docs
        git push web

For a more compact and abstract formula for turning a file of one suffix into another, you can use the `.SUFFIXES` pragma to define these using special symbols. The code for converting icons could look like this; in this case, `$<` refers to the source file, `$*` to the filename with no extension, and `$@` to the target.

    icons: create.png read.png update.png delete.png

    .SUFFIXES: .svg .png

    .svg.png:
        convert $< $*.raw.png && \
        pngcrush $*.raw.png $@

## Tools for building a `Makefile`

A variety of tools exist in the GNU Autotools toolchain for the construction of `configure` scripts and `make` files for larger software projects at a higher level, in particular [`autoconf`](http://en.wikipedia.org/wiki/Autoconf) and [`automake`](http://en.wikipedia.org/wiki/Automake). The use of these tools allows generating `configure` scripts and `make` files covering very large source bases, reducing the necessity of building otherwise extensive makefiles manually, and automating steps taken to ensure the source remains compatible and compilable on a variety of operating systems.

Covering this complex process would be a series of posts in its own right, and is out of scope of this survey.

*Thanks to user samwyse for the `.SUFFIXES` suggestion in the comments.*

# Debugging

When unexpected behaviour is noticed in a program, Linux provides a wide variety of command-line tools for diagnosing problems. The use of `gdb`, the GNU debugger, and related tools like the lesser-known Perl debugger, will be familiar to those using IDEs to set breakpoints in their code and to examine program state as it runs. Other tools of interest are available however to observe in more detail how a program is interacting with a system and using its resources.

## Debugging with `gdb`

You can use `gdb` in a very similar fashion to the built-in debuggers in modern IDEs like Eclipse and Visual Studio. If you are debugging a program that you’ve just compiled, it makes sense to compile it with its *debugging symbols* added to the binary, which you can do with a `gcc` call containing the `-g` option. If you’re having problems with some code, it helps to also use `-Wall` to show any errors you may have otherwise missed:

    $ gcc -g -Wall example.c -o example

The classic way to use `gdb` is as the shell for a running program compiled in C or C++, to allow you to inspect the program’s state as it proceeds towards its crash.

    $ gdb example
    ...
    Reading symbols from /home/tom/example...done.
    (gdb)

At the `(gdb)` prompt, you can type `run` to start the program, and it may provide you with more detailed information about the causes of errors such as segmentation faults, including the source file and line number at which the problem occurred. If you’re able to compile the code with debugging symbols as above and inspect its running state like this, it makes figuring out the cause of a particular bug a lot easier.

    (gdb) run
    Starting program: /home/tom/gdb/example 

    Program received signal SIGSEGV, Segmentation fault.
    0x000000000040072e in main () at example.c:43
    43     printf("%d\n", *segfault);

After an error terminates the program within the `(gdb)` shell, you can type `backtrace` to see what the calling function was, which can include the specific parameters passed that may have something to do with what caused the crash.

    (gdb) backtrace
    #0  0x000000000040072e in main () at example.c:43

You can set breakpoints for `gdb` using the `break` to halt the program’s run if it reaches a matching line number or function call:

    (gdb) break 42
    Breakpoint 1 at 0x400722: file example.c, line 42.
    (gdb) break malloc
    Breakpoint 1 at 0x4004c0
    (gdb) run
    Starting program: /home/tom/gdb/example 

    Breakpoint 1, 0x00007ffff7df2310 in malloc () from /lib64/ld-linux-x86-64.so.2

Thereafter it’s helpful to *step* through successive lines of code using `step`. You can repeat this, like any `gdb` command, by pressing Enter repeatedly to step through lines one at a time:

    (gdb) step
    Single stepping until exit from function _start,
    which has no line number information.
    0x00007ffff7a74db0 in __libc_start_main () from /lib/x86_64-linux-gnu/libc.so.6

You can even attach `gdb` to a process that is already running, by finding the process ID and passing it to `gdb`:

    $ pgrep example
    1524
    $ gdb -p 1524

This can be useful for [redirecting streams of output](http://stackoverflow.com/questions/593724/redirect-stderr-stdout-of-a-process-after-its-been-started-using-command-lin) for a task that is taking an unexpectedly long time to run.

## Debugging with `valgrind`

The much newer [valgrind](http://valgrind.org/) can be used as a debugging tool in a similar way. There are many different checks and debugging methods this program can run, but one of the most useful is its Memcheck tool, which can be used to detect common memory errors like buffer overflow:

    $ valgrind --leak-check=yes ./example
    ==29557== Memcheck, a memory error detector
    ==29557== Copyright (C) 2002-2011, and GNU GPL'd, by Julian Seward et al.
    ==29557== Using Valgrind-3.7.0 and LibVEX; rerun with -h for copyright info
    ==29557== Command: ./example
    ==29557== 
    ==29557== Invalid read of size 1
    ==29557==    at 0x40072E: main (example.c:43)
    ==29557==  Address 0x0 is not stack'd, malloc'd or (recently) free'd
    ==29557== 
    ...

The `gdb` and `valgrind` tools [can be used together](http://valgrind.org/docs/manual/manual-core-adv.html#manual-core-adv.gdbserver) for a very thorough survey of a program’s run. Zed Shaw’s [Learn C the Hard Way](http://c.learncodethehardway.org/book/) includes a really good introduction for elementary use of `valgrind` with a deliberately broken program.

## Tracing system and library calls with `ltrace`

The `strace` and `ltrace` tools are designed to allow watching system calls and library calls respectively for running programs, and logging them to the screen or, more usefully, to files.

You can run `ltrace` and have it run the program you want to monitor in this way for you by simply providing it as the sole parameter. It will then give you a listing of all the system and library calls it makes until it exits.

    $ ltrace ./example
    __libc_start_main(0x4006ad, 1, 0x7fff9d7e5838, 0x400770, 0x400760 
    srand(4, 0x7fff9d7e5838, 0x7fff9d7e5848, 0, 0x7ff3aebde320) = 0
    malloc(24)                                                  = 0x01070010
    rand(0, 0x1070020, 0, 0x1070000, 0x7ff3aebdee60)            = 0x754e7ddd
    malloc(24)                                                  = 0x01070030
    rand(0x7ff3aebdee60, 24, 0, 0x1070020, 0x7ff3aebdeec8)      = 0x11265233
    malloc(24)                                                  = 0x01070050
    rand(0x7ff3aebdee60, 24, 0, 0x1070040, 0x7ff3aebdeec8)      = 0x18799942
    malloc(24)                                                  = 0x01070070
    rand(0x7ff3aebdee60, 24, 0, 0x1070060, 0x7ff3aebdeec8)      = 0x214a541e
    malloc(24)                                                  = 0x01070090
    rand(0x7ff3aebdee60, 24, 0, 0x1070080, 0x7ff3aebdeec8)      = 0x1b6d90f3
    malloc(24)                                                  = 0x010700b0
    rand(0x7ff3aebdee60, 24, 0, 0x10700a0, 0x7ff3aebdeec8)      = 0x2e19c419
    malloc(24)                                                  = 0x010700d0
    rand(0x7ff3aebdee60, 24, 0, 0x10700c0, 0x7ff3aebdeec8)      = 0x35bc1a99
    malloc(24)                                                  = 0x010700f0
    rand(0x7ff3aebdee60, 24, 0, 0x10700e0, 0x7ff3aebdeec8)      = 0x53b8d61b
    malloc(24)                                                  = 0x01070110
    rand(0x7ff3aebdee60, 24, 0, 0x1070100, 0x7ff3aebdeec8)      = 0x18e0f924
    malloc(24)                                                  = 0x01070130
    rand(0x7ff3aebdee60, 24, 0, 0x1070120, 0x7ff3aebdeec8)      = 0x27a51979
    --- SIGSEGV (Segmentation fault) ---
    +++ killed by SIGSEGV +++

You can also attach it to a process that’s already running:

    $ pgrep example
    5138
    $ ltrace -p 5138

Generally, there’s quite a bit more than a couple of screenfuls of text generated by this, so it’s helpful to use the `-o` option to specify an output file to which to log the calls:

    $ ltrace -o example.ltrace ./example

You can then view this trace in a text editor like Vim, which includes syntax highlighting for `ltrace` output:

[![Vim session with ltrace output](http://blog.sanctum.geek.nz/wp-content/uploads/2012/02/ltrace-vim.png "ltrace-vim")](http://blog.sanctum.geek.nz/wp-content/uploads/2012/02/ltrace-vim.png)
Vim session with ltrace output

I’ve found `ltrace` very useful for debugging problems where I suspect improper linking may be at fault, or the absence of some needed resource in a `chroot` environment, since among its output it shows you its search for libraries at dynamic linking time and opening configuration files in `/etc`, and the use of devices like `/dev/random` or `/dev/zero`.

## Tracking open files with `lsof`

If you want to view what devices, files, or streams a running process has open, you can do that with `lsof`:

    $ pgrep example
    5051
    $ lsof -p 5051

For example, the first few lines of the `apache2` process running on my home server are:

    # lsof -p 30779
    COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
    apache2 30779 root  cwd    DIR    8,1     4096       2 /
    apache2 30779 root  rtd    DIR    8,1     4096       2 /
    apache2 30779 root  txt    REG    8,1   485384  990111 /usr/lib/apache2/mpm-prefork/apache2
    apache2 30779 root  DEL    REG    8,1          1087891 /lib/x86_64-linux-gnu/libgcc_s.so.1
    apache2 30779 root  mem    REG    8,1    35216 1079715 /usr/lib/php5/20090626/pdo_mysql.so
    ...

Interestingly, another way to list the open files for a process is to check the corresponding entry for the process in the dynamic `/proc` directory:

    # ls -l /proc/30779/fd

This can be very useful in confusing situations with file locks, or identifying whether a process is holding open files that it needn’t.

## Viewing memory allocation with `pmap`

As a final debugging tip, you can view the memory allocations for a particular process with `pmap`:

    # pmap 30779 
    30779:   /usr/sbin/apache2 -k start
    00007fdb3883e000     84K r-x--  /lib/x86_64-linux-gnu/libgcc_s.so.1 (deleted)
    00007fdb38853000   2048K -----  /lib/x86_64-linux-gnu/libgcc_s.so.1 (deleted)
    00007fdb38a53000      4K rw---  /lib/x86_64-linux-gnu/libgcc_s.so.1 (deleted)
    00007fdb38a54000      4K -----    [ anon ]
    00007fdb38a55000   8192K rw---    [ anon ]
    00007fdb392e5000     28K r-x--  /usr/lib/php5/20090626/pdo_mysql.so
    00007fdb392ec000   2048K -----  /usr/lib/php5/20090626/pdo_mysql.so
    00007fdb394ec000      4K r----  /usr/lib/php5/20090626/pdo_mysql.so
    00007fdb394ed000      4K rw---  /usr/lib/php5/20090626/pdo_mysql.so
    ...
    total           152520K

This will show you what libraries a running process is using, including those in shared memory. The total given at the bottom is a little misleading as for loaded shared libraries, the running process is not necessarily the only one using the memory; [determining “actual” memory usage for a given process](http://stackoverflow.com/questions/118307/a-way-to-determine-a-processs-real-memory-usage-i-e-private-dirty-rss) is a little more in-depth than it might seem with shared libraries added to the picture.

