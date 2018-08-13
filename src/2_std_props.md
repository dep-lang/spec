# Standard Library Properties

## `Sorted` Property

The Sorted property's definition looks like so

```dep
prop Sorted(Vec self) {
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

## `Lt()` and `Geq()`

These properties stand for "less than" and "greater than or equal to" respectively. The compiler automatically infers these properties to verify the postcondition.

```dep
prop Lt(Vec self, int ceil) {
  forall e in self: e < ceil
}
```

```dep
prop Geq(Vec self, int floor) {
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
