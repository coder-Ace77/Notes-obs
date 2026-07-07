
---

Hashing is a good tooling to solve the equlity of two strings. The idea is that each substring if can be mapped to a number via some function. 

A **hash** `h(s)` maps a string to a number. If we have a good hash:

- `s == t` ⟹ `h(s) == h(t)` (always a function is deterministic)
- `h(s) != h(t)` ⟹ `s != t` (contrapositive **guaranteed**)
- `h(s) == h(t)` ⟹ `s == t` (**only probably**)

So comparing hashes gives us:

- A cheap _rejection_ test that is always correct.
- A cheap _acceptance_ test that is usually correct, and can be made correct-with-tiny-error or verified.

### Polynomial hashes

Polynomial hashing is a technique used to map an arbitrary-length string (or sequence) to a fixed-size integer. The core idea is to treat the characters of a string as coefficients of a polynomial and evaluate that polynomial at a specific value (the **base**), taking the result modulo a large prime number (the **modulus**).

Given a string $S = s_0s_1s_2 \dots s_{n-1}$ of length $n$, we map each character $s_i$ to an integer value (for example, 'a' $\to 1$, 'b' $\to 2$, etc.).

The hash value $H(S)$ is computed using a chosen base $p$ and a modulus $m$:

$$H(S) = (s_0 \cdot p^{n-1} + s_1 \cdot p^{n-2} + \dots + s_{n-2} \cdot p^1 + s_{n-1} \cdot p^0) \pmod m$$

Alternatively, you can write the polynomial in ascending powers of $p$. Both ways work perfectly fine as long as you remain consistent:

$$H(S) = (s_0 \cdot p^0 + s_1 \cdot p^1 + \dots + s_{n-1} \cdot p^{n-1}) \pmod m$$

Note that mod taking by `m` means we have converted this from the determinstic to prob system meaning two strings can now have same hash value. 

To minimize **collisions** (different strings producing the same hash value), the choice of constants is critical:

- **The Base ($p$):** It should be a prime number strictly greater than the size of the alphabet. For lowercase English letters (26 characters), $p = 31$ or $p = 53$ are common choices.
- **The Modulus ($m$):** It should be a large prime number to distribute hash values uniformly. $m = 10^9 + 7$ or $m = 10^9 + 9$ are standard choices because they easily fit within standard 64-bit integer limits during multiplication.

#### Rolling over

Take the window `T[i..i+m-1]` with hash `H_i`. We want `H_{i+1}` for `T[i+1..i+m]` cheaply. Removing mod for illustrative purposes. 

```
H_i     = T[i]·b^(m-1) + T[i+1]·b^(m-2) + … + T[i+m-1]·b^0
H_{i+1} =                T[i+1]·b^(m-1) + … + T[i+m-1]·b^1 + T[i+m]·b^0
```

Looking closeyl we can write it as - 

```
H_{i+1} = ( H_i − T[i]·b^(m-1) )·b + T[i+m]
```

Three operations, all `O(1)` if we precompute `b^(m-1) mod M`. That's the rolling hash. Total matching cost: `O(n + m)`.

This is simple single string comparison pattern. 

```cpp
// Classic Rabin-Karp single-pattern search, mod arithmetic
long long pw = 1;              // b^(m-1) mod M
for (int i = 0; i < m-1; i++) pw = pw*B % M;

long long hp = 0, hw = 0;      // hash of pattern, hash of first window
for (int i = 0; i < m; i++) {
    hp = (hp*B + P[i]) % M;
    hw = (hw*B + T[i]) % M;
}

for (int i = 0; i + m <= n; i++) {
    if (hw == hp) {
        if (T.compare(i, m, P) == 0) report(i);
    }
    if (i + m < n) {
        hw = (hw - T[i]*pw % M + M) % M;   // note that we are using reverse method
        hw = (hw*B + T[i+m]) % M;
    }
}
```

However for the advanced patterns we ususally want to have `hash(l,r)` hash of substring in range `l to r`. 

Define prefix hashes (here the _first_ character carries the _highest_ power — a common convention):

```
h[0] = 0
h[i] = h[i-1]·b + s[i-1]     (mod M),  for i = 1..n
```

So the `hash` is 

```
hash(l, r) = ( h[r+1] − h[l]·b^(r-l+1) )  mod M
```

`h[r+1]` contains `s[l..r]` but each of its characters is inflated by an extra factor of `b^(r-l+1)` coming from the prefix `s[0..l-1]`. `h[l]·b^(r-l+1)` is exactly that prefix, scaled to line up. Subtract it and the prefix cancels, leaving the substring's hash.

So a generalized hashing function will look like 

```cpp
struct Hashing {
    int n; long long B, M;
    vector<long long> h, pw;
    Hashing(const string& s, long long B, long long M): B(B), M(M) {
        n = s.size();
        h.assign(n+1, 0); pw.assign(n+1, 1);
        for (int i = 0; i < n; i++) {
            h[i+1] = (h[i]*B + s[i]) % M;      // map char -> nonzero int if you like: s[i]-'a'+1
            pw[i+1] = pw[i]*B % M;
        }
    }
    // hash of s[l..r], 0-indexed inclusive
    long long get(int l, int r) {
        long long res = (h[r+1] - h[l]*pw[r-l+1]) % M;
        return (res % M + M) % M;              // normalize negative (§6)
    }
};
```

