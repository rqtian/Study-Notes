# Abstract
This note is highly based on article [In-Tree Pass Integration](https://medium.com/@mshockwave/writing-llvm-pass-in-2018-part-iii-d44cd0c2c354). The basic steps of writing a pass can be easily reached from the official [Writing an LLVM Pass](http://llvm.org/docs/WritingAnLLVMPass.html). However, it only introduces how to run the pass with opt. In this note, I will introduce how to integrate the pass in PassManagerBuilder pipeline. With this, clang can automatically call the pass.

# Definition
Suppose the pass name is: `MyNewTry`. Suppose it is a transformation pass and it is a ModulePass.

The source code directory is: `llvm`.

# Steps
## 1. Define the pass in path:       
The `.cpp` file should be in `llvm/lib/transforms/scalar/MyNewTry.cpp`.

The corresponding `.h` file should be put in `llvm/include/llvm/transforms/Scalar/MyNewTry.h`

## 2. createXXXX function.
   
Add the following code in `llvm/include/llvm/transforms/Scalar/MyNewTry.h`,  

```      
namespace llvm {
  ModulePass *createMyNewTry();
}
```
      
Add the following code in `llvm/lib/transforms/scalar/MyNewTry.cpp`,

```
ModulePass *llvm::createMyNewTry() { return new MyNewTry(); }
```  
## 3. initializeXXXXPass function in InitializedPasses.h file.
Add the following code in `llvm/include/llvm/InitializePasses.h`, within `namespace llvm{ ... }`

```
void initializeMyNewTryPass(PassRegistry&);
```
## 4. INITIALIZE_PASS_BEGIN/END/DEPENDENCY code.
Add the following code in the source code file of the pass, i.e. `llvm/lib/transforms/scalar/MyNewTry.cpp`

```
INITIALIZE_PASS_BEGIN(MyNewTry, "my-new-pass begins here", "Some description for the Pass", false, false)
INITIALIZE_PASS_DEPENDENCY(LoopInfoWrapperPass) // Or whatever your Pass dependencies
INITIALIZE_PASS_END(MyNewTry, "my-new-pass end here", "Some description for the Pass", false, false)
```
## 5. Put your initializeXXXXPass in the right place.
### 5.1 In the source code file of the pass: `llvm/lib/transforms/scalar/MyNewTry.cpp`
Add the following code in the constructor of the pass in the pass's source code file. 

```
initializeMyNewTryPass(*PassRegistry::getPassRegistry());
```
      
So the new constructor of the pass should be:
```
MyNewTry() : ModulePass(ID) {
  initializeMyNewTryPass(*PassRegistry::getPassRegistry());
}
```
### 5.2 

In `llvm/lib/Analysis/Analysis.cpp` if the pass is an analysis pass and put in `llvm/lib/Analysis/` directory.

In `llvm/lib/Trnasforms/Scalar/Scalar.cpp` if the pass is a transformation pass and put in `llvm/lib/Trnasforms/Scalar/` directory.
        
Update the `llvm::initializeAnalysis` function in the above file with adding the following code:

```
void llvm::initializeAnalysis(PassRegistry &Registry) {
  //...Other initialization function calls
  initializeMyNewTryPass(Registry);
  //...
}
```
  
## 6. About createXXXX function
### 6.1 
Update the `ForcePassLinking()` function in `llvm/include/llvm/LinkAllPasses.h` by adding the code as follows:

```
struct ForcePassLinking {
  ForcePassLinking() {
    //...Other function calls
    (void) llvm::createMyNewTry();
    //...
  }
}
```
### 6.2
Add the following code in "llvm/include/llvm/Transforms/Scalar.h"

```
ModulePass * createMyNewTry();
```
## 7. Put your createXXXX in the right place.
Add the following code in `llvm/lib/Transforms/IPO/PassManageBuilder.cpp`,

```
void PassManagerBuilder::populateModulePassManager(
  legacy::PassManagerBase &MPM) {
          
  ...
  MPM.add(createMyNewTry());
  ...
```
If you want to invoke the pass in LTO, you can also add the following code in `llvm/lib/Transforms/IPO/PassManageBuilder.cpp`, 
 
```
void PassManagerBuilder::populateLTOPassManager(legacy::PassManagerBase &PM) {
  ...
  PM.add(createMyNewTry());
  ...
```
# Done
