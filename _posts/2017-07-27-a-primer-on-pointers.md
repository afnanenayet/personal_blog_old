---
layout: post
title: "A Primer on Pointers"
description: "Explaining C-style manual memory management and syntax"
date: 2017-07-27
tags: [C, pointers, C++, memory, management, malloc, free]
comments: true
---

A lot of people have trouble grasping manual memory management when they first 
encounter it. The syntax can be a little confusing and debugging can be 
an extremely painful process - whether you have a segfault or Valgrind is 
complaining that you *still* have memory leaks. 

Using pointers in C, while a little intimidating at first, is not as difficult 
as most people expect. 

C gives you a lot of access to memory - you can allocate bytes for use in 
your program, you can deallocate them, you can directly access 
memory addresses in your program, and pass them around wherever you want. 
The possibilities are endless!

# A brief overview

So what exactly is a pointer?

A pointer is an operator in C that lets you "point" to a particular location in 
memory. This is a powerful construct that lets you directly manipulate memory, 
and pass references to the same location in memory to multiple functions, so you 
can directly modify variables in other functions, or just avoid unecessarily 
making a copy of a parameter to save memory. 

Let's pretend we have an array of integers

```c
int arr[] = {0, 1, 2, 3, 4};
int *ptr = &arr; // this is a pointer to the first element of arr
```

`arr` is an array of integers. Each element takes up some space in memory. 
In C, arrays are all contiguous, so every element is next to each other 
in memory. We can represent this in array in memory as such:

| 0x0 | 0x1 | 0x2 | 0x3 | 0x4 | 
| 0   | 1   | 2   | 3   | 4 |

We can access the second element in `arr` in two diferent ways:

```c
int first_elem = arr[1];
int first_elem_ptr = *(ptr + 1);
```

Because `ptr` points to the second element of `arr`, adding 1 to it brings it from 
address `0x0` to `0x1`, which is where you will find `1`. 

You can pass this pointer to other functions as well, and directly modify 
this array without returning/copying over a new array. 

# Syntax

We will first start by defining the syntax and necessary functions that 
you need to allocate memory, declare pointers, get the address of a 
function, and free the memory that you have allocated. 

## Operators 

In C, you have two operators that pertain to memory management: `&` and `*`. 
The `*` allows you to declare a pointer variable type. If you use the `*` on 
a pointer after it has already been declared, you will get the value that it 
references. This is called dereferencing. The `&` operator returns the address 
of a variable, so it will return a pointer to the variable it is being used on.

Here is an example of the operators in action:

```c
int integer = 1; // this is a normal variable
int *ptr; // declaring a pointer
ptr = &integer; // this points to the address of the integer
*ptr; // this returns the value that ptr points to, which is "1"
(*ptr == integer); // "true"
```

## Functions

There are two functions that you have to know use memory management in C:
`malloc` and `free`. `malloc` allocates a block of memory. It takes an 
argument which tells it how much memory to allocate (in bytes). 
`free` is called on a variable that has been initialized by `malloc` - 
it frees the memory that was being used. You want to free this memory 
up to be reclaimed by the operating system so you can avoid memory 
leaks. 

You will also want to make sure that everything that has been `malloc`'d 
is also `free`'d, and that you don't `free` a block of memory that has 
already had `free` called on it. 

Here are the references for [malloc](https://www.gnu.org/software/libc/manual/html_node/Basic-Allocation.html) 
and [free](http://en.cppreference.com/w/c/memory/free).

These functions are found in `stdlib.h`, so if you are using these 
functions in your C program, you will have to prepend the file with 
`#include <stdlib.h>`.

# Usage

Let's take a look at an example that utilizes the functions and operators that 
were just discussed. 

```c
#include <stdio.h> // printf
#include <stdlib.h> // free, malloc

int main() 
{    
    int *arr = malloc(sizeof(int) * 5); // allocate space for 5 integers

    // Setting each element in the array equal to the value of its index
    // this should yield {0, 1, 2, 3, 4}
    for (int i = 0; i < 5; i++) {
        *(arr + i) = i;
    }

    free(arr); // free the array we allocated

    int some_int = 0; 
    int *int_ptr = &some_int; // this pointer is set equal to the address of some_int
    int another_int = *int_ptr; // another_int = value that int_ptr points to
    int *another_int_ptr = &some_int;

    (int_ptr == another_int_ptr) // true
    (int_ptr = another_int) // false
    (another_int == some_int) // true
    (*another_int_ptr == some_int == another_int == *int_ptr) // true
    return 0;
}
```

This covers some of the basics of allocating and deallocating memory in C. 
Stay tuned for a primer on using GDB/LLDB and Valgrind. 

