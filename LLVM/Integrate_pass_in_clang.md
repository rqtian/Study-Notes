# Abstract
This note is highly based on article [Clang Integration](https://medium.com/@mshockwave/writing-llvm-pass-in-2018-part-iv-d69dac57171d). The common method to invoke a pass by clang is `clang -Xclang -load -Xclang mypass.so [other options]`. However, it is not convenient enough that every time we invoke the pass by loading the `.so` library of the pass. In this note, I will introduce how to invoke the pass by **passing one command line option to clang**. 

# Goal
Make clang support this: `clang -enable-mynewtry [other options]`

# Definition
Suppose the pass name is: `MyNewTry`. Suppose it is a transformation pass and it is a ModulePass.

The source code directory is: `llvm`.

# Steps
## 1. Add new option `-enable-mynewtry` in to `driver`:
Update file `clang/include/clang/Driver/OPtions.td` by adding the following code:
```
```
## 2. Connect `Driver` and `Clang` by translating driver option into clang option:
Update file `clang/lib/Driver/ToolChains/clang.cpp` by adding the following code:
```
```
## 3. Add new CodeGen options. 
### 3.1 Define the corresponding variable in the program for the option we are going to add.
Update file `clang/include/clang/Basic/CodeGenOptions.def` by adding the following code:
```
```
### 3.2 Translate program variable, and assign value to the program variable.
Update file `clang/libFrontend/CompilerInvocation.cpp` by adding the following code:
```
```
### 3.3 According to option variable, assign value to the passManager variable.
Update file `clang/lib/CodeGen/BackendUtil.cpp` by adding the following code:

## 4. Add LLVM pass in PassManagerBuilder.
Update file `llvm/lib/Transforms/IPO/PassManagerBuilder.cpp` by adding the following code:
```
```
Update file `include/llvm/Transforms/IPO/PassManagerBuilder.h` by adding the following code:
```
```

# Done 
