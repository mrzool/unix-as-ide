# Files

One prominent feature of an IDE is a built-in system for managing files, both the elementary functions like moving, renaming, and deleting, and ones more specific to development, like compiling and checking syntax. It may also be useful to have operations on sets of files, such as finding files of a certain extension or size, or searching files for specific patterns. In this first article, I’ll explore some useful ways to use tools that will be familiar to most Linux users for the purposes of working with sets of files in a project.

## Listing files

Using `ls` is probably one of the first commands an administrator will learn for getting a simple list of the contents of the directory. Most administrators will also know about the `-a` and `-l` switches, to show all files including dot files and to show more detailed data about files in columns, respectively.

There are other switches to GNU `ls` which are less frequently used, some of which turn out to be very useful for programming:

-   `-t` — List files in order of last modification date, newest first. This is useful for very large directories when you want to get a quick list of the most recent files changed, maybe piped through `head` or `sed 10q`. Probably most useful combined with `-l`. If you want the *oldest* files, you can add `-r` to reverse the list.
-   `-X` — Group files by extension; handy for polyglot code, to group header files and source files separately, or to separate source files from directories or build files.
-   `-v` — Naturally sort version numbers in filenames.
-   `-S` — Sort by filesize.
-   `-R` — List files recursively. This one is good combined with `-l` and pipedthrough a pager like `less`.

Since the listing is text like anything else, you could, for example, pipe the output of this command into a `vim` process, so you could add explanations of what each file is for and save it as an `inventory` file or add it to a README:

    $ ls -XR | vim -

This kind of stuff can even be automated by `make` with a little work, which I’ll cover in another article later in the series.

## Finding files

Funnily enough, you can get a complete list of files including relative paths with no decoration by simply typing `find` with no arguments, though it’s usually a good idea to pipe it through `sort`:

    $ find | sort
    .
    ./Makefile
    ./README
    ./build
    ./client.c
    ./client.h
    ./common.h
    ./project.c
    ./server.c
    ./server.h
    ./tests
    ./tests/suite1.pl
    ./tests/suite2.pl
    ./tests/suite3.pl
    ./tests/suite4.pl

If you want an `ls -l` style listing, you can add `-ls` as the action to `find` results:

    $ find -ls | sort -k 11
    1155096    4 drwxr-xr-x   4 tom      tom          4096 Feb 10 09:37 .
    1155152    4 drwxr-xr-x   2 tom      tom          4096 Feb 10 09:17 ./build
    1155155    4 -rw-r--r--   1 tom      tom          2290 Jan 11 07:21 ./client.c
    1155157    4 -rw-r--r--   1 tom      tom          1871 Jan 11 16:41 ./client.h
    1155159   32 -rw-r--r--   1 tom      tom         30390 Jan 10 15:29 ./common.h
    1155153   24 -rw-r--r--   1 tom      tom         21170 Jan 11 05:43 ./Makefile
    1155154   16 -rw-r--r--   1 tom      tom         13966 Jan 14 07:39 ./project.c
    1155080   28 -rw-r--r--   1 tom      tom         25840 Jan 15 22:28 ./README
    1155156   32 -rw-r--r--   1 tom      tom         31124 Jan 11 02:34 ./server.c
    1155158    4 -rw-r--r--   1 tom      tom          3599 Jan 16 05:27 ./server.h
    1155160    4 drwxr-xr-x   2 tom      tom          4096 Feb 10 09:29 ./tests
    1155161    4 -rw-r--r--   1 tom      tom           288 Jan 13 03:04 ./tests/suite1.pl
    1155162    4 -rw-r--r--   1 tom      tom          1792 Jan 13 10:06 ./tests/suite2.pl
    1155163    4 -rw-r--r--   1 tom      tom           112 Jan  9 23:42 ./tests/suite3.pl
    1155164    4 -rw-r--r--   1 tom      tom           144 Jan 15 02:10 ./tests/suite4.pl

Note that in this case I have to specify to `sort` that it should sort by the 11th column of output, the filenames; this is done with the `-k` option.

`find` has a complex filtering syntax all of its own; the following examples show some of the most useful filters you can apply to retrieve lists of certain files:

