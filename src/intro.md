Dep Language Specification

# Purpose

Dep has one clear goal: if it compiles, it has been proven correct! 

Correct here means that it matches the specification written as using properties in Dep.

## Example

Here is a correct quicksort implementation in Dep. The return type guarantess that it is sorted, and is a reordering of the input vector. All of this can be inferred by the compiler without the author adding any special logic.

```dep
fn q_sort(Vec src) -> Vec : Sorted + Reorder(src) {
  if (src.len < 2)
    return src;
  int pivot = src.getRandMedian();
  # pivot != src.Smallest()
  Vec mut left = Vec.new();
  Vec mut right = Vec.new();
  for (int curr in src) {  # left : Lt(pivot) && right : Geq(pivot)
                          // ^^ serves as loop invariant, but is *not* necessary
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

Dep also prevents a host of other errors by turning them into compile time errors.

Accidentally sort the left list twice? Compiler error: output not a reorder, and not sorted.

Leave out the base case? Compiler error: infinite recursion.

Get a totally random pivot instead of a median of some kind? Compiler error: potentially infinite recursion.

# Selling Points

 * Encode a rigorous formal specification naturally into the source code
 
 * Imperitive, expression oriented
 
 * Choice between GC-references or owned-RAII types