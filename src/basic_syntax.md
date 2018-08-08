
# Basic Syntax

## Variables

In Dep, declare a new variable like so

```dep
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
