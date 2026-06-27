
---

`Code can be memorized by any one, the factor that matters is can you use it or not.`- Uttkarsh Gupta

We define the kmp array as follows:

The ith entry of kmp array is the longest proper proper prefix of string s(0,i) that is also the proper suffix of string s(0,i) where the 0 and ith index are inclusive. Proper means prefix or suffix must not engulf the whole string.

By definition the value `kmp[i]`  shows the longest length of a substring ending in position  $i$  that coincides with the prefix. 

Lets go over one example:

`s = "abacabacdab"`
`kmp = [0,0,1,0,1,2,3,4,0,1,2]`

We observe two things here that either value of kmp array increases or it goes down.  

Th: `kmp[i]<=kmp[i-1]+1`
Proof: The value of kmp can never increase more than 1 for a given index. The reason being ifthat was the case some earlier bigger length should have been found. And is the crux that proper prefix can increase atmost by 1. 

Now the question is how to find this array

Observe that brute force solution will be `O(n^2)`. Now we will be going over a `O(n)` solution which can solve it in much quicker manner.

First of all if suppose for index i kmp value is `kmp(i)` now for index `i+1` kmp value `kmp(i+1)` can be `kmp(i)+1` if `s[kmp[i]+1]==s[i+1]` becasue any way it was matching till ith index now for 

![[Pasted image 20251002144638.png]]

Otherwise the length `km[i+1]<=kmp[i]` which means proper prefix actually is the subprefix of `s[0:kmp[i]]` and that is actually the important part.

![[Pasted image 20251002144934.png]]

So observe that both red parts are same which means to figure out suffix in the entire length it is just enough to concentrate on the begining part. Now we can find the kmp of red part what ever the value is if char at index next matched then we will have the next longest strings. 

Code:

```cpp
vector<int> get_kmp(string s){

	int n = s.size();
	vector<int> kmp(n+1,-1);
	int i=0,j=-1;
	
	while(i<n){
		while(j!=-1 && s[i]!=s[j])j=kmp[j];
		i++;j++;
		kmp[i]=j;
	}
	
	return kmp;
}
```

Observe that j stores the value of current kmp and i stores the index however we are storing the values in 1 based indexing.

Exaplanation: consider for any index i the j value is `kmp[i-1]` now see that we are checking for `j!=-1 and s[i]!=s[j]` this checks for potential match. If not we go over next potential match in the end j will be -1 and we will found no match. 

This algorithm is `O(n)` reason being j can increase 1 each time so it can descrease at most `n` thus inner while can run in summation for n times so we can say that entire algorithm is `O(n)`.


### Applications of kmp:

#### Needle in the hay:

Given two strings `s` and `t`, return the index of the first occurrence of `t` in `s`, or `-1` if `t` is not part of `s`.

##### Solution: 

1. create `temp = t#s`
2. find kmp of this array 
3. Answer is the first index i where `kmp[i]==t.size()`

Why?? Since t comes as prefix 

![[Pasted image 20251002152904.png]]

### Minimum insertions to make a string palindrome:

Find the minimum numbers of characters to add at end so that string becomes palindrome.
##### Solution:

Suppose we have any string `abcb` now its palindromic append to the end should be the some prefix of string. Meaning that remaining inner part should be a palindrome in itself. So now problem is converted to finding the longest suffix that is palindrome in itself. 

###### Finding longest suffix that is also palindrome. 

Now we need to find the connection between palindrome and kmp. Essentially kmp deals with prefix and suffix while palindrome deals with reading the string from forward and from backward same. 

We introduce the following pattern to solve palindrome problems with kmp as :

consider following string `temp = rev(s)#s`  The `kmp.back()` is essentially the longest suffix that is also the palindrome.

![[Pasted image 20251002160004.png]]

Reason being the fact that `kmp.back()` is the longest prefix that is also suffix for the last index. Note that string was reversed in the first part. So if length `l` is the prefix prefix and suffix as well thi means both match and so if read from back they will be the same. 

Similarly if the ask was to append at the start then equivalent deduction will be to find the longest prefix that is also palindrome in this case we will apply kmp at 

`s+#+rev(s)` and that is it.

## Period

