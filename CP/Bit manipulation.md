
---

|Operator|Symbol|Meaning|Example|
|---|---|---|---|
|AND|`&`|1 if both bits are 1|`5 & 3 → 101 & 011 = 001 (1)`|
|OR|`\|`|1 if either bit is 1|`5 \| 3 → 101 \| 011 = 111 (7)`|
|XOR|`^`|1 if bits differ|`5 ^ 3 → 101 ^ 011 = 110 (6)`|
|NOT|`~`|Flips all bits|`~5` (depends on bit width)|
|Left Shift|`<<`|Shift bits left (multiply by 2)|`5 << 1 = 10`|
|Right Shift|`>>`|Shift bits right (divide by 2)|`5 >> 1 = 2`|
## Tricks 

A number with only one bit set if we substract 1 from it then all the bits after the one and only set bit will become 1. If we do bit wise `&` then It will be zero. 
Also additon of 1 to a number has a ripple effect where the number 1 will propogate uptil a `0` is reqached and will 
1. Power of 2 check - 

```cpp
bool isPowerOfTwo(int n) {
    return n && !(n & (n - 1));
}
```

Subtracting 1 flips all bits after the rightmost 1 (including it). So, if only one bit was set, `n & (n-1)` becomes 0.

2. Checking if nth bit is set `x & (1 << n)` or equivalently `(x >> n) & 1`. 
3. How to toggle the nth bit `x = (x ^ (1 << n))`
4. Set the nth bit `x = (x | (1 << n))` Equivalently we can unset the nth bit by `x = (x & ~(1 << n))`
5. Turning off the rightmost 1 `x = x & (x - 1)`
6. Isolate rightmost 1 `x = x & -x`
7. Highest set bit 

```cpp
int highestBit = 1 << (31 - __builtin_clz(n));
```

-x is equivalent to `~x+1`

8. Iterating over subsets

```cpp
for (int mask = 0; mask < (1 << n); mask++) {
    // mask represents a subset
}
```

Xor properties - 

`a^a=0 , a^0=a`. 
Swapping without temp variable

```cpp
a ^= b;
b ^= a;
a ^= b;
```

Sometimes we can use binary search in problems with xors. Used in problems like finding **maximum possible XOR** with constraints.  For example, you can check each bit from MSB to LSB and greedily decide if it can be set.

9. Bit wise numbers are also used in carrying the subsets in the dp called bitmask dp. Here the current subset is being carried over to the set of choices. 
10. Xor is associative meaning if x and y are two numbers and then `z` is their xor. then All these are valid

```
a^b=z
a=b^z
b=a^z
```

11. Sometimes we can solve the problem by considering the numbers as sum of its individual bits. It is specially useful if the problem asks to find the some of some function for each number. Then we can consider number as

`a = bn*(2**n)+bn-1*(2**(n-1))+bn-2*(2**(n-2))+...+b1*2+b0`.Now the problem simplfyfies for each bit independently. More over this is pretty powerful trick to solve problems where we can consider the issue to solve the problems by bits.  This technique can be used to solve the problem of sum xor of all pairs 

12. Monotonicity of or - Given or of the  sequence of numbers a1,a2..an if we add any number to it the new or will be atleast as large as previous or. Menaing adding the values to or sum only increases it.
13. Similarly the and is monotonically the descreasing function.
14. Prefix xor pattern for numbers `1^2^3...^n` The XOR sum of any four consecutive numbers starting from a multiple of 4 (4k,4k+1,4k+2,4k+3) is always 0. This is because all the higher-order bits cancel out (they appear 4 times), and the last two bits (00⊕01⊕10⊕11) also sum to 0.

So if g(n) = `(1^2^3...^n)` then 

This pattern holds true for all N. We can summarize it based on N(mod4):

- If N(mod4)=0, then g(N)=N
- If N(mod4)=1, then g(N)=1
- If N(mod4)=2, then g(N)=N+1
- If N(mod4)=3, then g(N)=0

