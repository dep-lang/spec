# Property Debugging with the Hash Symbol (`#`)

You've heard of `println` debugging, where you print out values at certain points in the code to test your understanding of what the code is doing.

However, in Dep because properties must work together, it's also important to test your understanding of the compile-time guarantees that the compiler can infer. By putting a property after the pound symbol, the compile will tell you whether that property holds. This is essentially compile-time assertions

```dep
int i = 5;
# i > 0
int my_rand = rand(100) * 2;
# my_rand : Even
```

The compiler will show that all of these can be proven true at compile-time.

## On Compiler & Hash Assert Interaction

Although hash asserts will never change your code, they *can* help the compiler. It is non-trivial for the compiler to infer the needed properties to prove your code correct, and you hash asserts can serve as sort of a hint for what properties the compiler should be keeping track of...

Or not, I'm still getting to that part of the compiler. It defintely helps the humans who read your code, though :) 