In many string analysis problems, we are interested in determining whether a string can be represented as repetitions of a smaller substring — and if so, what the size of that repeating pattern is. This repeating pattern size is known as the **period** of the string.

However, there are two versions of the problem:

1. **Without partial matching** – the string must be made up of **whole** repeated blocks.
2. **With partial matching** – prefixes that overlap with suffixes are allowed (as in **CSES “Period”** problem).

KMP is a pattern matching algorithm that precomputes a **prefix function** (also known as the **LPS array**, or _longest proper prefix which is also a suffix_ array).For each position `i` in the string `S`, `pi[i]` represents the **length of the longest proper prefix** of `S[0...i]` which is **also a suffix** of this substring.

This prefix function helps detect repetitions, because if a suffix equals a prefix, that implies a repeating pattern in between.

Let’s start with the **simpler version**, where we only want to check if the string is made up of **whole repetitions** of some substring.

Using the prefix function:

- Let `n` be the length of the string.
- Let `L = kmp[n - 1]`, i.e., the length of the longest proper prefix that is also a suffix of the entire string.

The candidate period is then: `period = n - L` if period is the divisor of the `n` then its the answer

### Why it works?

The reason this works lies in how KMP captures **self-similarity** within the string.

If the string has a prefix that’s also a suffix of length `L`, that means the first `L` characters reappear at the end — a signature of periodicity.  
Subtracting this overlap from the full length gives the potential repeating block size: `n - L`.

If `n` is divisible by `n - L`, then the same block perfectly tiles the whole string.  
Otherwise, the border doesn't repeat cleanly meaning there’s no full repetition pattern.

Now, consider a more advanced case: **partial matches** are allowed.

For instance, in `"aaaaa"`, the prefixes that overlap with suffixes are not necessarily forming perfect repetitions, yet they represent valid _borders_ or _periodic structures_ within the string.

In the **CSES problem “Period”**, the goal is to print **all lengths `k`** such that `S` can be considered as a repetition of a prefix of length `k`, possibly with a **partial match** at the end.

To find all such possible periods, we can iteratively follow the prefix function chain:

```cpp
int k = n;
while (k > 0) {
    k = pi[k - 1];
    if (k > 0)
        cout << k << " ";
}
```

### Counting the number of occurrences of each prefix

Here we discuss two problems at once. Given a string  

`s`  of length  `n` . In the first variation of the problem we want to count the number of appearances of each prefix  `s[0...i]`  in the same string. In the second variation of the problem another string  `t` is given and we want to count the number of appearances of each prefix  `s[0 ... i]`  in  `t` .

First we solve the first problem. Consider the value of the prefix function  

`pi[i]`  at a position  `i`. By definition it means that the prefix of length  `pi[i]`  of the string  `s`  occurs and ends at position  `i` , and there is no longer prefix that follows this definition. At the same time shorter prefixes can end at this position. It is not difficult to see, that we have the same question that we already answered when we computed the prefix function itself: Given a prefix of length  `j`  that is a suffix ending at position  `i` , what is the next smaller prefix  `< j`  that is also a suffix ending at position  `i` . Thus at the position  `i`  ends the prefix of length  `pi[i]` , the prefix of length  `pi[pi[i] - 1]` , the prefix  `pi[pi[pi[i] - 1] - 1]` , and so on, until the index becomes zero. Thus we can compute the answer in the following way.

```cpp
vector<int> ans(n + 1);
for (int i = 0; i < n; i++)
    ans[pi[i]]++;
for (int i = n-1; i > 0; i--)
    ans[pi[i-1]] += ans[i];
for (int i = 0; i <= n; i++)
    ans[i]++;
```

Here for each value of the prefix function we first count how many times it occurs in the array  `pi` , and then compute the final answers: if we know that the length prefix  `i`  appears exactly  `ans[i]`  times, then this number must be added to the number of occurrences of its longest suffix that is also a prefix. At the end we need to add  `1`  to each result, since we also need to count the original prefixes also.

Now let us consider the second problem. We apply the trick from Knuth-Morris-Pratt: we create the string  `s + # + t` and compute its prefix function. The only differences to the first task is, that we are only interested in the prefix values that relate to the string  $t$ , i.e.  `pi[i]`  for  `i >= n + 1` . With those values we can perform the exact same computations as in the first task.

