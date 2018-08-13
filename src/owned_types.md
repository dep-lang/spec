# Owned Types

However, we would like introduce the concept of ownership similar to in Rust. This provides a benefit not only in the efficiency of your code, but also in its structure.

I'm imagining something like 

```dep
Vec own my_vec = Vec.own();
```

Where the `Vec.own()` method returns a new owned reference to a `Vec`. You could also get a reference like

```dep
Vec own my_vec = Vec.own();
Vec ref borrow = ref my_vec;
```

I also had another idea where there is a shortened syntax for both of these

```dep
Vec$ owned_vec = Vec.own();
Vec& borrow = &owned_vec;
```

## Deallocation of Memory

When an owned type reaches the end of its scope, its contents are deallocated, as in RAII.