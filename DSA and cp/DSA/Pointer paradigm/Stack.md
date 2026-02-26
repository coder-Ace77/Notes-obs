
---

Stack is a data structure which allows two operations - 

1. push() - Push an element at the end
2. pop() - Remove the element from the end

A stack is a Last-In, First-Out (LIFO) structure.

This models situations where _you can’t proceed until you’ve finished the most recent unfinished task._ For instance - 

1. Nested parentheses `((()))` — the last open bracket must close first. So the structure of parenthesis is in such a way that the opening bracket comming last has to be finished first. While the bracket comming in earlier will be closed later on. 
2. Undo/Redo — the last action undone is the first restored.
3. Function calls — the last function called must return before the earlier one continues.

So whenever your logic has a **“defer until later”** pattern, or **nested or dependent operations**, a stack is the right choice.

## Uses

### Expression evaluation and syntax checking

When you encounter nested brackets or operators, you need to remember what’s still “open”  like open brackets `(` or incomplete expressions. The last open bracket must close first perfect LIFO logic.

```cpp
Input: "()[]{}"
We push every open bracket onto stack.
When we see a closing bracket, it must match the top of stack.
If all matched → valid.
```

### Monotonic stack

Finding nearest greater/smaller element to left or right in O(n). 

When traversing, we keep a stack of _useful_ elements — those which might be the next greater/smaller for future ones.  
If a new number invalidates the top (e.g., it's greater), we pop until the order is restored.  
The stack maintains a **monotonic structure** — increasing or decreasing.

```cpp
Next Greater Element to the right
nums = [2, 1, 3]

Traverse right to left:
- 3: stack empty → NGE = -1 → push(3)
- 1: top=3>1 → NGE=3 → push(1)
- 2: pop(1) because 1<2, top=3>2 → NGE=3
```

Every element "waits" until a larger one appears. The newest elements are the ones most likely to be invalidated soon.

**Use a stack whenever a problem's logic depends on processing the _most recent_ unresolved item.** 

## Important patterns

### Stack using two queues

We can implement this pattern by maintaining one queue which is the reverse. 
Suppose at all times q1 maintains the reverse of what was entered earlier. And q2 will be empty. If we can show how will we be using the q2 to maintain the reverse order in q1 then we are done.

Idea is to first insert element into `q2` and then push all the elements of q1 to q2 and swap. Since q1 was already in rev order these operations are such that an element was inserted in front of other elements.

```cpp
class MyStack {
    queue<int> q1, q2;

public:
    void push(int x) {
        q2.push(x);
        while (!q1.empty()) {
            q2.push(q1.front());
            q1.pop();
        }
        swap(q1, q2);
    }

    int pop() {
        int x = q1.front();
        q1.pop();
        return x;
    }

    int top() {
        return q1.front();
    }

    bool empty() {
        return q1.empty();
    }
};
```

### Min stack

Design a stack that supports push, pop, top, and retrieving the minimum element in constant time. Observe that minimum/ maximum / sum will be following the canonical structure. So, 

`min(stack[0:i]) = min(min(stack[0:i-1]),stack[i])`

So it is really easy to implement the stack using extra space. Maintaining two stacks one for minimum and other for the normal. 

```cpp
class MinStack {
    stack<int> st;
    stack<int> minSt;
public:
    void push(int val) {
        st.push(val);
        if (minSt.empty() || val <= minSt.top()) {
            minSt.push(val);
        }
    }

    void pop() {
        if (st.top() == minSt.top()) {
            minSt.pop();
        }
        st.pop();
    }

    int top() {
        return st.top();
    }

    int getMin() {
        return minSt.top();
    }
};
```

But it turns out that we can implement this kind of stack without using extra space. 
Here we maintain the minimum in the separate variable and stack maintains the `ele - curr_min`. 
Suppose

- `val` = value to push
- `minVal` = current minimum before pushing

While pushing
- If the stack is **empty**, push `val` and set `minVal = val`.
- If `val >= minVal`, just push `val`.
- If `val < minVal`, we push **encoded value** `2*val - minVal`.  
    This value is always **less than** the new `minVal`, so we can detect it later.  
    Then update `minVal = val`.

When popping
- If the top element `>= minVal`, just pop normally.
- If the top element `< minVal`, it’s an encoded value → we must **restore** the previous minimum:

```ini
previousMin = 2*minVal - top
```

```cpp
class MinStack {
    stack<long long> st;
    long long minVal;
public:
    void push(long long val) {
        if (st.empty()) {
            st.push(val);
            minVal = val;
        } else if (val >= minVal) {
            st.push(val);
        } else {
            // encode value
            st.push(2 * val - minVal);
            minVal = val;
        }
    }

    void pop() {
        if (st.empty()) return;
        long long topVal = st.top();
        st.pop();
        if (topVal < minVal) {
            // restore previous min
            minVal = 2 * minVal - topVal;
        }
    }

    long long top() {
        if (st.empty()) return -1;
        long long topVal = st.top();
        return (topVal >= minVal) ? topVal : minVal;
    }

    long long getMin() {
        return minVal;
    }
};
```

### Monotonic stack

A **monotonic stack** is a stack that maintains its elements in a **sorted order** (either increasing or decreasing) **as you iterate** through an array.

- Find the **next greater element** (NGE)
- Find the **next smaller element** (NSE)
- Find **previous greater/smaller** elements
- Solve range-based problems (like “for each element, find boundary to the left/right where a condition breaks”)

When processing elements one by one (usually left to right):
- We **remove elements** from the stack that are **no longer useful** for future comparisons (e.g., smaller ones if we want next greater element).
- The stack thus always keeps a **monotonic property**:
    - **Monotonic increasing stack** → top of stack is the smallest so far
    - **Monotonic decreasing stack** → top of stack is the largest so far

Template
```cpp
vector<int> nextGreaterElements(vector<int>& nums) {
    int n = nums.size();
    vector<int> ans(n, -1);
    stack<int> st; 

    for (int i = 0; i < n; i++) {
        while (!st.empty() && nums[st.top()] < nums[i]) {
            ans[st.top()] = nums[i];
            st.pop();
        }
        st.push(i);
    }
    return ans;
}
```

More general template

```cpp
vector<int> mono(vector<int> &arr,bool next,bool greater){
        int n = arr.size();
        vector<int> ans(n);
        int start = (next?0:n-1);
        int end = (next?n:-1);
        int step = (next?1:-1);
        stack<int> st;
        for(int i=start;i!=end;i+=step){
            if(greater){
                while(!st.empty() && arr[st.top()]<=arr[i]){
                    st.pop();
                }
            }else{
                while(!st.empty() && arr[st.top()]>=arr[i]){
                    st.pop();
                }
            }
            if(!st.empty())ans[i]=st.top();
            else ans[i]=-1;
            st.push(i);
        }
        return ans;
    }
```

Sometimes we can use contribution technique where we calcalate answer by finding in how many valid subsequences/subarray an element can be found out. 

### Lexicographical ordering and monotonic stack

Sometimes we may need to solve and find the smallest lexicographical / largest lexicographical strings. Lexicographical case has a special property that the ordering between two strings depends entirely on the first point of difference in strings. The smaller string will have this first character smaller. This sole observation is usually enough to have greedy choices. 

Now in these problems the main idea is to assume that given till `ith` step certain algo `A` was able to create the most optimal string. Now what about the step `(i+1)th` can you create some strategy such that answer remains optimal even after `(i+1)th` step. Now in many cases for this greedy approach to work we use stack as it easily allows to remove the most recent additions. Note that if it was guaranted that answer of `i+1th` step will be the expansion/addition of answer of ith step then there is no need of stack. Rather simple addition works fine. Stack is needed in case `A` needs to undo some chars in the optimal string.

#### Proving

Usually since the strategy `A` it is greedy and is required to be proved. We follow this general pattern of proof by contradiction. 

Suppose that optimal string is `x` and our algo`A` produced `y` then at first point of difference `ith`. Our sting has bigger character and then proof depends on the strategy. 

#### Removing duplicate letters

Given a string `s`, remove duplicate letters so that every letter appears once and only once. You must make sure your result is lexicographically smallest among all possible results.

#### Solution

Assume that our stack maintains the optimal ordering now we need to come up with a greedy algo. Assme that string uptil that say ith point is `ans` now if the last character of ans say `c` is larger than `s[i]` and if it comes after i also then we can easily remove it. Note that `ans-c+s[i]` is lexicographically smaller than `ans+s[i]`. However note that we also have restriction to use `1` char in the answer. Now if this char `s[i]` was already in the string earlier. Then it means all the char after it can not removed. Why? Reason being we would have removed this is next was smaller as `s[i]` is present here anyways. This means we should not add this `s[i]`. So we also need to track all the elements already present in the string.

Now can `s[i]` be removed in some later step the answer is yes. Reason being we may encounter some `char b<s[i]` which could empty it.

```cpp
string removeDuplicateLetters(string s) {
	map<char,int> mp;
	vector<int> used(26,1);
	for(auto ch:s){
		mp[ch]++;
		used[ch-'a']=0;
	}
	string st;
	for(char ch:s){
		mp[ch]--;
		if(used[ch-'a'])continue;
		while(!st.empty() && st.back()>ch && mp[st.back()]>0){
			used[st.back()-'a']=0;
			st.pop_back();
		}
		st+=ch;
		used[ch-'a']=1;
	}
	return st;
}
```

We have used map to see if this char comes after it or not. 

Now proof of correctness - By contradiction assume that `ans`is not optimal ans correct ans is `opt` now for the first point of diff `ith` `ans[i]>opt[i]`. Two cases either `ans` has more deleted chars or opt has more deleted chars before i. If ans has more deleted chars the that means we deleted some smaller char and then end up with larger char. But notice with our algo this kind of deletion can never happen. Deleting always happen if smaller char can be pushed towards left. 

Now for the second case opt has more deleted chars. Here that means we must have larger char which we must have been able to delete by upcoming smaller one. This completes the proof. 

Some pointers are one can think of these problems as - 

- Shifting of smaller chars to the left. 
- Deleting of larger chars. 

To solve such problems one has to come out of the thinking of back and and forth. Clear understanding and implementation and observations are very crucial. 

### Removing k digits

Given string num representing a non-negative integer `num`, and an integer `k`, return _the smallest possible integer after removing_ `k` _digits from_ `num`.

#### Solution 

Smallest possible integer is actually same as the smallest lexicographical string. Now again observe that we can actually remove any digit such that there is smaller digit upnext. Since we are talking about the lexicographically smallest. This greedy choice is best if applided left to right. 

Observe that we should the digit if prev digit is larger than current one which makes it monotonic increasing stack. 

```cpp
string removeKdigits(string num, int k) {  
	string st="";
	int n = num.size();
	for(int i=0;i<n;i++){
		while(!st.empty() && k>0 && st.back()>num[i]){
			st.pop_back();
			k--;
		}
		st.push_back(num[i]);
	}
	
	while (k > 0 && !st.empty()) {
		st.pop_back();
		k--;
	}
	int i = 0;
	while (i < st.size() && st[i] == '0') i++;
	string res = st.substr(i);
	return res.empty() ? "0" : res;
}
```

## Polish notation

Polish notation (invented by **Jan Łukasiewicz**, 1920s) is a way to **write arithmetic expressions without parentheses**.]

