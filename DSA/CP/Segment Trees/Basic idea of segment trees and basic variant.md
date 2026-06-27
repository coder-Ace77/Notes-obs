
---

Segment trees are fully balanced full binary trees. It is known that binary trees can be implemented using either pointers or by using the arrays.

The binary trees implementation using arrays for complete trees is higly space optimised and great for cache locality. This is the key reason most of the segment trees implementations use the global array. More importantly there are two ways by which we can establish the parent child relationship. 

`0 based - root at index = 0` - In here childs are present at - `2*i+1` and `2*i+2` and parent at `(i-1)/2`

`1 based - root at index = 1` - Here childs are present at `2*i` and `2*i+1` respectively. While parent at `i/2`.

We will be using `0` based indexing moving forward.

A segment tree's leaves are composed entirely of the array values. While the value held by any non leaf is the `f(val(left_child),val(right_child))`. Each node in the segment tree has a notion of a value. For the leaf nodes values are mostly but not always value of the array. But for the non leaf node its calculated from its childs. 

This means the nodes always have the values present as the power of `2` of the array. Now how to 