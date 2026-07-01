
---

Actual kadance algorthm is essentailly a dynamic programming solution. The thing to understand about Kadane's algorithm is that it is not really a "trick" for sums  it is a way of _reframing_ a global question as a local one, and that reframing is the entire conceptual payload. The naive question "what is the maximum-sum contiguous subarray in this array?"

Kadane's insight is to refuse to answer that question directly and instead answer a much narrower one at every position: "what is the best subarray that _ends exactly here_, at index i?"

Contiguity is doing all the work. A subarray ending at index i either includes the element at i−1 or it doesn't; there is no third possibility, no "skip the awkward middle element." Because the choice is binary, the recurrence is clean: the best subarray ending at i is either the single element a[i] standing alone, or a[i] glued onto the best subarray that ended at i−1. You never have to consider anything more exotic, because contiguity forbids it.

That binary choice **extend or restart** is the soul of the algorithm. As you sweep left to right, you carry one running quantity: the best sum achievable by a subarray ending at the current spot. At each new element you ask a single question: "Is the baggage I'm carrying helping me or hurting me?" If the running sum has gone negative, it is pure liability.

So our dp solution is as follows - 

```cpp
// dp[i] = max sub array sum ending at i
vector<int> dp(n);
dp[0]=arr[0];

for(int i=1;i<n;i++){
	// two cases either extend prev or start fresh
	dp[i]=max(dp[i-1]+arr[i],arr[i]);
}

// max(dp) is answer
```

### Modified kadance

Kadane's algorithm is a **single-pass dynamic program in which you carry forward the minimum amount of state required to make the extend-versus-restart decision optimally.** In the vanilla problem that state is a single number. _Every_ "modified Kadane" problem is, conceptually, the same skeleton  sweep once, maintain a best-ending-here, decide extend-or-restart, track a global best  but with a richer notion of _what state you have to carry_ so that the local decision stays correct under new rules. When you sit down in front of an unfamiliar problem and suspect Kadane applies, the single question to ask yourself is: **"What do I need to remember about the best object ending at the previous position so that I can correctly decide what happens when I extend it by one more element?"** Answer that, and you've found the modification. The mechanical loop barely changes; the _state design_ is the whole game.

The modifications tend to come from four directions, and recognizing which one you're facing is most of the battle. **First, the objective stops being a plain sum.** The clearest case is a _product_: a large negative running product is one negative element away from becoming a large _positive_ one, so the most-negative value you've seen is not garbage to be discarded it's a coiled asset. The fix is to widen the state from one number to two, tracking both the maximum and the minimum product ending here, because under multiplication a negative input _swaps their roles_.

**Second, the structure of the array is twisted** it might be circular, so the optimal subarray could wrap around the end and you have to reason about both the normal case and the wrap case (cleverly, the best wrapping subarray is the total minus the _minimum_ non-wrapping subarray, so you run Kadane twice with opposite signs). Or the array is conceptually concatenated with copies of itself, and you have to reason about prefixes, suffixes, and full-array totals rather than just interior runs.

**Third, a constraint is bolted on**, like "you may delete at most one element" or "you must apply exactly one operation somewhere." Now the state becomes a tiny tuple best-ending-here _given zero deletions used so far_ and best-ending-here _given one deletion already spent_ and the recurrence describes how those two states feed each other as you extend.

**Fourth, the domain lifts to two dimensions**: to find the maximum-sum rectangle in a matrix, you fix a pair of top and bottom rows, collapse every column between them into a single summed value, and run ordinary 1-D Kadane across that collapsed strip turning a daunting 2-D search into an outer loop over row-pairs wrapping a Kadane you already know.

#### Problem

Your task is to find max sub array sum given we can do one of the following operation once. 

- Multiply each number in the chosen subarray by `k`.
- Divide each number in the chosen subarray by `k`. Floor for pve and ceil for neg. 

####  Solution

Now here we introduce concept of state. We will be using 4 states in dp for each ending position. 

- Still not started to applied div/mul
- Currently in the mul loop
- Currently in the div loop
- Finished

Now since we can calculate the current states from prev ones our solution is pretty simple to reason about. 

```cpp
class Solution {
public:
    long long opDiv(long long n, long long k) {
        if (n < 0) return -((-n) / k);   // ceiling toward zero for negatives
        return n / k;                    // floor for non-negatives
    }

    long long maxSubarraySum(vector<int>& nums, int k) {
        int n = nums.size();
        const long long NEG = LLONG_MIN / 4;   // impossible-state sentinel

        // a: unmodified prefix | b: multiply block active
        // c: divide block active | d: unmodified suffix (after block)
        long long a = nums[0];
        long long b = 1LL * nums[0] * k;
        long long c = opDiv(nums[0], k);
        long long d = NEG;                 // no block can end before index 0

        long long ans = max(b, c);         // only operated states are valid answers

        for (int i = 1; i < n; i++) {
            long long x = nums[i];
            long long na = max(a, 0LL) + x;                       // continue/start unmodified prefix
            long long nb = max({b, a, 0LL}) + 1LL * x * k;        // continue block | start after prefix | fresh
            long long nc = max({c, a, 0LL}) + opDiv(x, k);        // same, for divide
            long long nd = max({b, c, d}) + x;                    // block ended, append unmodified (no fresh start)

            a = na; b = nb; c = nc; d = nd;
            ans = max({ans, b, c, d});
        }
        return ans;
    }
};
```

Finally sometimes the problem will involve the change of array and then applying `kadance` in disquise. For an instance we may modify the array to the `adj diff` array and then answer is maximum subarray sum in the problem involving incremental changes for an example stock buy and sell. 

**Maximum Subarray Sum with One Deletion.** The added-constraint case in its purest teaching form. State becomes a pair: best-ending-here with no deletion used, and best-ending-here with one deletion already spent, where the second feeds off the first. This is the template for every "at most one operation" Kadane.

With this being said kadance can appear in many other forms especially when the ask is maximum subarray sum in either real or disguished form. 