With this you can equality-test any two substrings in `O(1)`. 

#### Probabilities

General formula for the collision probabilty between a pair of strings `(s,t)` is 

```
P[collision] <= (L-1)/M
```

So for one comparison its about 1/1000 with usual parimeters of b = 31 and M = `1e9+7`. But with n comparisons over window formula changes to `n(L-1)/M`. Meaning we can get a false positive for arounf million length string. Verification makes correctness certain; hashing just makes the common case (`h differs`) fast. A verify costs `O(m)` but only fires on hash matches, which are rare, so amortized cost stays `O(n+m)`. 

Now the dangerous regime. Many problems **can't verify** because they never hold two candidate strings side by side — they throw hashes into a `set`/`map` and count distinct ones, or compare all-pairs implicitly. Example: "count distinct substrings," "how many distinct rotations," etc.

If you store `k` hashes, each roughly uniform in `[0, M)`, Since each hash is a random value between `(0,M)` the prob of collision is 

```
E[collisions] ≈ C(k,2) / M ≈ k² / (2M)
```

Plug in real numbers and a M = 1e9+7 leads to around 500 collisions for a standard `1e6` length string. This is why this pair thing is not valid for sets with small lager size of sets. This however can be resolved by taking larger `M`. 

That's why "distinct substrings" style problems demand **~10^18 of hash space** — either double hashing (two independent `~10^9` mods, combined space `M1·M2 ≈ 10^18`) or one `2^61−1` modulus. 

For the competitve programing the standard method is to use stronger hash `2^61-1` and a random base this beats the bithday paradox and mostly is safe then to do a million pair comparisons. 

When you need to be _extra_ safe (or the problem clearly hunts hash solutions), use **double hashing**: two independent `(base, mod)` pairs, treat two strings as equal only if _both_ hashes match. Effective space `≈ 10^18–10^36`.

General code

```cpp
struct HashingDouble {
    int n;
    long long B1, M1, B2, M2;
    vector<long long> h1, h2, pw1, pw2;

    HashingDouble(const string& s,
                  long long B1, long long M1,
                  long long B2, long long M2)
        : B1(B1), M1(M1), B2(B2), M2(M2) {
        n = (int)s.size();
        h1.assign(n+1, 0); pw1.assign(n+1, 1);
        h2.assign(n+1, 0); pw2.assign(n+1, 1);
        for (int i = 0; i < n; i++) {
            long long c = s[i] - 'a' + 1;        // map to 1..26, never 0 (see Part 1, §6d)
            h1[i+1] = (h1[i]*B1 + c) % M1;
            h2[i+1] = (h2[i]*B2 + c) % M2;
            pw1[i+1] = pw1[i]*B1 % M1;
            pw2[i+1] = pw2[i]*B2 % M2;
        }
    }

    // fingerprint of s[l..r], 0-indexed inclusive, packed into one 64-bit key
    long long get(int l, int r) {
        long long a = (h1[r+1] - h1[l]*pw1[r-l+1]) % M1;
        a = (a % M1 + M1) % M1;                  // normalize negative
        long long b = (h2[r+1] - h2[l]*pw2[r-l+1]) % M2;
        b = (b % M2 + M2) % M2;
        return a * M2 + b;                       // unique combined key, fits in long long
    }
};
```

Note that it produced one answer instead of two. Its done so that we can store the result as one integer as finger print of the string instead of using two integers. 

Note we should choose base randomly to make it hack proof. 

```cpp
mt19937_64 rng(chrono::steady_clock::now().time_since_epoch().count());
long long B1 = uniform_int_distribution<long long>(256, (long long)1e9)(rng);
long long B2 = uniform_int_distribution<long long>(256, (long long)1e9)(rng);
HashingDouble H(s, B1, 1000000007LL, B2, 1000000009LL);
```

### Usage

First and most simple usage is to check if one string is contained inside another one this is the exact rival of the kmp algorithm.

Given a text and a pattern, report every position in the text where the pattern begins.

```cpp
vector<int> findAll(const string& T, const string& P) {
    int n = (int)T.size(), m = (int)P.size();
    vector<int> occ;
    if (m == 0 || m > n) return occ;

    mt19937_64 rng(chrono::steady_clock::now().time_since_epoch().count());
    unsigned long long B = uniform_int_distribution<unsigned long long>(256, HashingLong::MOD - 1)(rng);

    HashingLong ht(T, B), hp(P, B);          // SAME base B for both — this is the rule
    unsigned long long target = hp.get(0, m - 1);

    for (int i = 0; i + m <= n; i++) {
        if (ht.get(i, i + m - 1) == target) {
            // if (T.compare(i, m, P) == 0)   // uncomment to make it certain
            occ.push_back(i);
        }
    }
    return occ;
}
```

#### Longest palindromic substring

