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

They utilize different key commands. You can find a reference for these commands 
[here]().

## Valgrind

### The basics

