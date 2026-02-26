
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

