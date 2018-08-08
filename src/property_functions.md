
# Property Functions

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
