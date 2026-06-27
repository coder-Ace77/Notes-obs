
---

## Meet in the middle 

A technique where brute force is large but half brute force is still manageble. The idea is simple we should be able to do the brute force of the half and must be able to combine the results efficiently. 

Three step process

- **Divide** the problem into two halves.
- **Enumerate all possibilities** for each half (independently).
- **Combine the results** of both halves to form the final solution.

Example max subset sum where values are large `a[i]<=1e9` and `n<=40`. 

```cpp
vector<long long> leftHalf, rightHalf;
int mid = n / 2;

// Step 1. Split into halves
for (int i = 0; i < (1 << mid); i++) {
    long long sum = 0;
    for (int j = 0; j < mid; j++)
        if (i >> j & 1) sum += a[j];
    leftHalf.push_back(sum);
}

for (int i = 0; i < (1 << (n - mid)); i++) {
    long long sum = 0;
    for (int j = 0; j < n - mid; j++)
        if (i >> j & 1) sum += a[mid + j];
    rightHalf.push_back(sum);
}

// Step 2. Sort rightHalf for binary search
sort(rightHalf.begin(), rightHalf.end());

// Step 3. For each left sum, find best right sum ≤ S - left
long long ans = 0;
for (long long x : leftHalf) {
    if (x > S) continue;
    long long rem = S - x;
    auto it = upper_bound(rightHalf.begin(), rightHalf.end(), rem);
    if (it != rightHalf.begin()) {
        --it;
        ans = max(ans, x + *it);
    }
}
cout << ans;
```

### Contribution to the sum technique

This paradigm is deployed when a problem asks for the aggregate sum of a specific property across an exponentially large set of configurations (e.g., the sum of the maximums of all possible subarrays, or the total XOR of all subsets).

Generating all configurations (subarrays, subsets, or permutations) is computationally impossible. To solve this, you must completely invert the axis of iteration. Instead of asking "What does this configuration contain?", you ask "How many configurations contain this specific element?"

**Phase 1: Element Isolation.** You strip away the concept of configurations entirely and look only at the raw input elements. You iterate through the input sequentially, treating each element as a standalone mathematical entity.

**Phase 2: The Combinatorial Multiplier.** For each element, you define the strict boundary conditions under which this element dictates the property in question (e.g., the exact left and right bounds where this element is the absolute maximum). You then use basic combinatorics to calculate the exact number of configurations that can be formed within those boundaries. You multiply the element's raw value by this combinatorial count (its "contribution") and add it to a global accumulator. 

This paradigm relies on the distributive property of summation and the interchangeability of nested sums.

Mathematically, if you want to find the sum of a function $f$ over all configurations $C$, the naive approach is:

By swapping the order of summation, you isolate the elements $E$:

$$ \sum_{e \in E} \left( f(e) \times \text{Count of valid configurations containing } e \right) $$

Because the counts are derived mathematically (often in $O(1)$ time per element using formulas or monotonic stacks) rather than generated, an $O(N^2)$ or $O(2^N)$ state space is perfectly compressed into an $O(N)$ linear sweep. You bypass the structural generation entirely by calculating the exact arithmetic weight of each element's existence in the theoretical superset.


## The Median Alignment (Absolute Deviation Minimization)

This paradigm is utilized in spatial optimization or cost-minimization problems where multiple entities must converge to a single target value, and the cost of movement scales linearly with distance (Absolute Difference).

### The Abstract Mechanism

When asked to find an optimal meeting point that minimizes total travel cost for a set of distributed points, the arithmetic mean (average) fails because it is skewed by outliers.

**Phase 1: Spatial Ordering.** You discard the magnitude of the distances and focus entirely on the ordinal sequence. You sort the entities strictly by their spatial or numerical position, establishing a 1D coordinate axis.

**Phase 2: Ordinal Convergence.** You ignore the actual distances between the points and look only at their index in the sorted array. The optimal target is mathematically guaranteed to be the exact ordinal center of the dataset—the median. If the dataset has an even number of elements, any point lying geographically between the two central-most elements is equally optimal. You select this median and compute the absolute differences against it.

### The Mathematical Justification (Nested Intervals & Gradients)

The optimality of the median is proven by two mathematical perspectives:

**Perspective 1: The Gradient of Cost.**

Imagine placing your target at some arbitrary point on the axis. If there are $L$ points to your left and $R$ points to your right, moving the target $1$ unit to the right will increase the distance to all $L$ points by $1$ and decrease the distance to all $R$ points by $1$. The net change in total cost is $(L - R)$.

To minimize cost, you must move in the direction that lowers the net change until $(L - R) = 0$. The only place on the axis where the number of entities to the left exactly equals the number of entities to the right is the median.

**Perspective 2: Nested Pairings.**

Consider only the absolute leftmost point and the absolute rightmost point. To minimize the sum of distances for just this pair, the target can be placed _anywhere_ strictly between them; the cost will identically equal the distance between the two points.

You can peel away the outermost pair and repeat this logic for the next outermost pair. This creates a series of universally valid nested intervals. The intersection of all these optimal intervals collapses precisely onto the median element (or the median gap). Therefore, the median satisfies the optimal placement for every symmetric pair simultaneously.

Problem:

You are given an integer array $A$ of size $N$. In one move, you can increment or decrement any element by $1$. Find the minimum number of total moves required to make all elements equal.

Because the nested interval math proves that the cost of a symmetric pair is _always_ just the difference between their values ($Right - Left$), we do not even need to calculate or isolate the actual median target value.

```python
def minMovesToMakeEqual(nums):
    # Phase 1: Spatial Ordering
    nums.sort()
    
    left = 0
    right = len(nums) - 1
    total_moves = 0
    
    # Phase 2: Nested Interval Summation
    while left < right:
        # The cost to align this specific symmetric pair 
        # is mathematically guaranteed to be their absolute difference.
        total_moves += nums[right] - nums[left]
        
        left += 1
        right -= 1
        
    return total_moves
```

