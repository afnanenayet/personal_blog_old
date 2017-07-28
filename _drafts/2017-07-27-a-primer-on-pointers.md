---
layout: post
title: "A Primer on Pointers"
description: "Explaining C-style manual memory management and syntax"
date: 2017-07-27
tags: [C, pointers, C++, memory, management]
comments: true
---

A lot of people have trouble grasping manual memory management when they first 
encounter it. The syntax can be a little confusing and debugging can be 
an extremely painful process - whether you have a segfault or Valgrind is 
complaining that you *still* have memory leaks. 

Using pointers in C, while a little intimidating at first, is not as difficult 
as most people expect. 

C gives you a lot of access to memory - you can allocate bytes for use in 
your program, you can deallocate them, and you can directly access 
memorr addresses in your program 

# Syntax
We will first start by deifning the syntax and necessary functions that 
you need to allocate memory, declare pointers, get the address of a 
function, and free the memory that you have allocated. 

## Operators 

In C, you have two operators that pertain to memory management: `&` and `*`. 
The `*` allows you to declare pointers. 

## Functions

# Usage
