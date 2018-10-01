---
layout: post
title: "Using the Latest LLVM Release on MacOS"
description: "Reliably use the latest clang tooling on your Mac"
date: 2018-10-01
tags: [llvm, clang, compilers, macos, c, c++]
comments: true
---

MacOS is really frustrating with how it handles its libraries and compilers.
It is also frustrating because it ships an unspecified version of LLVM,
which generally isn't the latest stable release. You can, however, with a
little tweaking, use the latest version of LLVM or GCC on your Mac, and
reliably use it for your C and C++ tooling.

## Installation

First, you need to install the latest version of LLVM. Most people nowadays
are using [Homebrew](brew.sh). If you don't have it, you can install LLVM from source,
which takes a lot of time to compile. If you have brew, you can install LLVM
with

```sh
brew install llvm
```

You have several options for installing LLVM, all of which will require brew
to compile it from source. I usually install the latest stable version of
LLDB and I build it against Homebrew's Python 2.

```sh
brew install llvm --with-python@2 --with-lldb
```

This took me roughly two hours to build on a 2015 rMBP 13" with an i5.

Once it's done, add Brew's LLVM to your path:

```sh
echo 'export PATH="/usr/local/opt/llvm/bin:$PATH"' >> ~/.zshrc
# or ~/.bashrc depending on your shell
```

## Environment variables

Most build tools (namely CMake and Make) respect certain environment
variables, so you should set these once you have the LLVM suite in
your path.

I usually go ahead and set up aliases so I can conveniently compile small
programs from the command line with the latest version of clang.

For environment variables:

```sh
export CC=clang
export CXX=clang++
export LD=ld.lld
export AR=llvm-ar
export RANLIB=llvm-ranlib
```

_Note: if you don't have LLVM in your `$PATH`, then you will need to add the
full path to your environment variables_

These are the aliases I set:

```sh
alias cc=$CC
alias c++=$CXX
alias ld=$LD
alias ar=$AR
alias ranlib=$RANLIB
```

## Working with XCode

If you use XCode or `xcodebuild`, then you will realize that it does not use
the versions of LLVM/Clang that you set up in the environment. The LLVM
project actually provides a way to build an XCode toolchain that contains
everything you need to switch XCode to the latest versions of clang and the
other tools you need to compile your projects.

In order to do this, you need to manually build LLVM from source and build
the toolchain.

You can follow this tutorial: https://quuxplusone.github.io/blog/2018/04/16/building-llvm-from-source/
which shows you how to build LLVM from the Github mirrors.

When you get to section 3, change the cmake command slightly so you enable the
XCode toolchain target:

```sh
mkdir -p build
cd build
cmake -G Ninja \
	-DCMAKE_BUILD_TYPE=RelWithDebInfo \
	-DLLVM_CREATE_XCODE_TOOLCHAIN=On
```

I also recommend using `Ninja` rather than `Make` to build LLVM, because it
will build significantly faster.

Now that you have the XCode toolchain, you can place it in the `Toolchains`
directory in XCode.

```sh
mv LLVM7.0.0.xctoolchain /Applications/Xcode.app/Contents/Developer/Toolchains
```

You need to instruct XCode to actually use the toolchain. You can do so in two
ways: from your environment variables, and through the XCode app itself.

To set it through an environment variable:

```sh
export TOOLCHAINS="LLVM7.0.0"
```

In `Xcode.app`, you can select `Xcode -> Toolchains -> org.llvm.7.0.0` in the
menu.

These steps should generally work with your standard C/C++ Make, CMake, and
Xcode setups, though there can be a lot of quirks because of the way people
install libraries and how Apple sets up their compiler suite.
