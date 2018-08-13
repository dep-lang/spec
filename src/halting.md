# Halting

## The Halting Problem

Those of you who are aware of the famous halting problem, will find it suspicious that there is a section on halting in Dep.

As a short, and by no means comprehensive introduction to the halting problem, it essentially proves the following.

>For any nontrivial runtime property of a program, there *cannot exist* a program that can always prove whether this holds true or not.

>The main example of this theorem, which acts as its namesake, is whether a program will halt or run in an infinite loop.

The explanation for how this theorem works is nontrivial, but it means that Dep cannot always tell whether a function will halt or not.

## The Catch

It *can* however, prove that some functions will always halt. 

This is very useful for proving the correctness of your programs because Hoare logic, which the Dep compiler uses to prove the programs correctness, actually only proves the so-called "partial correctness" of the program.

Meaning, that Dep can prove that your program meets your specification, assuming that it halts.

## The `fn halt` Keyword

When you declare a function with `fn halt` instead of the traditional `fn`, you are saying

>This function will always halt, and for it to compile, the compiler must be able to prove this is true.

This means that all of the function calls inside of `fn halt` functions must also be declared with `fn halt`

Additionally, any loops must be provably finite in iteration.

## Recursion

It would be quite unfortunate for you to write a recursive function which meets your specification, but that never halts because the base case was written incorrectly. 

This is precisely why Dep has built-in facilities to prevent infinite recursion, which we will explore in the next section.