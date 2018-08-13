# Recursion Base Case

>This section assumes general knowledge & experience with recursion

In recursion, there are two kinds of control paths: recursive cases & base cases

The base case is one in which a "raw" value is returned (e.g. `return 1;`)

The recursive case is where the value returned is the return value of a call to the recursive method (e.g. `return q_sort(left) + q_sort(right);` inside of the function `q_sort()`)

## Recursion Without a Base Case

If recursion does not hit a base case, then it will recurse forever, realistically until a stack overflow occurs.

Any situation in which this happens is considered a bug. 

Note that even if some input reaches a base case, but not all input does (e.g. negative input to fibonacci sequence), this is still a bug.

## Preventing Infinite Recursion in Dep

In Dep, all input to a recursive funciton must be proven to reach a base case.

This includes input to the first invokation, as well as input given in the recursive case.

## `q_sort` Example

I know, I know, we keep using quicksort, sorry. It's just a great example of how Dep can help you!

In quicksort, we must split the input into smaller vectors to be recursively sorted. However, it's actually very simple to accidentally create a quicksort implementation that could infinitely recurse, but only under certain conditions that would make it hard to find and reproduce.

The case I am describing, is if the pivot is always the smallest element. We normally prevent this by picking the median of a couple elements, but it is conceivable to accidentally implement it where the pivot is just a randomly chosen element.

In this case, then everything other than the pivot is put into the right vector, since they are bigger, but because the pivot is equal to the pivot, it also goes into the right.

The result is that all of the input goes into one vector, and we haven't made it any smaller!

```dep
fn bad_q_sort(Vec src) -> Vec : Sorted + Reorder(src) {
  if (src.len < 2) return src;
  int pivot = src.getRandElement(); // This may pick the smallest element
  for (int curr in src) {
    if (curr < pivot)
      left.push(curr);
    else // curr >= pivot case, which always happens if pivot is smallest
      right.push(curr);
  }
  left = bad_q_sort(left);
  right = bad_q_sort(right);
  return left.push(right);
}
```

In this example, the pivot is just a randomly chosen element, meaning that it could theoretically always be the smallest element.

You are probably thinking, "eventually an element other than the smallest has to be chosen". Yes, this is true, however, it could pick the smallest element enough times to cause a stack overflow.

More importantly, even if the smallest element is only picked 1% of the time, then 1% of the calls are useless because nothing was done to the input!

### What the Compiler Does

If this bad example code where to be compiled in Dep, the compiler would see that the base case requires the input's length to be less than 2.

The compiler checks if the length can be inferred to always be two at this point, and sees that it cannot.

This means that in the recursive case, the input to the recursive call must make the input closer to meeting the base case. 

If the recursive case does not always put the input on a path that will eventually lead to base case being met, it should not be compiled

### Aside: Difficulties in Indirect Recursive Cases

The above requirements of the compiler are difficult to meet because some recursive cases can indirectly cause the base case to be met.

```dep
fn hills(int src) {
  if (src < 0)
    return;
  if (src < 100)
    return hills(src + 1);
  else
    return hills(-1);
}
```

In the above example, one of the recursive cases clearly moves the input toward meeting the base case, but the other seemingly moves it in the opposite direction.

However, the recursive case where `src` is increased actually leads inevitably to the base case being met.

