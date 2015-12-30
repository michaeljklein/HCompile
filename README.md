This is a project to implement a small, lightweight JIT function in Haskell, of the form 'compile :: (a -> b -> c) -> a -> IO (b -> c)', that takes a function of two arguments and a first argument to return a version of the function compiled with that first argument as a constant.

For example, suppose we have the function: 'f a b = if a then b*b else b'. Then 'compile f True' would equal "f' b = b*b", where f' is optimized for the specific input.

One application is in polynomial interpolation, in which calling the interpolated polynomial may require traversing some arbitrary size data structure to compute the output of the polynomial. If this new function is to be called many times, it would be faster to know the constants at compile time and optimize for them. 

The planned workflow has the following steps:

1. At compile time, collect the source for the functions to be compiled at runtime
2. Compile the functions individually to LLVM IR (or bitcode)
3. Optimize the generated code
4. Convert the optimized LLVM code to a template, which can be easily filled in to include the desired constants
5. Include the template in the compiled program
6. When 'compile f a' is called, convert 'a' to LLVM IR and place in the template
7. Reoptimize the filled template, using only options that utilize constant propogation
8. 'compile f a' returns a FFI ccall function to the compiled template.

I have considered trying more of these steps during run time, however there are a few problems:
* Processing to LLVM at runtime has a one-time overhead when each function is called for the first time
* Either a LLVM disassembler (not lightweight at all) or the source for the involved functions would have to be included in the program binary. At that point, you'd have to preprocess the code at compile time enough that you might as well do as much as possible at compile time

I originally considered using C instead of LLVM, as it seems much easier to code the interfaces, etc., but C is best optimized after being converted to assembly/LLVM IR and so I'd lose most of the ability to optimize at compile time, damaging the utility of the project.

The prototype method requires the function to be compiled to be in its own module. I'm playing around with ways to detect which functions are passed to 'compile' at compile time, but the 'best' way likely would require messing with GHC internals. So the two most viable option so far is to provide a list of functions to be called, within the program.

At the moment, I have been having problems in OSX and Ubuntu with GHC emitting LLVM. While the code is (more or less) readable, there are plenty of errors for trivial functions and I'm mainly trying to work out how to isolate/detect the function arguements and Haskell interface fluff.
