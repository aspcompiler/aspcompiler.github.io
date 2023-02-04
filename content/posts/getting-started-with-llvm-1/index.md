---
title: "Getting Started With LLVM (1)"
date: 2023-02-04T14:42:43-08:00
tags: ["LLVM"]
categories: "LLVM"
draft: false
---

# Why should I learn LLVM?

According to ChatGPT, there are several reasons why you should learn the LLVM compiler framework:

1. Popularity: LLVM is widely used in the industry, making it an important skill for those working in software development or computer science.

1. Versatility: LLVM can be used for a wide range of programming languages, making it a valuable tool for those who work with multiple languages.

1. Performance: LLVM is known for its high-performance code generation, making it a valuable tool for optimizing software applications.

1. Open-source: LLVM is an open-source framework, which means that you have access to the source code and can make modifications to fit your needs.

1. Community: LLVM has a large and active community of developers and users, making it easier to find support and resources when working with the framework.

1. Career opportunities: Knowledge of LLVM can help you stand out in the job market and open up new career opportunities in areas such as compiler development, software engineering, and optimization.

# How to get started according to ChatGPT?

According to ChatGPT:

To get started with the LLVM compiler framework, follow these steps:

1. Familiarize yourself with the basics of compilers and how they work.

1. Read the LLVM documentation: The LLVM documentation is a great resource for learning about the framework, its features, and how to use them.

1. Set up a development environment: You will need to have a development environment set up with the LLVM compiler framework installed. There are many tutorials available online to help you get started with the setup process.

1. Practice writing simple programs: Start by writing simple programs and experimenting with the LLVM compiler framework to get a feel for how it works.

1. Participate in online communities: Join online forums or communities dedicated to LLVM to ask questions, learn from others, and stay up-to-date with the latest developments.

1. Read LLVM source code: Reading LLVM source code is a great way to understand how the framework works and learn best practices for using it.

1. Get involved in development: Consider contributing to the LLVM project by fixing bugs or adding new features. This is a great way to get hands-on experience with the framework and make a meaningful contribution to the community.

# A concrete way to get started

Well, that sounds abstract. Let us be more concrete. LLVM has an excellent [tutorial](https://llvm.org/docs/tutorial/MyFirstLanguageFrontend/index.html). However, there are challenges to get started:

1. The LLVM tutorial started by [guiding us to build LLVM from source](https://llvm.org/docs/GettingStarted.html). It takes hours and LLVM is very demanding of the host C++ compiler, and as such tends to expose bugs in the compiler.

1. The LLVM tutorial quickly jumps to more advanced compiler concepts, such [Static Single Assignment(SSA)](https://llvm.org/docs/tutorial/MyFirstLanguageFrontend/LangImpl03.html). A [blog](https://lowlevelbits.org/how-to-learn-compilers-llvm-edition/) recommended a large number of materials to learn before even starting.

So the objective of this blog series is to provide some concrete steps to start with LLVM. We will solve the 2 issues above with:

1. Instead of building LLVM from the source, we will download prebuilt binaries as well as how to use these binaries.

1. I will try to explain more advanced compiler concepts, assuming you already have some basic ideas about a compiler, such as lexer, parser, syntactic and semantic analysis, optimization and code-generation.

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
