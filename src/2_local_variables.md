# Using Properties on Local Variables

Properties can also be applied to normal variable decalarations

```dep
int mut i = 5;
int mut j : Bigger(i) = 10;
//^^ OK
j = 15;
//^^ Still OK
//j = 0;
//^^ Causes compile time error, j defies type property Bigger(i)
```

Note that the property doesn't just have to hold true initially, it must always hold true. If the compiler cannot guarantee that all properties are true, then it will not compile.

## Multiple Properties

Variables can have as many properties as you want. When defining multiple properties, put a `+` between the properties

```dep
int i : Bigger(5) + Not(10) = 7;
```

We use the `+` symbol here instead of something like `,` because it would make parsing difficult in some situations. For example, using generics.
