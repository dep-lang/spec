# Mutability

Variables are immutable by default in Dep. To make them mutable, add the `mut` keyword

```dep
Vec my_vec = Vec.new();
//my_vec.push(5);
//BAD: my_vec is immutable, push requires a mutable Vec
Vec mut my_mut_vec = Vec.new();
my_mut_vec.push(5);
//OK!
```