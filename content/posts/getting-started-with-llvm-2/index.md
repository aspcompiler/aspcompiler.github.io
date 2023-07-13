---
title: "Getting Started With LLVM (2)"
date: 2023-07-12T23:28:48-07:00
tags: ["LLVM"]
categories: "LLVM"
draft: false
---

# Introduction

In the previous [installation](/posts/getting-started-with-llvm-1/), we provided a way to get started hands-on with LLVM with minimum effort. In this blog, we will follow the [chapter 3](https://llvm.org/docs/tutorial/MyFirstLanguageFrontend/LangImpl03.html) and [chapter 4](https://llvm.org/docs/tutorial/MyFirstLanguageFrontend/LangImpl04.html) of the LLVM tutorial to generate code.

Chapter 3 is about building the LLVM intermediate representation(IR) from the AST. Chapter 4 is about optimization passes as well as the just-in-time(JIT) compilation.

While the LLVM tutorials are very detailed, I will try to summarize the key points and provide some additional explanations.

Last note of caution is that the current stable distribution is LLVM 16 while the tutorial is already updated to the pre-release of LLVM 17. So if you have installed LLVM 16 binary rather than building from the source, you may encounter compatibility issues. If so, you should
use the [stable source](https://github.com/llvm/llvm-project/tree/release/16.x/llvm/examples/Kaleidoscope) instead.

# Building the LLVM IR

The key for building the LLVM is to instantiate the `TheContext`, `Builder` and `TheModule`. `TheContext` is an opaque object that owns a lot of core LLVM data structures, such as the type and constant value tables. The `Builder` object is a helper object that makes it easy to generate LLVM instructions. `TheModule` is an LLVM construct that contains functions and global variables.


```cpp
static std::unique_ptr<LLVMContext> TheContext;
static std::unique_ptr<IRBuilder<>> Builder(TheContext);
static std::unique_ptr<Module> TheModule;
```

Building the LLVM then follow several patterns:

1. Getting from Context for type and constants mentioned above.
```cpp
ConstantFP::get(*TheContext, APFloat(Val));
FunctionType *FT =
    FunctionType::get(Type::getDoubleTy(*TheContext), Doubles, false);
```
2. Constructing using the Builder.
```cpp
Builder->CreateFAdd(L, R, "addtmp");
Builder->CreateCall(CalleeF, ArgsV, "calltmp");
Builder->SetInsertPoint(BB);
Builder->CreateRet(RetVal);
```
3. Create functions and basic blocks:
```cpp
Function *F = Function::Create(FT, Function::ExternalLinkage, Name, TheModule.get());
BasicBlock *BB = BasicBlock::Create(*TheContext, "entry", F);
```

Lastly, we can print the IR using the `TheModule` object:
```cpp
TheModule->print(errs(), nullptr);
```

# Optimization Passes

The LLVM uses FunctionPassManager to manage the optimization passes. You can many built-in passes and create custom passes.
```cpp
// Create a new pass manager attached to it.
  TheFPM = std::make_unique<legacy::FunctionPassManager>(TheModule.get());

  // Do simple "peephole" optimizations and bit-twiddling optzns.
  TheFPM->add(createInstructionCombiningPass());
  // Reassociate expressions.
  TheFPM->add(createReassociatePass());
  // Eliminate Common SubExpressions.
  TheFPM->add(createGVNPass());
  // Simplify the control flow graph (deleting unreachable blocks, etc).
  TheFPM->add(createCFGSimplificationPass());

  TheFPM->doInitialization();

  // Generate the IR

  // Optimize the function.
  TheFPM->run(*TheFunction);
  ```

# JIT Compilation

To use the JIT compiler, it is necessary to initialize:

```cpp
  InitializeNativeTarget();
  InitializeNativeTargetAsmPrinter();
  InitializeNativeTargetAsmParser();
```

Then set the data layout in the module:
```cpp
  TheModule->setDataLayout(TheJIT->getTargetMachine().createDataLayout());
```

Then we can use the JIT compiler to compile the module:
```cpp
  auto H = TheJIT->addModule(std::move(TheModule));
```

Then generate the `TreadSafeModule` and add to the JIT.
```cpp
  auto TSM = llvm::orc::ThreadSafeModule(std::move(TheModule), std::move(Context));
  auto H = TheJIT->addModule(std::move(TSM));
```

Lastly, get the function pointer of the top-level function and call it:
```cpp
  auto ExprSymbol = TheJIT->lookup("__anon_expr");
  assert(ExprSymbol && "Function not found");
  double (*FP)() = (double (*)())(intptr_t)cantFail(ExprSymbol.getAddress());
  fprintf(stderr, "Evaluated to %f\n", FP());
```

