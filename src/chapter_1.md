Temporary (Pre)Dep Language Specification

This will definitely move, at the very least to a markdown file in the source code.

Right now this is just going to be mostly example code to get this out here

# Purpose

Dep has one clear goal: if it compiles, it has been proven correct! 

The author adds "properties" to methods and classes that specify preconditions, postconditions, and representation invariants. With this information Dep can prove at compile time that all of your code meets these specifications.

# Hello World

```rust
fn main() {
  println!("Hello, World!");
}
```

 * `fn` starts a function
 * Curly braces around the function body (C/Java style)
 * The `println!()` function is a macro, which is why it ends with a `!`

# Basic Syntax

## Variables

In Dep, declare a new variable like so

```rust
int num = 401;
```

It contains:
 * Type 
 * Name 
 * Initial value (optional)

Dep has the usual primitive types `int`, `long`, `float`, `double`

Primitives all lowercase, but all user defined types including standard library types like `String` and `Vec` must use UpperCase

## Mutability

Variables are immutable by default in Dep. To make them mutable, add the `mut` keyword

```rust
Vec my_vec = Vec.new();
//my_vec.push(5);
//BAD: my_vec is immutable, push requires a mutable Vec
Vec mut my_mut_vec = Vec.new();
my_mut_vec.push(5);
//OK!
```

## Functions

```rust
fn sum(int i, int j) -> int {
  return i + j;
}
```

Return type is denoted after the `->` following the function signature. If no return type is specified, this means the function returns nothing.

# Properties in Dep

## Zero-Cost Abstraction

In Dep, you can guarantee the correctness of your code without ever sacrificing speed. Just like how the type system in C/C++ has effect on speed, properties in Dep are only checked at compile-time. 

This is the core principle of Dep:

>Prove your programs correct with zero-cost abstractions

## Function Contracts

