
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
