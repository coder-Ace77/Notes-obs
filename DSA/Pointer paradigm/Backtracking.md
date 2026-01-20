
---

Backtracking is fancy word for recursive all possible combinations recursive search solution of problems.
In almost all the search problems following code remains valid

```cpp
curr = {current state};
void rec(int position){
	if(position==end){
		if(check(curr)){
			// do somthing like count or put into all solutions
		}
		return;
	}
	
	for(auto choice:all_choices){
		if(allowed(choice,curr)){
			curr.add(choice);
			rec(position+1);
			curr.remove(choice);
		}
	}
}
```

Depending upon the circumstances curr can be any data structure or simple array or even bitmask. The only thing matters is that it should be able store the current state and must be able to manipulate the state. 
Secondly we have to implement the check function. This check function should be able to validate current solution based on state. Sometimes while  selecting soe possible choice it is possible to eleminate certain choices. This step is called pruning and can drastically improve the solution. 

Finally sometimes we are required to maintain the combination meaning choices like `[1,2,3]` and `[3,2,1]` are considered same. At this point we come up with a increasing order argument that if we try to put the choices in some order like increasing then automatically 3 can never come after 1 and so we will get only one combination. In the cases where multiple items can repeat but thier permutation is not asked for example this set of choices `[1,1,1]` So we add the choices at the given step. 

## Problems

1. Given a string containing digits from `2-9` inclusive, return all possible letter combinations that the number could represent. Return the answer in **any order**.

	A mapping of digits to letters (just like on the telephone buttons) is given below. Note that 1 does not map to any letters.

Solution:

```cpp
vector<string> letterCombinations(string digits) {
	vector<string> ans;
	string curr="";
	map<char,string> mp;
	mp['2']="abc";
	mp['3']="def";
	mp['4']="ghi";
	mp['5']="jkl";
	mp['6']="mno";
	mp['7']="pqrs";
	mp['8']="tuv";
	mp['9']="wxyz";
	
	function<void(int)> rec = [&](int index){
		if(index==digits.size()){
			if(curr.size()>0)ans.push_back(curr);
		}
		for(auto x:mp[digits[index]]){
			curr.push_back(x);
			rec(index+1);
			curr.pop_back();
		}
	};
	rec(0);
	return ans;
}
```


2. Given two integers `n` and `k`, return _all possible combinations of_ `k` _numbers chosen from the range_ `[1, n]`.You may return the answer in **any order**.

Solution:

```cpp
vector<vector<int>> combine(int n, int k) {

	vector<vector<int>> ans;
	vector<int> temp;
	function<void(int)> rec = [&](int index){
		if(index==k){
			ans.push_back(temp);
			return;
		}
		int prev = (temp.size()>0?temp.back()+1:1);
		for(int i=prev;i<=n;i++){
			temp.push_back(i);
			rec(index+1);
			temp.pop_back();
		}
	};
	rec(0);
	return ans;
}
```
