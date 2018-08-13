# Function Contracts

You can specify a [precondition](https://en.wikipedia.org/wiki/Precondition) and [postcondition](https://en.wikipedia.org/wiki/Postcondition) in the method signature itself! Start with this function:

```dep
fn sum(int left, int right) -> int {
  return left + right;
}
```

We can define what about the result will always be true (AKA the postcondition). This is just a change to the method signature by adding a "property" to the return value. See that `: Sum(a,b)` after the return type? In this case, `Sum(a,b)` is a built-in property that means that this value (the one being returned) must be equivalent to the sum of its two arguments 

```dep
fn sum(int a, int b) -> int : Sum(a,b) {
//                          ^^^^^^^^^^
  return a + b;
}
```