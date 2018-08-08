
# Properties in Dep

## Zero-Cost Abstraction

In Dep, you can guarantee the correctness of your code without ever sacrificing speed. Just like how the type system in C/C++ has effect on speed, properties in Dep are only checked at compile-time. 

This is the core principle of Dep:

>Prove your programs correct with zero-cost abstractions

## Function Contracts

You can specify a [precondition](https://en.wikipedia.org/wiki/Precondition) and [postcondition](https://en.wikipedia.org/wiki/Postcondition) in the method signature itself! Start with this function:

```dep
fn sum(i32 left, i32 right) -> i32 {
  return left + right;
}
```

We can define what about the result will always be true (AKA the postcondition). This is just a change to the method signature by adding a "property" to the return value. See that `: Sum(a,b)` after the return type? In this case, `Sum(a,b)` is a built-in property that means that this value (the one being returned) must be equivalent to the sum of its two arguments 

```dep
fn sum(i32 a, i32 b) -> i32 : Sum(a,b) {
//                          ^^^^^^^^^^
  return a + b;
}
```

## The `:` (Colon) in Dep

In Dep, the colon has one, and only one, meaning: 

```dep
The properties after the colon describe the variable before the colon
```

## Making Our Own Properties

That's trivial, let's try something a little more interesting. Scaling by a random number > 1

```dep
fn scaleUpRand(i32 in) -> i32 {
  return in * rand(2, 100);
}
```

Let's say we want to know that our number increased as a result of this operation

```dep
prop Bigger(i32 self, i32 other) {
  self > other
}
```

Now we can embed this property into our method signature like before

```dep
fn scaleUpRand(i32 in) -> i32 : Bigger(in) {
  return in * rand(2, 100);
}
```

Note, the compiler can recognize that because the minimum value that `rand(2,100)` returns is 2, then then minimum value returned is `in*2`. Because `in*2 > in`, it can guarantee at compile time that the return value is bigger than the input!

You may also be saying, "Wait, I thought Bigger takes two arguments, you only gave one...". When we defined `Bigger` the first argument was named `self` which is a special keyword making this property like a method, it infers the first argument is the value it is being defined for (e.g. the return value)

## Using Properties on Local Variables

Properties can also be applied to normal variable decalarations

```dep
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

```dep
int i : Bigger(5) + Not(10) = 7;
```

We use the `+` symbol here instead of something like `,` because it would make parsing difficult in some situations. For example, using generics.

## Property Debugging with the Hash Symbol (`#`)

You've heard of `println` debugging, where you print out values at certain points in the code to test your understanding of what the code is doing.

However, in Dep because properties must work together, it's also important to test your understanding of the compile-time guarantees that the compiler can infer. By putting a property after the pound symbol, the compile will tell you whether that property holds. This is essentially compile-time assertions

```dep
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

```dep
fn q_sort(Vec<int> src) -> Vec<int> : Sorted + Reorder(src) { ... }
```

Let's break down just the method signature.

First, it takes in a vector (resizing array) of ints and returns a vector of ints.

Next, after the return type we see the colon, signaling a list of properties about the return.

There are two properties of the return type, `Sorted` and `Reorder(in)`

## `Sorted` Property

The Sorted property's definition looks like so

```dep
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
```python
def quicksort(src):
  # Base case: one or zero element array is already sorted
  if (src.len < 2):
    return src
  pivot = random element from src
  left = new array
  right = new array
  # Split the input into values less than pivot and greater than pivot
  for curr in src:
    if (curr < pivot):
      left.push(curr)
    else:
      right.push(curr)
  # Sort each half recursively
  left = quicksort(left)
  right = quicksort(right)
  # Since left & right are sorted, put right on the end of left
  return left.push(right)
```

The Dep version will look like this:

```dep
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

```dep
fn Lt(Vec self, int ceil) {
  forall e in self: e < ceil
}
```

```dep
fn Geq(Vec self, int floor) {
  forall e in self: e >= floor
}
```

These properties can be applied to prove that all the elements are less than or greater than/equal to a certain value. In this case, we are concerned with the pivot.

In the for loop we can verify these properties as the loop invariant

```dep
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
