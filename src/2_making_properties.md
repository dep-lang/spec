# Making Our Own Properties

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
