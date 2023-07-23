---
title: "LLVM for the impatient (1)"
date: 2023-02-04T14:42:43-08:00
tags: ["LLVM"]
categories: "LLVM"
draft: false
---

# Intro

LLVM is a common compiler backend. If you are writing a new language, you just need to target the LLVM backend. Then you will be able to use all the great optimization passes and target all the platforms that LLVM supports.

A best way to get started with LLVM is by following the [tutorial](https://llvm.org/docs/tutorial/MyFirstLanguageFrontend/index.html). The intended audience for this
series is:
1. Those who are learning from the tutorial but sometimes lost in the details. Many learners of LLVM are already familiar with basic concepts like parser, optimizers
and code generation. So I tried to bring out the most important points just about LLVM and thus focus in learning through the tutorial.
1. Those who work with LLVM occasionally and need a quick refresher (like me). Or sometimes when I need to a template to put together a POC.

I highly value the experimental approach to learn anything. So some important tips are:
1. The tutorial instructs us to build LLVM from the source. However, it takes hours to build LLVM so a much faster way is just to download the prebuilt binaries.
1. The LLVM API does change and the tutorials are often quickly updated from the `main`. To work with the source code, get the version compatible with the binary
from the source code [branch](https://github.com/llvm/llvm-project/tree/main/llvm/examples/Kaleidoscope).
1. Many compilers can emit LLVM IR. For example, [Clang](https://clang.llvm.org/) can emit LLVM IR with the `-emit-llvm` flag. Rust can emit LLVM IR with `--emit=llvm-ir`. This is a great way to learn LLVM IR.
1. LLVM has some great tools such as `llvm-as`, `opt` and `llvm-dis` can be used to experiment with optimizer and visualize LLVM.

# Getting the LLVM binaries

The [LLVM download page](https://releases.llvm.org/download.html) provides prebuilt binaries for a variety of operating systems and CPU architectures. On Linux or MacOS, we can download `clang+llvm*` for our platform. For Windows, download `llvm*-win64.exe`. If the download is a compressed archive, just expand into any directory.

On some platforms, it is possible package manager. For example, on macOS, we can install it with [Homebrew](https://formulae.brew.sh/formula/llvm).

The installers/package managers usually do not add Clang/LLVM to PATH and that is fine. It is likely we will need to work with multiple versions of LLVM. Some OS, such as macOS, has a system Clang, so we do not want to override it. I strongly recommend that we add Clang/LLVM in terminal sessions when we need them.

# Add Clang/LLVM to path

To add LLVM to path, just run:

```bash
export PATH=PATH_TO_LLVM/bin:$PATH
```

If we install LLVM on macOS with Brew, we can run

```bash
export PATH="$(brew --prefix llvm)/bin:$PATH"
```

# Other environment variables

To build C++ with LLVM libraries, we can configure the LDFLAGS environment variable to point to the libraries. This can be done with:

```bash
export LDFLAGS="-LPATH_TO_LLVM/lib"
```

Or with Brew installed LLVM:

```bash
export LDFLAGS="-L$(brew --prefix llvm)/lib"
```

Alternatively, we can also use the `llvm-config` utility that comes with LLVM:

```bash
clang++ `llvm-config --cxxflags` my_source.cpp
```

# Building LLVM tutorial code

Once setting up, we can now build our first LLVM source code. Let us save the [tutorial code](https://llvm.org/docs/tutorial/MyFirstLanguageFrontend/LangImpl02.html#full-code-listing) into a file, said, `toy.cpp`. Assuming Clang is already in the path, we can run:

```bash
# Compile
clang++ -g -O3 toy.cpp `llvm-config --cxxflags`
# Run
./a.out
```

Here -g means to generate debug symbol and -O3 is [optimization level 3](https://clang.llvm.org/docs/CommandGuide/clang.html).
We can now run Kaleidoscope Read-Eval-Print Loop (REPL):

```bash
$ ./a.out
ready> def foo(x y) x+foo(y, 4.0);
Parsed a function definition.
ready> def foo(x y) x+y y;
Parsed a function definition.
Parsed a top-level expr
ready> def foo(x y) x+y );
Parsed a function definition.
Error: unknown token when expecting an expression
ready> extern sin(a);
ready> Parsed an extern
ready> ^D
$
```

The first 2 chapters of [LLVM tutorial](https://llvm.org/docs/tutorial/MyFirstLanguageFrontend/index.html) are relatively straigh-forward. You should read through them if you are not already familiar with the material.

# Summary

In the first blog post of this series, we walk through how to compile and run the first LLVM tutorial. 
