# Introduction

Newbies and experienced professional programmers alike appreciate the concept of the IDE, or [integrated development environment](http://en.wikipedia.org/wiki/Integrated_development_environment). Having the primary tools necessary for organising, writing, maintaining, testing, and debugging code in an integrated application with common interfaces for all the different tools is certainly a very valuable asset. Additionally, an environment expressly designed for programming in various languages affords advantages such as autocompletion, and syntax checking and highlighting.

With such tools available to developers on all major desktop operating systems including Linux and BSD, and with many of the best free of charge, there’s not really a good reason to write your code in Windows Notepad, or with `nano` or `cat`.

However, there’s a minor meme among devotees of Unix and its modern-day derivatives that “Unix is an IDE”, meaning that the tools available to developers on the terminal cover the major features in cutting-edge desktop IDEs with some ease. Opinion is quite divided on this, but whether or not you feel it’s fair to call Unix an IDE in the same sense as Eclipse or Microsoft Visual Studio, it may surprise you just how comprehensive a development environment the humble Bash shell can be.

## How is UNIX an IDE?

The primary rationale for using an IDE is that it gathers all your tools in the same place, and you can use them in concert with roughly the same user interface paradigm, and without having to exert too much effort to make separate applications cooperate. The reason this becomes especially desirable with GUI applications is because it’s very difficult to make windowed applications speak a common language or work well with each other; aside from cutting and pasting text, they don’t share a *common interface*.

The interesting thing about this problem for shell users is that well-designed and enduring Unix tools already share a common user interface in *streams of text* and *files as persistent objects*, otherwise expressed in the axiom “everything’s a file”. Pretty much everything in Unix is built around these two concepts, and it’s this common user interface, coupled with a forty-year history of high-powered tools whose users and developers have especially prized interoperability, that goes a long way to making Unix as powerful as a full-blown IDE.

## The right idea

This attitude isn’t the preserve of battle-hardened Unix greybeards; you can see it in another form in the way the modern incarnations of the two grand old text editors Emacs and Vi (GNU Emacs and Vim) have such active communities developing plugins to make them support pretty much any kind of editing task. There are plugins to do pretty much anything you could really want to do in programming in both editors, and like any Vim junkie I could spout off at least six or seven that I feel are “essential”.

However, it often becomes apparent to me when reading about these efforts that the developers concerned are trying to make these text editors into IDEs in their own right. There are posts about [never needing to leave Vim](http://kevinw.github.com/2010/12/15/this-is-your-brain-on-vim/), or [never needing to leave Emacs](http://news.ycombinator.com/item?id=819447). But I think that trying to shoehorn Vim or Emacs into becoming something that it’s not isn’t quite thinking about the problem in the right way. Bram Moolenaar, the author of Vim, appears to agree to some extent, as you can see by reading [`:help design-not`](http://vimdoc.sourceforge.net/htmldoc/develop.html#design-not). The shell is only ever a Ctrl+Z away, and its mature, highly composable toolset will afford you more power than either editor ever could.

## About this series

In this series of posts, I will be going through six major features of an IDE, and giving examples showing how common tools available in Linux allow you to use them together with ease. This will by no means be a comprehensive survey, nor are the tools I will demonstrate the only options.

-   **File and project management** — `ls`, `find`, `grep`/`awk`, `bash`
-   **Text editor and editing tools** — `vim`, `awk`, `sort`, `column`
-   **Compiler and/or interpreter** — `gcc`, `perl`
-   **Build tools** — `make`
-   **Debugger** — `gdb`, `valgrind`, `ltrace`, `lsof`, `pmap`
-   **Version control** — `diff`, `patch`, `svn`, `git`

## What I’m not trying to say

I don’t think IDEs are bad; I think they’re brilliant, which is why I’m trying to convince you that Unix can be used as one, or at least thought of as one. I’m also not going to say that Unix is always the best tool for any programming task; it is arguably much better suited for C, C++, Python, Perl, or Shell development than it is for more “industry” languages like Java or C\#, especially if writing GUI-heavy applications. In particular, I’m not going to try to convince you to scrap your hard-won Eclipse or Microsoft Visual Studio knowledge for the sometimes esoteric world of the command line. All I want to do is show you what we’re doing on the other side of the fence.
