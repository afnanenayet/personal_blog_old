---
layout: post
title: "Debugging C with GDB and Valgrind"
description: "Looking at how you can use GDB/LLDB with Valgrind to debug C code"
date: 2017-07-29
tags: [C, lldb, gdb, valgrind, memory, leaks, segfault, debugging, debug]
comments: true
---

Debugging C can be a chore, but being able to pinpoint your memory leaks with 
Valgrind and monitoring the flow of your program with GDB (or LLDB) can 
speed up the development of your code significantly. It's a significant 
improvement over sticking a bunch of `printf` statements in your code and 
taking them out before production (which should by no means be your sole 
tool for debugging).

# Setup

Installing gdb is pretty easy on both OS X and Linux. Use your favorite package 
manager to install [gcc](https://gcc.gnu.org) and 
[gdb](https://www.gnu.org/software/gdb/). You can also install the 
[LLVM](https://llvm.org) compiler collection 
which uses lldb as its debugging tool rather than gdb. 

Installing [Valgrind](http://valgrind.org) is similar - fire up your favorite 
package manager, build from source, or download a release directly from the 
webpage. On MacOS, there are some caveats if you're trying to install 
Valgrind on MacOS > 10.12. 

# Usage

## GDB/LLDB

GDB and LLDB allow a user to scrutizine the execution of their programs 
line by line. You can look at the state of your program, inspecting variables, 
seeing what is executing, and even evaluating arbitrary statements based on 
the state of your program. We will examine how to use these programs. 

### The basics

Both GDB/LLDB provide some basic debugging functions like:
- step next
- step over
- continue execution
- step into
- break

GDB allows you to inspect your code by viewing its state as it executes 
on a per line basis. You can step through your program line by line. More 
commonly, if you suspect that your problem is in a particular segment of 
code, you can create a breakpoint at a function or a line number. You can 
do some powerful things while you step through your program like inspect 
what variables are in your program, and execute arbitrary statements 
based on the state of your program. 

GDB/LLDB gives you access to certain key commands that will help you debug your 
program. `next` will execute the current line, and the pause at the next line. 
This will not go into any functions, it will block gdb until the instructions on 
the current line have been executed. `step` (aka step into) will actually step 
into the function on the current line. If you have a function call on the current 
line, then GDB will "step into" that function, allowing you to continue debugging 
line by line inside the function. `finish` is like stepping out - it will continue 
execution until the function of the current line finishes executing/until it 
returns. Look below for reference sheets for GDB and LLDB. 

### Usage

In order to run gdb, you need to make sure that your binary has debug symbols 
so that GDB knows the relationship between what is being executed and your 
code. You compiling with optimization flags (`-OX`) can cause issues because 
an optimization may not necessarily stay true to your code. For example, 
a compiler will not keep unused variables because they waste stack space. 
To stay safe, compile debug builds without any optimizations. You can 
use an optimization flag with the debug symbols. The flag is `-g`, so 
when compiling with gcc, add the `-g` flag to your list of flags. We will 
explore an example with an example program I created. 

### Example

#### Setup

You can download my example source code [here](https://gist.github.com/afnanenayet/1c0a71b40a67b63a42773fe2a9539f5e)

You can compile the code with this command

```sh
gcc -g gdb_valgrind_example.c -o gve -Wall -Wextra -std=c11 -pedantic 
-Wmissing-prototypes -Wstrict-prototypes -Wold-style-definition
```

Try running the code. It seems like it should work. It's simple, it just 
allocates a string and initializes it to `"hello"`. 

You can run the code with the following command:
```
./gve
```

The output I got was:

```
This next string should be blank...
zsh: segmentation fault (core dumped)  ./gve
```

So what happened? Let's step through this program with GDB. Using LLDB should 
be extremely similar. 

#### Debug

In order to start running gdb, we have to give it a program to run. GDB 
takes a binary followed by its arguments to initialize the debug runtime.

```
gdb --args gve
```

If we had arguments, we'd use something like `gdb --args gve arg1 arg2 ...`

Now GDB is initialized with our program, `gve` (*G*DB *V*algrind *E*xample)

Here's what you'll see at first (or something like this)

```
GNU gdb (GDB) Fedora 7.12.1-48.fc25
Copyright (C) 2017 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-redhat-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from gve...(no debugging symbols found)...done.
(gdb)
```

We want to set a breakpoint at the `main` function. Otherwise, GDB will just 
run without giving us anything useful to look at.

```
(gdb) break main
Breakpoint 1 at 0x400577
```

Let's run the program, (just type in `run`)

```
Starting program: /net/ifs-users/afnan/gve

Breakpoint 1, 0x0000000000400577 in main ()
```

Now GDB is frozen at the beginning of the `main` function. Let's step through 
the main function and see what's happening to the `str` variable before and 
after we call `init_str`. 

```
Breakpoint 1, main () at gdb_valgrind_example.c:32
32	    char *str = NULL;
(gdb) n
33	    printf("This next string should be blank...\n");
(gdb) n
This next string should be blank...
34	    init_str(str);
```

Right before we call `init_str`, let's check the value of `str` by calling 
`p str`

```
(gdb) p str
$1 = 0x0
```

We can see that `str` is `NULL`, as it was assigned. Let's step and see what 
happens after we call `init_str`. Theoretically, `str` should be changed 
to "hello" after `init_str` is called.

```
(gdb) n
35	    printf("%s\n", str);
(gdb) p str
$3 = 0x0
```

We can see that `str` is still `NULL`. Now we know the function is not working 
as intended. We could do two things: we could rerun the program and step until 
we reach the `init_str` function call, or we could just set a breakpoint 
directly at the function. Let's set a breakpoint, since it'll be faster.

We've already gone past the function call, so we will have to restart the 
program. Let's set a breakpoint at `init_str` then restart the program. 

```
(gdb) break init_str
Breakpoint 2 at 0x400552: file gdb_valgrind_example.c, line 25.
(gdb) run
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Starting program: /net/ifs-users/afnan/gve

Breakpoint 1, main () at gdb_valgrind_example.c:32
32	    char *str = NULL;
(gdb)
```

Our breakpoint set earlier is still there (at `main`). We don't want to 
step all the way to the function, so let's continue until we hit the 
next breakpoint. 

```
(gdb) c
Continuing.
This next string should be blank...

Breakpoint 2, init_str (str=0x0) at gdb_valgrind_example.c:25
25	    str = malloc(5);
```

Now we've reached the start of the `init_str` function. GDB tells us which 
file and line the breakpoint is one. It also tells us the value of the argument:
`str = 0x0`. So we know `str=NULL` when `init_str` is starting. Let's step through 
the function, inspecting the variables as we go through, so we can see what's going 
on.

```
25	    str = malloc(5);
(gdb) n
26	    strcpy(str, "hello");
(gdb) p str
$4 = 0x602420 ""
(gdb) n
27	}
(gdb) p str
$5 = 0x602420 "hello"
(gdb) n
main () at gdb_valgrind_example.c:35
35	    printf("%s\n", str);
(gdb) p str
$6 = 0x0
```

It looks like `str` gets allocated correctly as evidence by `$4`. It is assigned 
correctly with `strcpy`, but when we return, `str` becomes `NULL` again.

This means that we're not actually changing `str` outside of the scope of 
`init_str`. This is because we need some pointer indirection - to affect a 
variable outside of its enclosing scope, we need a pointer to that variable.
It's essentially the same here: we need a pointer to our `char*`.

We can easily rectify the problem by passing a reference to our string rather 
than passing in the string directly. This is called indirection. Let's 
fix the program:

```c
/* gdb_valgrind_example.c    Afnan Enayet
 *
 * An example designed to help a user understand how to
 * use GDB/LLDB and Valgrind
 *
 * Functions:
 *   - init_str: a functions that initializes a string 
 *     to "hello"
 *
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/****** Function prototypes ******/

static void init_str(char **str);

/****** Function definitions *****/

/* Initializes and creates a string that says "hello"
*/
static void init_str(char **str)
{
    *str = malloc(5);
    strcpy(*str, "hello");
}

int main(void)
{
    // Allocating a string with the literal "hello"
    char *str = NULL;
    printf("This next string should say hello...\n");
    init_str(&str);
    printf("%s\n", str);
    return 0;
}
```

Let's go ahead and check that the program is working as intended. Recompile 
the file and load it with GDB. Let's set a breakpoint right before and 
right after the function call so we can see if `str` was allocated/set 
properly.

```
(gdb) break 33
Breakpoint 1 at 0x40058c: file gdb_valgrind_example.c, line 33.
(gdb) break 35
Breakpoint 2 at 0x4005a2: file gdb_valgrind_example.c, line 35.
(gdb) r
Starting program: /net/ifs-users/afnan/gve

Breakpoint 1, main () at gdb_valgrind_example.c:33
33	    printf("This next string should say hello...\n");
(gdb) p str
$1 = 0x0
(gdb) n
This next string should say hello...
34	    init_str(&str);
(gdb) n

Breakpoint 2, main () at gdb_valgrind_example.c:35
35	    printf("%s\n", str);
(gdb) p str
$2 = 0x602420 "hello"
```
We can see that the string now reads "hello". The program has been corrected.

-------------------------------------------------------------------------------

You can find a reference for these commands (in GDB)
[here](http://darkdust.net/files/GDB%20Cheat%20Sheet.pdf). 
[Here](http://lldb.llvm.org/lldb-gdb.html)
is a cheat sheet from LLVM with LLDB commands and their gdb equivalents.

## Valgrind

Even if a program is working correctly or appears to be working correctly, memory 
errors could be present that you'll never see, or will pop up seemingly randomly
in a manner that makes it difficult to diagnose with GDB. 

Valgrind is a tool that inspects your binary to see if there are any memory errors 
or leaks. It's a very powerful tool that you should use to inspect your programs 
to ensure program correctness. 

### The basics

Valgrind contains a number of tools that you can use to help you diagnose/inspect 
your program. You can initialize a certain tool by passing an argument flag to 
Valgrind. We're going to use memcheck, which is described 
[here](http://valgrind.org/info/tools.html). There are other tools available that 
are very helpful for increasing the performance of your C/C++ programs, but this 
particular post will not cover them. The memcheck tool will run your program, display 
output, and display errors as they arise while the program is executing.

The example usage for our program would be `valgrind --tool=memcheck 
--leak-check=full`

Let's try it

```
==27065== Memcheck, a memory error detector
==27065== Copyright (C) 2002-2015, and GNU GPL'd, by Julian Seward et al.
==27065== Using Valgrind-3.12.0 and LibVEX; rerun with -h for copyright info
==27065== Command: ./gve
==27065==
This next string should say hello...
==27065== Invalid write of size 2
==27065==    at 0x400573: init_str (gdb_valgrind_example.c:26)
==27065==    by 0x4005A1: main (gdb_valgrind_example.c:34)
==27065==  Address 0x5200484 is 4 bytes inside a block of size 5 alloc'd
==27065==    at 0x4C2DB9D: malloc (vg_replace_malloc.c:299)
==27065==    by 0x40055B: init_str (gdb_valgrind_example.c:25)
==27065==    by 0x4005A1: main (gdb_valgrind_example.c:34)
==27065==
==27065== Invalid read of size 1
==27065==    at 0x4C30BC4: strlen (vg_replace_strmem.c:454)
==27065==    by 0x4EAAA61: puts (ioputs.c:35)
==27065==    by 0x4005AD: main (gdb_valgrind_example.c:35)
==27065==  Address 0x5200485 is 0 bytes after a block of size 5 alloc'd
==27065==    at 0x4C2DB9D: malloc (vg_replace_malloc.c:299)
==27065==    by 0x40055B: init_str (gdb_valgrind_example.c:25)
==27065==    by 0x4005A1: main (gdb_valgrind_example.c:34)
==27065==
hello
==27065==
==27065== HEAP SUMMARY:
==27065==     in use at exit: 5 bytes in 1 blocks
==27065==   total heap usage: 2 allocs, 1 frees, 1,029 bytes allocated
==27065==
==27065== 5 bytes in 1 blocks are definitely lost in loss record 1 of 1
==27065==    at 0x4C2DB9D: malloc (vg_replace_malloc.c:299)
==27065==    by 0x40055B: init_str (gdb_valgrind_example.c:25)
==27065==    by 0x4005A1: main (gdb_valgrind_example.c:34)
==27065==
==27065== LEAK SUMMARY:
==27065==    definitely lost: 5 bytes in 1 blocks
==27065==    indirectly lost: 0 bytes in 0 blocks
==27065==      possibly lost: 0 bytes in 0 blocks
==27065==    still reachable: 0 bytes in 0 blocks
==27065==         suppressed: 0 bytes in 0 blocks
==27065==
==27065== For counts of detected and suppressed errors, rerun with: -v
==27065== ERROR SUMMARY: 4 errors from 3 contexts (suppressed: 0 from 0)
```

We have a memory leak! We've lost 5 bytes from where we called `malloc` in 
the `init_str` method. This means that we forgot to free the string that 
we allocated. Let's fix this and then run Valgrind again. 

Here is the corrected program:

```c
/* gdb_valgrind_example.c    Afnan Enayet
 *
 * An example designed to help a user understand how to
 * use GDB/LLDB and Valgrind
 *
 * Functions:
 *   - change_str: a functions that converts all of the
 *     characters in a string to spaces
 *
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/****** Function prototypes ******/
static void init_str(char **str);

/****** Function definitions *****/

/* Initializes and creates a string that says "hello"
*/
static void init_str(char **str)
{
    *str = malloc(5 * sizeof(char));
    strcpy(*str, "hello");
}

int main(void)
{
    // Allocating a string with the literal "hello"
    char *str = NULL;
    printf("This next string should say hello...\n");
    init_str(&str);
    printf("%s\n", str);
    free(str);
    return 0;
}
```

Here is the Valgrind output:

```
[afnan@moose]~% valgrind --tool=memcheck --leak-check=full ./gve
==27500== Memcheck, a memory error detector
==27500== Copyright (C) 2002-2015, and GNU GPL'd, by Julian Seward et al.
==27500== Using Valgrind-3.12.0 and LibVEX; rerun with -h for copyright info
==27500== Command: ./gve
==27500==
This next string should say hello...
==27500== Invalid write of size 2
==27500==    at 0x4005B3: init_str (gdb_valgrind_example.c:26)
==27500==    by 0x4005E1: main (gdb_valgrind_example.c:34)
==27500==  Address 0x5200484 is 4 bytes inside a block of size 5 alloc'd
==27500==    at 0x4C2DB9D: malloc (vg_replace_malloc.c:299)
==27500==    by 0x40059B: init_str (gdb_valgrind_example.c:25)
==27500==    by 0x4005E1: main (gdb_valgrind_example.c:34)
==27500==
==27500== Invalid read of size 1
==27500==    at 0x4C30BC4: strlen (vg_replace_strmem.c:454)
==27500==    by 0x4EAAA61: puts (ioputs.c:35)
==27500==    by 0x4005ED: main (gdb_valgrind_example.c:35)
==27500==  Address 0x5200485 is 0 bytes after a block of size 5 alloc'd
==27500==    at 0x4C2DB9D: malloc (vg_replace_malloc.c:299)
==27500==    by 0x40059B: init_str (gdb_valgrind_example.c:25)
==27500==    by 0x4005E1: main (gdb_valgrind_example.c:34)
==27500==
hello
==27500==
==27500== HEAP SUMMARY:
==27500==     in use at exit: 0 bytes in 0 blocks
==27500==   total heap usage: 2 allocs, 2 frees, 1,029 bytes allocated
==27500==
==27500== All heap blocks were freed -- no leaks are possible
==27500==
==27500== For counts of detected and suppressed errors, rerun with: -v
==27500== ERROR SUMMARY: 3 errors from 2 contexts (suppressed: 0 from 0)
```

There are some errors, but we fixed the memory leak! I will address these 
errors later or in another blog post. 

Happy debugging!


