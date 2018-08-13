
# Memory Management

This section is very volatile, and not fully decided. However, I'd like to present my tentative idea

## Non-Nullable & Nullable Types

By default, when you declare a variable with a non-primitive type, like `String` as shown here...

```dep
String s = "Hello";
Vec v = Vec.new();
```

In this situation, the Type identifier on the far left was just the type name. This means that you have defined a non-nullable reference, like in Swift.

```dep
//String s = null;
//BAD: s is non-nullable
```

Also as in Swift, null-able data types can be declared using a `?` after the type name

```dep
String? s = null;
//OK!
```

This means that if your function might return null, the return type must be nullable

```dep
fn mightBeNull() -> String? {
  if (rand(100) == 0)
    null;
  else
    "roll again...";
}
```

And if you have a non-nullable data type, it cannot take nullable data

```dep
String s = mightBeNull();
//BAD: mightBeNull returns a nullable String
```

Here we can use the reduced ternary operator (`?:`) AKA the Elvis operator to return early if this `mightBeNull()` is `null`

```dep
String s = mightBeNull() ?: return;
```

## Garbage Collection

The nullable and non-nullable references are garbage collected. Whether by ARC or tracing is not specified by this document.