| Type                         | Example for `(3 + 4) * 5` | Evaluation order |
| ---------------------------- | ------------------------- | ---------------- |
| **Prefix (Polish)**          | `* + 3 4 5`               | right-to-left    |
| **Postfix (Reverse Polish)** | `3 4 + 5 *`               | left-to-right    |
Both remove the need for parentheses because **operator order** is fixed by position.
Stacks are **LIFO**, matching **expression evaluation order**:

- When you see an operator, you apply it to **most recently seen operands** (top of stack).
- When you see a number, you push it for future use.

That’s exactly what postfix or prefix evaluation needs — you process operands first, then apply operators when you have enough values.

```cpp
stack<long long> st;
for (auto& token : tokens) {
	if (token == "+" || token == "-" || token == "*" || token == "/") {
		long long b = st.top(); st.pop();
		long long a = st.top(); st.pop();
		if (token == "+") st.push(a + b);
		else if (token == "-") st.push(a - b);
		else if (token == "*") st.push(a * b);
		else st.push(a / b);
	} else {
		st.push(stoll(token));
	}
}
return st.top();
```

For polish notitation evaluation is same but we make it so that traversal is done from right to left

```cpp
stack<long long> st;
for (int i = tokens.size() - 1; i >= 0; i--) {
	string t = tokens[i];
	if (t == "+" || t == "-" || t == "*" || t == "/") {
		long long a = st.top(); st.pop();
		long long b = st.top(); st.pop();
		if (t == "+") st.push(a + b);
		else if (t == "-") st.push(a - b);
		else if (t == "*") st.push(a * b);
		else st.push(a / b);
	} else st.push(stoll(t));
}
```