-   `find -name '*.c'` — Find files with names matching a shell-style pattern. Use `-iname` for a case-insensitive search.
-   `find -path '*test*'` — Find files with paths matching a shell-style pattern. Use `-ipath` for a case-insensitive search.
-   `find -mtime -5` — Find files edited within the last five days. You can use `+5` instead to find files edited *before* five days ago.
-   `find -newer server.c` — Find files more recently modified than `server.c`.
-   `find -type d` — Find directories. For files, use `-type f`; for symbolic links, use `-type l`.

Note, in particular, that all of these can be combined, for example to find C source files edited in the last two days:

    $ find -name '*.c' -mtime -2

By default, the action `find` takes for search results is simply to list them on standard output, but there are several other useful actions:

-   `-ls` — Provide an `ls -l` style listing, as above
-   `-delete` — Delete matching files
-   `-exec` — Run an arbitrary command line on each file, replacing `{}` with the appropriate filename, and terminated by `\;`; for example:

        $ find -name '*.pl' -exec perl -c {} \;

    You can use `+` as the terminating character instead if you want to put all of the results on one invocation of the command. One trick I find myself using often is using `find` to generate lists of files that I then edit in vertically split Vim windows:

        $ find -name '*.c' -exec vim {} +

*Earlier versions of Unix as IDE suggested the use of `xargs` with `find` results. In most cases this should not really be necessary, and it’s more robust to handle filenames with whitespace using `-exec` or a `while read -r` loop.*

## Searching files

More often than *attributes* of a set of files, however, you want to find files based on their *contents*, and it’s no surprise that `grep`, in particular `grep -R`, is useful here. This searches the current directory tree recursively for anything matching ‘someVar’:

    $ grep -FR someVar .

Don’t forget the case insensitivity flag either, since by default `grep` works with fixed case:

    $ grep -iR somevar .

Also, you can print a list of files that match without printing the matches themselves with `grep -l`:

    $ grep -lR someVar .

If you write scripts or batch jobs using the output of the above, use a `while` loop with `read` to handle spaces and other special characters in filenames:

    grep -lR someVar | while IFS= read -r file; do
        head "$file"
    done

If you’re using version control for your project, this often includes metadata in the `.svn`, `.git`, or `.hg` directories. This is dealt with easily enough by *excluding* (`grep -v`) anything matching an appropriate fixed (`grep -F`) string:

    $ grep -R someVar . | grep -vF .svn

Some versions of `grep` include `--exclude` and `--exclude-dir` options, which may be tidier.

With all this said, there’s a very popular [alternative to grep](http://betterthangrep.com/) called `ack`, which excludes this sort of stuff for you by default. It also allows you to use Perl-compatible regular expressions (PCRE), which are a favourite for many hackers. It has a lot of utilities that are generally useful for working with source code, so while there’s nothing wrong with good old `grep` since you know it will always be there, if you can install `ack` I highly recommend it. There’s a Debian package called `ack-grep`, and being a Perl script it’s otherwise very simple to install.

Unix purists might be displeased with my even mentioning a relatively new Perl script alternative to classic `grep`, but I don’t believe that the Unix philosophy or using Unix as an IDE is dependent on sticking to the same classic tools when alternatives with the same spirit that solve new problems are available.

## File metadata

The `file` tool gives you a one-line summary of what kind of file you’re looking at, based on its extension, headers and other cues. This is very handy used with `find` when examining a set of unfamiliar files:

    $ find -exec file {} \;
    .:            directory
    ./hanoi:      Perl script, ASCII text executable
    ./.hanoi.swp: Vim swap file, version 7.3
    ./factorial:  Perl script, ASCII text executable
    ./bits.c:     C source, ASCII text
    ./bits:       ELF 32-bit LSB executable, Intel 80386, version ...

## Matching files

As a final tip for this section, I’d suggest learning a bit about pattern matching and brace expansion in Bash, which you can do in my earlier post entitled [Bash shell expansion](http://blog.sanctum.geek.nz/bash-shell-expansion/).

All of the above make the classic UNIX shell into a pretty powerful means of managing files in programming projects.