You can specify a [precondition](https://en.wikipedia.org/wiki/Precondition) and [postcondition](https://en.wikipedia.org/wiki/Postcondition) in the method signature itself! Start with this function:

```rust
fn sum(i32 left, i32 right) -> i32 {
  return left + right;
}
```

We can define what about the result will always be true (AKA the postcondition). This is just a change to the method signature by adding a "property" to the return value. See that `: Sum(a,b)` after the return type? In this case, `Sum(a,b)` is a built-in property that means that this value (the one being returned) must be equivalent to the sum of its two arguments 

```rust
fn sum(i32 a, i32 b) -> i32 : Sum(a,b) {
//                          ^^^^^^^^^^
  return a + b;
}
```

## The `:` (Colon) in Dep

In Dep, the colon has one, and only one, meaning: 

```quote
The properties after the colon describe the variable before the colon
```

## Making Our Own Properties

That's trivial, let's try something a little more interesting. Scaling by a random number > 1

```rust
fn scaleUpRand(i32 in) -> i32 {
  return in * rand(2, 100);
}
```

Let's say we want to know that our number increased as a result of this operation

```rust
prop Bigger(i32 self, i32 other) {
  self > other
}
```

Now we can embed this property into our method signature like before

```rust
fn scaleUpRand(i32 in) -> i32 : Bigger(in) {
  return in * rand(2, 100);
}
```

Note, the compiler can recognize that because the minimum value that `rand(2,100)` returns is 2, then then minimum value returned is `in*2`. Because `in*2 > in`, it can guarantee at compile time that the return value is bigger than the input!

You may also be saying, "Wait, I thought Bigger takes two arguments, you only gave one...". When we defined `Bigger` the first argument was named `self` which is a special keyword making this property like a method, it infers the first argument is the value it is being defined for (e.g. the return value)

## Using Properties on Local Variables

Properties can also be applied to normal variable decalarations

```rust
int mut i = 5;
int mut j : Bigger(i) = 10;
//^^ OK
j = 15;
//^^ Still OK
//j = 0;
//^^ Causes compile time error, j defies type property Bigger(i)
```

Note that the property doesn't just have to hold true initially, it must always hold true. If the compiler cannot guarantee that all properties are true, then it will not compile.

## Multiple Properties

Variables can have as many properties as you want. When defining multiple properties, put a `+` between the properties

```rust
int i : Bigger(5) + Not(10) = 7;
```

We use the `+` symbol here instead of something like `,` because it would make parsing difficult in some situations. For example, using generics.

## Property Debugging with the Hash Symbol (`#`)

You've heard of `println` debugging, where you print out values at certain points in the code to test your understanding of what the code is doing.

However, in Dep because properties must work together, it's also important to test your understanding of the compile-time guarantees that the compiler can infer. By putting a property after the pound symbol, the compile will tell you whether that property holds. This is essentially compile-time assertions

```rust
int i = 5;
# i > 0
int my_rand = rand(100) * 2;
# my_rand : Even
```

The compiler will show that all of these can be proven true at compile-time.

### On Compiler & Hash Assert Interaction

Although hash asserts will never change your code, they *can* help the compiler. It is non-trivial for the compiler to infer the needed properties to prove your code correct, and you hash asserts can serve as sort of a hint for what properties the compiler should be keeping track of...

Or not, I'm still getting to that part of the compiler. It defintely helps the humans who read your code, though :) 

# The Power of Contracts

Let's write quicksort. Just by compiling in Dep, it will be proven that this function is a correct implementation of a sorting function. We do this by creating a contract in the function

```rust
fn q_sort(Vec<int> src) -> Vec<int> : Sorted + Reorder(src) { ... }
```

Let's break down just the method signature.

First, it takes in a vector (resizing array) of ints and returns a vector of ints.

Next, after the return type we see the colon, signaling a list of properties about the return.

There are two properties of the return type, `Sorted` and `Reorder(in)`

## `Sorted` Property

The Sorted property's definition looks like so

```rust
prop Sorted(Vec<int> self) {
  forall (e,f) in self.AdjPair(): e < f
}
```

The way we can read this is "Sorted is a property of Vec<int>, where for every e & f adjacent to each other, e is less than f"

You may be saying, "hold on, you said that no runtime code is executed by properties, but this one has a for loop!" You are correct that there is a for loop, however, there are two key differences.

The loop is using `forall` which is proving an expression (`e<f`) holds for every `(e,f)` in this Vec. It never actually generates any code, the compiler simply verifies that, in this case, any adjacent pair will be sorted.

Similar idea, The function returning an iterator in this case is `self.AdjPair()`. Notice the UpperCase on the method. This means that the method is also a property! These "property functions" also never generate any code, `AdjPair()` simply returns an expression describing every adjacent pair in this Vec. No iterator is generated at runtime.

## `Reorder(Vec)` Property

The other property of the return value is `Reorder(src)`. What this means is that all of the elements in `src` are also in the output, but not necessarily in the same order.

This is important because we need to know that the function doesn't discard any values in the original input.

## Quicksort Implementation

In case you don't know how quicksort works or forgot, here's a quick refresher. Note this is not the in-place quicksort, so it will allocate more memory. *However*, in-place quicksort is entirely possible and proveable in Dep, we may revisit it later to show properties that work on systems-style applications

The pseudocode for Quicksort goes like
```
fn quicksort(src) -> out {
  # Base case: one or zero element array is already sorted
  if (src.len < 2) 
    return src
  pivot = random element from src
  left = new array
  right = new array
  # Split the input into values less than pivot and greater than pivot
  foreach curr in src {
    if (curr < pivot)
      left.push(curr)
    else
      right.push(curr)
  }
  # Sort each half recursively
  left = quicksort(left)
  right = quicksort(right)
  # Since left & right are sorted, put right on the end of left
  return left.push(right)
}
```

The Dep version will look like this:

```rust
fn q_sort(Vec src) -> Vec : Sorted + Reorder(src) {
  if (src.len < 2)
    return src;
  int pivot = src.getRand();
  Vec left = Vec.new();
  Vec right = Vec.new();
  for (int curr in src) {
    if (curr < pivot) 
      left.push(curr);
    else 
      right.push(curr);
  }
  left = q_sort(left);
  right = q_sort(right);
  return left.push(right);
}
```

The great thing is that this looks almost exactly like our pseudocode! The only difference was the properties of the return value which we discussed earlier. Despite the fact that we just translated our pseudocode, this is all proven correct at compile-time!

This is all thanks to the property definitions of the standard library which accounts for `Vec` and `Vec.push` as well as several other implicit properties which the compiler can compose together to get our return value's properties

In the next section we will talk about some of these

# `Lt()` and `Geq()`

These properties stand for "less than" and "greater than or equal to" respectively. The compiler automatically infers these properties to verify the postcondition.

```rust
fn Lt(Vec self, int ceil) {
  forall e in self: e < ceil
}
```

```rust
fn Geq(Vec self, int floor) {
  forall e in self: e >= floor
}
```

These properties can be applied to prove that all the elements are less than or greater than/equal to a certain value. In this case, we are concerned with the pivot.

In the for loop we can verify these properties as the loop invariant

```rust
for (int curr in src) { # left : Lt(pivot) && right : Geq(pivot)
    if (curr < pivot) 
        left.push(curr);
    else 
        right.push(curr);
}
```

If we add these compile-time asserts to our code it will still compile because the compiler can infer that at any iteration of this loop, these two properties will hold

For example, we know that at the start of the first iteration of the loop, left will be empty, meaning that all of the elements in it are less than the pivot (since there are none). 

It gets more interesting when we realize that the compiler has infered that because `left` is only modified in one branch of the if statement, it can make stronger assumptions about how `left` is being modified. And indeed, `left` will only ever be modified by pushing elements less than the pivot onto the end of it.

Applying the same logic we can see how `Geq()` was applied

## Property Functions

## `AdjPair()`

In the quicksort example, we used the property `Sorted` whose definition includes this operation on a  `Vec`

```rust 
self.AdjPair()
```

This is a property function, like properties it generates no run-time code and uses UpperCase.

`AdjPair` describes every adjacent pair in a vector, let's look at its definition

```rust
prop fn AdjPair(Vec self) -> prop Iter<(int, int)> {
//                           ^^^^ not necessary, but nice if you want to be explicit
  forall i in 0..self.len-2: (self.arr[i], self.arr[i+1])
}
```

This property function returns a "property iterator" over the adjacent pairs in `self`. The `prop` keyword in the return type indicates that this value will never be created at run-time. `prop` values can only be returned by `prop` functions, and because of this, the `prop` in the return type is optional, but we use it to differentiate between a real, run-time iterator and a property iterator that simply represents an expression to the compiler.

Again, we use the `forall` syntax to make an expression, this time representing values instead of just boolean expressions. Specifically, we are creating a set of tuples, where each one represents an adjacent pair in the `self` internal array.

## Property Types

The `prop Iter` from the last section is one of actually only a handful of these property types. Because property types are only ever needed at compile-time they work closely with the language and cannot be extended by the user.

## `Reorder(Vec)`

```rust
prop Reorder(Vec self, Vec src) {
  forall e in self: e in src,
  forall e in src: e in self
}
```

# The `in` keyword

In Dep, the `in` keyword can be used both in runtime `for` loops and compile-time `forall` expressions

They both call functions which can be overridden. `iter()` for `for` loops and `propIter()` for `forall` expressions

```rust
fn iter(Vec self) -> Iter<int>;
```

```rust
prop fn propIter(Vec self) -> prop Iter<int> {
  forall i in self.len-1: self.arr[i]
}
```

# Memory Management

This section is very volatile, and not fully decided. However, I'd like to present my tentative idea

## Non-Nullable & Nullable Types

By default, when you declare a variable with a non-primitive type, like `String` as shown here...

```rust
String s = "Hello";
Vec v = Vec.new();
```

In this situation, the Type identifier on the far left was just the type name. This means that you have defined a non-nullable reference, like in Swift.

```rust
//String s = null;
//BAD: s is non-nullable
```

Also as in Swift, null-able data types can be declared using a `?` after the type name

```rust
String? s = null;
//OK!
```

This means that if your function might return null, the return type must be nullable

```rust
fn mightBeNull() -> String? {
  if (rand(100) == 0)
    null;
  else
    "roll again...";
}
```

And if you have a non-nullable data type, it cannot take nullable data

```rust
//String s = mightBeNull();
//BAD: mightBeNull returns a nullable String
```

Here we can use the reduced ternary operator (`?:`) AKA the Elvis operator to return early if this `mightBeNull()` is `null`

```rust
String s = mightBeNull() ?: return;
```

## Owned Types

However, we would like introduce the concept of ownership similar to in Rust. This provides a benefit not only in the efficiency of your code, but also in its structure.

I'm imagining something like 

```rust
Vec own my_vec = Vec.own();
```

Where the `Vec.own()` method returns a new owned reference to a `Vec`. You could also get a reference like

```rust
Vec own my_vec = Vec.own();
Vec ref borrow = ref my_vec;
```

I also had another idea where there is a shortened syntax for both of these

```rust
Vec$ owned_vec = Vec.own();
Vec& borrow = &owned_vec;
```

# Pre-Dep vs Dep

What's the whole deal with this Pre-Dep business? Basically, I'm not 100% set on the semantics and syntax of the language, and some things are so in-flux that I'm not confident I can describe a "beta" form of the language and then just make small changes. More like I need to make an "alpha" of the language (Pre-Dep) , and then decide for real on some of the semantics which will get finalized and become the Dep language.

I chose the name Pre-Dep because in my opinion it's easier to describe the earlier state of a language using a different name than version number like Python 2 vs 3