Common problems

| Problem Type                                 | Example                        |
| -------------------------------------------- | ------------------------------ |
| Find single non-repeating element            | XOR of all elements            |
| Count pairs with given XOR                   | Use frequency + XOR properties |
| Maximum XOR subarray                         | Trie of prefixes               |
| DP over subsets                              | Traveling salesman, partition  |
| Constructing masks for compatibility         | Matching problems              |
| Set cover / minimal cover                    | bitmask DP                     |
| Graph path existence with parity constraints | BFS over bitmask states        |

15. One common pattern of problems is that xor of number with itself is 0 so if an array has all the numbers come twice except one. then this one number can be identified by total xr. 
16. Another is all numbers come two times except two numbers which come one one time. Here One thing to note is if this xor sum has some bit 1. Then this bit will be one in exaclty one of the two. With this we can divide the numbers into two sets one having this bit set and others don't. Now the xor sum of each set will be requried answer. 
17. Finding bitwise range query. 

First most simple is xor range query as it satisfies the following identity due to two exact same numbers cancelling out.
`qry(a,b) = qry(0,a-1)^qry(0,b)` So it solvable using prefix sums.
Second for bitwaise `and` and `or`. 
To solve it we keep count of numbers of set bits encountered for each bit.
Now for a while lets consider a bit only. If we have cnt of prefix then count in a given range(i,j) = `cnt[j]-cnt[i-1]`. 
Now this bit and is 1 if entire range has `cnt==j-i-1` or numbers of 1 in this range is equal to numbers in between. In the other words if all the numbers had this `bit=1`. Similary for or we just check if number of 1 is atleast 1. 

18. Finally there are many problems realted to the finding of bitwise or or bitwise xor quickly. We have seen how to calculate for xor using mod4 trick. For the `or` in range `a|(a+1)|(a+2)...b` observe that the bits in the common prefix will remain the sum. However the bit(most significant) where we get the first difference from there onwards every bit in answer will be `1`. 
19. For finding the bitwise and of a range quickly observe that the bitwise and is 1 if both bit remain 1. clearly only subset of set prefix can be 1. It turns out that the 1's in common prefix will remain `1` all the other bits will become 0. because of mismatch. 

```cpp
int rangeBitwiseAnd(int left, int right) {
	int ans=0;
	for(int b=31;b>=0;b--){
		if((left&(1<<b))==(right&(1<<b))){
			ans|=(left&(1<<b));
		}else{
			break;
		}
	}
	return ans;
}
```

20. Finally in bitwise problems great attention must be given to the fact if we can solve the problem in bit by bit manner or not. If we can then these problems become very easy to solve.

## Arithematic sum and bit wise

the sum of two numbers can be written as

`a+b = a^b+2*(a&b)` At this point it is important to look the sum as sum of individual bits and look at them independently 
When we do so complex additon can be broken into bitwise operation and solve individually. 

Why this identity? The sum without carry is captured by the xor sum and and captures the carry. But note that if carry was found on the bit b then in final answer it will be added to bit b+1(its carry). So multiply by 2 or `<<` left shift handles that.

So in some problems the `a^b` is constant and so sum to be found is determined entirely by the `a&b` formulation. This gives very easy point to maximise. 

**Minimizing `a+b` for a Fixed `a⊕b`:** Since `a+b=(a⊕b)+2⋅(a∧b)`, to minimize the sum, you must minimize the carry term `a∧b`. This is a standard constraint in problems that involve reconstructing numbers.

#### Gray code

Gray code is the list of numbers such that any two successive numbers differ by one bit only. Now there is acutally a standard way to formulate theb grey code. 

```
gray[i] = i ^ (i>>1)
```

Now why is this formulation correct. - Binary counting flips bits in a **ripple carry** pattern.XOR with a right-shift “smooths out” that ripple — keeping only the edge of the ripple.


### Xor basis

To be done