Stack can also be used to convert infix to postfix. 

1. Initialize:
    - an empty stack `st` for operators.
    - an empty string or list `output` for result.
2. For each token `t` in expression:
    - **If `t` is operand (A–Z, a–z, 0–9)** → append to `output`.
    - **If `t` is '('** → push to stack.
    - **If `t` is ')'** → pop until '(' is found.
    - **If `t` is operator (`+, -, *, /, ^`)**:
        - while stack is not empty **and** top of stack has **higher or equal precedence** than current operator, pop and append to output.
        - then push current operator to stack.
3. After traversing expression, pop all remaining operators from stack and append to output.

```cpp

int precedence(char op) {
    if (op == '^') return 3;
    if (op == '*' || op == '/') return 2;
    if (op == '+' || op == '-') return 1;
    return 0;
}

bool isLeftAssociative(char op) {
    return (op != '^');
}

string infixToPostfix(string s) {
    stack<char> st;
    string output;

    for (char c : s) {
        if (isalnum(c)) {
            output += c;
        } 
        else if (c == '(') {
            st.push(c);
        } 
        else if (c == ')') {
            while (!st.empty() && st.top() != '(') {
                output += st.top();
                st.pop();
            }
            st.pop(); // remove '('
        } 
        else { // operator
            while (!st.empty() && precedence(st.top()) > 0) {
                if ((isLeftAssociative(c) && precedence(c) <= precedence(st.top())) ||
                    (!isLeftAssociative(c) && precedence(c) < precedence(st.top()))) {
                    output += st.top();
                    st.pop();
                } else break;
            }
            st.push(c);
        }
    }

    while (!st.empty()) {
        output += st.top();
        st.pop();
    }
    return output;
}
```

