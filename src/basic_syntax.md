
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

### Type

Type represents what kind of data is in the variable.

Here, by defining `num` as an `int` it means that this number can only hold integer (whole number) values, without any decimal place.

Trying to assign a non-integer value to it would result in a compile-time error

```dep
int num = 0.5;
//ERR: int num cannot take float value
```

However, if we want to use a decimal point, we can employ the `float` type

```dep
float percentage = 0.99;
```

If you are not familiar with floating point numbers, the underlying representation of `float`, just be aware that on all computers, there are inaccuracies in computing. 

For example, `0.7` can only be approximated as `0.6999999881`. However, there are many situations where using `float` is necessary and there are measures to ensure an acceptable degree of correctness even in those situations.

#### Primitives

The "primitive" types in Dep, meaning the types which are built-in to the language are

 * `byte` 8-bit integer
 * `short` 16-bit integer
 * `int` 32-bit integer
 * `long` 64-bit integer

 * `float` 32-bit floating point
 * `double` 64-bit floating point
 
 * `bool` 8-bit space boolean value

Note that Dep primitive sizes are platform and architecture independent

The primitive type names are in all lowercase, but all user defined types including standard library types like `String`, `Vec`, and `TreeMap` must use UpperCase

## Mutability

Variables are immutable by default in Dep. To make them mutable, add the `mut` keyword

```dep
Vec my_vec = Vec.new();
//my_vec.push(5);
//BAD: my_vec is immutable, push requires a mutable Vec
Vec mut my_mut_vec = Vec.new();
my_mut_vec.push(5);
//OK!
```

## Functions

```dep
fn sum(int i, int j) -> int {
  return i + j;
}
```

Return type is denoted after the `->` following the function signature. If no return type is specified, this means the function returns nothing.