Then finding the maximal length palindromic string. Although we can use manacher algorithm to solve this. With the hashes its pretty simple. Since the palindrome follows monotonicity where if a `substr(l,r)` is palindrome any internal substr is the palindorme itself. What we do is to have binary search to solve the problem for each center point as well as for the even points. 

```cpp
pair<int,int> longestPalindrome(const string& s) {   // returns [l, r] of the best, inclusive
    int n = (int)s.size();
    if (n == 0) return {0, -1};

    string r = s;
    reverse(r.begin(), r.end());

    mt19937_64 rng(chrono::steady_clock::now().time_since_epoch().count());
    unsigned long long B = uniform_int_distribution<unsigned long long>(256, HashingLong::MOD - 1)(rng);

    HashingLong hf(s, B), hr(r, B);          // forward and reversed, SAME base

    auto isPal = [&](int l, int rr) -> bool {
        return hf.get(l, rr) == hr.get(n - 1 - rr, n - 1 - l);
    };

    int bestLen = 1, bestL = 0;

    for (int c = 0; c < n; c++) {
        // odd-length palindrome centered on character c: range [c-d, c+d]
        {
            int lo = 0, hi = min(c, n - 1 - c), best = 0;
            while (lo <= hi) {
                int mid = (lo + hi) / 2;
                if (isPal(c - mid, c + mid)) { best = mid; lo = mid + 1; }
                else hi = mid - 1;
            }
            if (2 * best + 1 > bestLen) { bestLen = 2 * best + 1; bestL = c - best; }
        }
        // even-length palindrome centered between c and c+1: range [c-h+1, c+h]
        if (c + 1 < n) {
            int lo = 1, hi = min(c + 1, n - 1 - c), best = 0;
            while (lo <= hi) {
                int mid = (lo + hi) / 2;
                if (isPal(c - mid + 1, c + mid)) { best = mid; lo = mid + 1; }
                else hi = mid - 1;
            }
            if (2 * best > bestLen) { bestLen = 2 * best; bestL = c - best + 1; }
        }
    }
    return {bestL, bestL + bestLen - 1};
}
```

Now usages like number of distinct substrings or longest common substring both actually suffer from same thing as too much collision which makes them not preferred for usage. 

#### Comparing substrings lexicographically (and the smallest rotation)

Two substrings compare lexicographically by their first point of disagreement: they agree on some common prefix and then either one runs out (the shorter one is smaller) or a character differs (the string with the smaller character there is smaller). Fingerprints let you find the length of that shared prefix fast by binary-searching it — ask "do the first `k` characters of each match?", which is one fingerprint comparison, and binary-search the largest `k` that still matches. Once you know the shared-prefix length you look at the single character just past it and the comparison is settled. So a lexicographic comparison of two arbitrary substrings costs a logarithm instead of a linear scan.

The shared prefix has the nesting property once more: if the first `k` characters agree, the first `k − 1` agree too, so the matching prefix lengths form a run from zero up to the true common-prefix length, and binary search finds its top. For the smallest rotation, notice that every rotation of a string of length `n` is a length-`n` block of the string concatenated with itself — the rotation starting at position `k` is the block from `k` to `k + n − 1` in the doubled string. So you scan all `n` starting positions, keep the best one, and settle each head-to-head with the logarithmic comparison above. (Booth's algorithm does smallest rotation in linear time, but the hashing comparison is a general-purpose tool: it sorts suffixes, ranks rotations, and answers any "which substring is smaller" question, not just this one.)

```cpp
// longest common prefix length of the substrings starting at i and j, capped at maxLen
int lcp(HashingLong& h, int i, int j, int maxLen) {
    int lo = 0, hi = maxLen, res = 0;
    while (lo <= hi) {
        int mid = (lo + hi) / 2;
        if (mid == 0) { lo = 1; continue; }
        if (h.get(i, i + mid - 1) == h.get(j, j + mid - 1)) { res = mid; lo = mid + 1; }
        else hi = mid - 1;
    }
    return res;
}

// compare t[a1..b1] vs t[a2..b2] lexicographically: -1 if first smaller, 0 equal, 1 if first larger
int cmpSub(const string& t, HashingLong& h, int a1, int b1, int a2, int b2) {
    int len1 = b1 - a1 + 1, len2 = b2 - a2 + 1, len = min(len1, len2);
    int l = lcp(h, a1, a2, len);
    if (l == len)                            // one is a prefix of the other (or they're equal)
        return (len1 == len2) ? 0 : (len1 < len2 ? -1 : 1);
    return t[a1 + l] < t[a2 + l] ? -1 : 1;   // they differ at the first post-prefix character
}

// starting index (in the original string) of its lexicographically smallest rotation
int smallestRotation(const string& s) {
    int n = (int)s.size();
    string t = s + s;

    mt19937_64 rng(chrono::steady_clock::now().time_since_epoch().count());
    unsigned long long B = uniform_int_distribution<unsigned long long>(256, HashingLong::MOD - 1)(rng);

    HashingLong h(t, B);

    int best = 0;
    for (int k = 1; k < n; k++)
        if (cmpSub(t, h, k, k + n - 1, best, best + n - 1) < 0) best = k;
    return best;
}
```

