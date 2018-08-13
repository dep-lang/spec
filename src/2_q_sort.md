# Quicksort Example

Let's write quicksort. Just by compiling in Dep, it will be proven that this function is a correct implementation of a sorting function. We do this by creating a contract in the function

```dep
fn q_sort(Vec<int> src) -> Vec<int> : Sorted + Reorder(src) { ... }
```

Let's break down just the method signature.

First, it takes in a vector (resizing array) of ints and returns a vector of ints.

Next, after the return type we see the colon, signaling a list of properties about the return.

There are two properties of the return type, `Sorted` and `Reorder(in)`


## Quicksort Implementation

In case you don't know how quicksort works or forgot, here's a quick refresher. Note this is not the in-place quicksort, so it will allocate more memory. *However*, in-place quicksort is entirely possible and proveable in Dep, we may revisit it later to show properties that work on systems-style applications

The pseudocode for Quicksort goes like
```python
def quicksort(src):
  # Base case: one or zero element array is already sorted
  if (src.len < 2):
    return src
  pivot = median of several random elements in src
  left = new array
  right = new array
  # Split the input into values less than pivot and greater than pivot
  for curr in src:
    if (curr < pivot):
      left.push(curr)
    else:
      right.push(curr)
  # Sort each half recursively
  left = quicksort(left)
  right = quicksort(right)
  # Since left & right are sorted, put right on the end of left
  return left.push(right)
```

The Dep version will look like this:

```dep
fn q_sort(Vec src) -> Vec : Sorted + Reorder(src) {
  if (src.len < 2)
    return src;
  int pivot = src.getRandMedian();
  Vec mut left = Vec.new();
  Vec mut right = Vec.new();
  for (int curr in src) {
    if (curr < pivot) 
      left.push(curr);
    else 
      right.push(curr);
  }
  left = q_sort(left);
  right = q_sort(right);
  return left.push(right);
}
```

The great thing is that this looks almost exactly like our pseudocode! The only difference was the properties of the return value which we discussed earlier. Despite the fact that we just translated our pseudocode, this is all proven correct at compile-time!

This is all thanks to the property definitions of the standard library which accounts for `Vec` and `Vec.push` as well as several other implicit properties which the compiler can compose together to get our return value's properties

In the next section we will talk about some of these

