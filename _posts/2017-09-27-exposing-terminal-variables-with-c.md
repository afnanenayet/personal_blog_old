---
layout: post
title: "Exposing terminal variables with C"
description: ""
date: 2017-09-27
tags: [c, fun, terminal, memory]
comments: true
---

I was writing a C program that took in a arguments, and I was printing out these
arguments when I printed out more arguments than were given. I noticed some
interesting output -- all of my terminal variables.

# Code

The code to print out a number of variables is as follows, (I decided to
continue printing variables through `UNSIGNED_LONG`);

```c
/* Afnan Enayet    leak_data.c
*
* A script written to read terminal variables
*/

#include <stdio.h>
#include <limits.h>

/****** function prototypes ******/

void print_vars(char **args);

/****** function definitions ******/

int main(int argc, char *argv[])
{
print_vars(argv);
return 0;
}

void print_vars(char **args)
{
const unsigned long loop_limit = ULONG_MAX;

// Reading arguments from memory addresses outside of the program
// (should bleed into term vars)
// Prints lines in the following format...
// address: string corresponding to that address
for (int i = 0; i < loop_limit; i++) {
printf("%p: %s\n", args + i, args[i]);
}
}
```

Compilation: `clang leak_data.c -o leak`

# What it gives us

The code runs and prints out the first couple of arguments, if any were supplied.
The first argument will be the name of the program. The next couple of lines will
be the arguments that were supplied. After that, things get interesting. Let's
look at the output I got.

I ran `./leak arg1 arg2 arg3`

```sh
0x7ffeebc30820: ./leak
0x7ffeebc30828: arg1
0x7ffeebc30830: arg2
0x7ffeebc30838: arg3
0x7ffeebc30840: (null)
0x7ffeebc30848: TERM_SESSION_ID=w1t0p0:2143472D-FCA6-48BF-986B-C4BC658CE8A1
0x7ffeebc30850: SSH_AUTH_SOCK=/private/tmp/com.apple.launchd.Eger9zAfyh/Listeners
0x7ffeebc30858: Apple_PubSub_Socket_Render=/private/tmp/com.apple.launchd.kys2B1tfHZ/Render
0x7ffeebc30860: COLORFGBG=12;8
0x7ffeebc30868: ITERM_PROFILE=Default
0x7ffeebc30870: XPC_FLAGS=0x0
0x7ffeebc30878: LANG=en_US.UTF-8
0x7ffeebc30880: PWD=/Users/aenayet/Documents/Programming/ptr_hack
0x7ffeebc30888: SHELL=/bin/zsh
0x7ffeebc30890: TERM_PROGRAM_VERSION=3.1.2
0x7ffeebc30898: TERM_PROGRAM=iTerm.app
0x7ffeebc308a0: PATH=/Users/aenayet/.cargo/bin:/usr/local/opt/curl/bin:/usr/local/sbin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/TeX/texbin:/usr/local/MacGPG2/bin:/opt/X11/bin
0x7ffeebc308a8: DISPLAY=/private/tmp/com.apple.launchd.N2bb7ZxPPp/org.macosforge.xquartz:0
0x7ffeebc308b0: COLORTERM=truecolor
0x7ffeebc308b8: TERM=xterm-256color
0x7ffeebc308c0: HOME=/Users/aenayet
0x7ffeebc308c8: TMPDIR=/var/folders/1x/hmr55rhn6t79cz0bd5jkrsy40000gn/T/
0x7ffeebc308d0: USER=aenayet
0x7ffeebc308d8: XPC_SERVICE_NAME=0
0x7ffeebc308e0: LOGNAME=aenayet
0x7ffeebc308e8: __CF_USER_TEXT_ENCODING=0x0:0:0
0x7ffeebc308f8: SHLVL=1
0x7ffeebc30900: OLDPWD=/Users/aenayet/Documents/Programming
0x7ffeebc30908: ZSH=/Users/aenayet/.oh-my-zsh
0x7ffeebc30910: PAGER=less
0x7ffeebc30918: LESS=-R
0x7ffeebc30920: LC_CTYPE=en_US.UTF-8
0x7ffeebc30928: LSCOLORS=Gxfxcxdxbxegedabagacad
0x7ffeebc30930: AUTOJUMP_SOURCED=1
0x7ffeebc30938: AUTOJUMP_ERROR_PATH=/Users/aenayet/Library/autojump/errors.log
0x7ffeebc30940: EDITOR=nvim
0x7ffeebc30948: ARCHFLAGS=-arch x86_64
0x7ffeebc30958: RUST_SRC=~/rust_src/rust
0x7ffeebc30960: RUST_SRC_PATH=~/rust_src/rust
0x7ffeebc30968: TEST=hello
0x7ffeebc30970: _=/Users/aenayet/Documents/Programming/ptr_hack/./leak
0x7ffeebc30978: (null)
0x7ffeebc30980: executable_path=./leak
0x7ffeebc30988:
0x7ffeebc30990:
0x7ffeebc30998:
0x7ffeebc309a0: main_stack=
0x7ffeebc309a8: executable_file=0x1801000004,0x2000d10b1
0x7ffeebc309b0: dyld_file=0x1801000004,0x2000b1132
0x7ffeebc309b8: (null)
```

We can see that `arg1`, `arg2`, and `arg3` are in the memory addresses they're
supposed to be in. They correspond to `argv[1]`, `argv[2]`, and `argv[3]`,
respectively. You can see that everything after that is a terminal variable.
It contains every accessible `$VARIABLE`.

This isn't a security leak, since the path and other variables are need to
be accessible to programs anyway. I thought it was interesting because I
haven't ever noticed this before, and I enjoy tinkering with things like
this. You can see that the address of the executable file is also supplied at
the end of the stack. After that, the program tries to access restricted
memory, and segfaults.

