# The `:` (Colon) in Dep

In Dep, the colon has one, and only one[^1], meaning: 

```dep
The properties after the colon describe the variable before the colon
```

This extends to situations across Dep

* Variable definitions

* Return value and parameter bounds

* Loop invariants

* Function type bounds

[^1]: Okay, so the ternary operator also uses `:`, but that is the one exception because it's already widely used and concise