Sometimes stacks can also be used on string operations. One pattern is bracket matching. However there are other patterns as well.

1. Remving/collapsing adjacent characters. When characters cancel each other (like duplicates or opposite cases), the most recent one needs to be removed first — a perfect LIFO pattern.

### Longest valid parenthesis

Given a string `s` consisting of only `'('` and `')'`,  
find the **length of the longest valid (well-formed) parentheses substring**.

This is an example of problem where we have to store two different kinds of indexes in the array. Note that this pattern is very prelevant in the industry. 

The idea is that we will keep two kinds of index in stack- indexes where where we have `(` opening bracket and second where the last substring ended. 

We push indexes of `'('`.  
When we see `')'`, we pop a `'('` index if available — that means we found a match.
If the stack becomes empty, that means **this point becomes a new “base” for the next valid substring**.
So, **stack stores the last unmatched ')' or '(' position**, helping us calculate valid lengths.

```cpp
int longestValidParentheses(string s) {
    stack<int> st;
    st.push(-1);
    int maxLen = 0;

    for (int i = 0; i < s.size(); i++) {
        if (s[i] == '(')
            st.push(i);
        else {
            st.pop();
            if (st.empty())
                st.push(i); 
            else
                maxLen = max(maxLen, i - st.top());
        }
    }
    return maxLen;
}
```

Finally stack can be used to optimise the dfs by replacing function calls with the stack. It is easy to do preorder or postorder as we just just need to push the values in the stack however situation changes drastically when doing inorder. So here we first push all the left elements into the stack and then pop out the smallest element from the stack. 

```cpp
vector<int> preorderTraversal(TreeNode* root) {
    vector<int> result;
    if (!root) return result;
    stack<TreeNode*> st;
    st.push(root);
    while (!st.empty()) {
        TreeNode* node = st.top();
        st.pop();
        result.push_back(node->val); // Visit node
        // Push right first so left is processed first
        if (node->right) st.push(node->right);
        if (node->left)  st.push(node->left);
    }

    return result;
}
```  

inorder

```cpp
vector<int> inorderTraversal(TreeNode* root) {
    vector<int> result;
    stack<TreeNode*> st;
    TreeNode* curr = root;
    while (curr || !st.empty()) {
        // Reach the leftmost node
        while (curr) {
            st.push(curr);
            curr = curr->left;
        }
        // Now process the top 
        curr = st.top();
        st.pop();
        result.push_back(curr->val);
        // Move to the right subtree
        curr = curr->right;
    }
    return result;
}
```