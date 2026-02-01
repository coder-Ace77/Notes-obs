
---

A string of parenthesis is called well formed if 

- Every opening bracket has a corresponding closing bracket of same type.
- Brackets are closed in the correct with of same time. The ordering is that last to open bracket must be closed first. This naturally ensures the stack structure. 
- Finally all the opening brackets comes before corresponding closing brackets. This also means while traversing the string the count of closing bracket can never exceed the closing brackets. 

Now with done to check a parenthesis is `valid/wellformed` we need to check only two things. 
The running count of opening brackets is always more than or equal to the running count of closing brackets. And finally all the opening brackets get closed off. 

Note that there is an alternate definition of valid parenthesis which is used - 

A bracket sequence is called regular if it is possible to obtain correct arithmetic expression by inserting characters + and 1 into this sequence. Both are the same definitions. 

### Valid parenthesis

```cpp

int cnt=0;

for(char ch:s){
	if(ch=='('){
		cnt++;
	}else{
		cnt--;
		if(cnt<0){
			return false;
		}
	}
}

return cnt==0;
```

#### Balance  and prefix balance

The counter we used is called balance and is used in many parenthesis problems. 
For the valid parenthesis the running `balance` should in the end be `0` and at any point of time must never be negative. 

We can also define prefix balance as the ith entry of prefix be the running balance sum at that point. This can be used to find if given subsequence is valid or not.

Regular bracket sequence - 

A bracket sequence S is regular if it satisfies one of the following recursive rules:

- An empty string in `RBS`.
- If A is `RBS`, then `( + A + )` is  `RBS`.
- If A and B are `RBS` then `A+B` is also `RBS`.

### Longest bracket subsequence

The problem here is to remove some brackets from the original string so that resultant sequence is valid.

### Solution

The idea here is that we can not blindly add all the opening and closing brackets. In fact to have a valid sub-sequence we can think of it as deletion of certain characters. Given we will maintain a valid balance of the running sequence. 

When the balance will be `0` and we encounter any `)` then we ignore this kind of chars. 

```cpp
int ans=0;
int bal=0;

for(char ch:s){
	if(ch=='('){
		bal++;
	}else{
		if(bal>0){bal--;ans+=2;}
	}
}
cout<<ans;
```

Now it is true because at every point in time we are maintaining the best correct answer. Essentially by this system what can be wrong is that we may have picked some wrong opening brackets. Wrong closing bracket is never picked because of the balance non negative property. 
Now since the invalid bracket is always the oldest one. we may have to exclude the first some brackets picked this is why we have when closing some bracket only then we add answer. Meaning in the end we might have some opening brackets which we may ignore later on. 

### Longest bracket subsegment

Here we have to find the longest substring such that it holds valid subsequence. 

### Solution

First thing is that the actual string is going to be alternating between valid and invalid expressions. 

`s = V+I+V+I+V` ourtask is to find the longest `|V|` now this means once we find that string has started to be invalid then we should stop at that time only. More importantly once string starts to be invalid aka balance prefix is negative at some point no maater future it will be invalid. This means if prefix is invalid at any point we should discard all till that and start fresh. 

Now we will consider that string to be in form 

`S = I+V+I+V...` and so on. We will consider that first char itself is invalid. Now we will put two kinds of indexes in the stack either index of opening bracket or the index of ending of last invalid subsegment. 

Now we can keep putting the brackets if its opening. However if its closing one two cases can appear. Either the last index is of opeing bracket in that case we will close it and now what will remain is either the opening of prev segment or index of end of illegal segment. Which means that we can calc the length of this valid subsegment. 

Similarlty if this is invalid index this direcltly means what ever remains is invalid and we shall not calc the result. Both of the cases can be handled by one logic only.

```cpp
int longestValidParentheses(string s){
	stack<int> st;
	st.push(-1);
	int n = s.size();
	int ans = 0;
	for(int i=0;i<n;i++){
		if(s[i]=='('){
			st.push(i);
		}else{
			st.pop();
			if(!st.empty()){
				ans = max(ans,i-st.top());
			}else{
				st.push(i);
			}
		}
	}
	return ans;
}
```

However this is more advanced solution and we can rewrite this as explained. The trick is that stack always contains the last invalid `index`. Intially it is `-1`. Then once invalid zone is detected it is reset the last of it. Which is encountered only on the closing brackets. 


> [!NOTE] Note
> In the valid parenthesis only point it which in valid subseq can be detected is at neg balance and that is at closing index. Also while it is true that opening brackets can give us problem but usually with one type of brackets. The closing will always match last `ending` openings and so unmatched opening will always be the earliest onces. This is one of the most important property of valid parenthesis and that is why most solutions use stack. 

#### Data structures  

Stack can be used to follow the last seen first closed property. We can also keep the last seen and match the closing with opening. 

Core of invalid parenthesis - In every parenthisis string if we match and pop out all the valid pairs. Then remaining string is called core of invalid strings and is always in the form  mclosing brackets followed by k opening onces. Note that if that is not the case if opening comes before closing we will instead get valid pair. 

To do the range query of if range `(l,r)`  is regular we can use property of balance that balance follows prefix property. So running balance for any index `i<=R and i>=L` must be positive and total `sum=0`.

#### Catalan numbers

The number of distinct `RBS` of length `2n` is the nth catalan number. 

```
Nth catalan number = 1/(n+1)*(2n C n)
```
#### DP patterns 

Dynamic Programming (DP) is often the go-to approach for parenthesis problems when you need to **count** configurations, find the **longest** valid sequence, or determine the **minimum cost** to fix a string.

##### Total valid RBS

Counting the regular brackets with the n pairs. 

`dp[i]=numberof RBS with i pairs`

Note that every RBS can be written as `(A)B` where A and B are also `RBS`. We can divide the correct number of brackets to `A` and `B`.
So following recurrece follows 

`dp[i] = sum(j=0,i-1)dp[j]*dp[i-1-j]`
`dp[0]=1` Empty string is also empty RBS. This is `O(n^2)`.

#### Longest valid parenthesis

We have seen how to solve longest longest valid parenthesis substring using greedy stack. Now lets see how to solve it using dp.

```
dp[i]=longest valid parenthesis ending at i
dp[i]=0 if s[i]==(
dp[i]=dp[i-2]+2 if s[i-1]==( and s[i]=). Reason being we have completed structure of A() and A can also be RBS

if s[i-1] is closing then we might have some internal RBS leading to this structure. `((A))` clearly we need to see if s[j]=( where j = i-dp[i-1]-1. 
if that is case

dp[i]=dp[i-1]+2+dp[j-1]

```

This completes all the schenarios.

FInally some problems may need to have balance as the state.

```
dp[i][j] = total something till position i ending with balance j.
```

