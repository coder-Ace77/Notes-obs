
---

Tries is a tree like datastructure where each node can point to maximum of `characters ch` of vocabulary.  A **Trie (prefix tree)** is a tree-based data structure used for **storing strings** efficiently, especially useful for **prefix-based operations** (like autocomplete, dictionary search, etc.).

Each node represents a **character**, and a path from root to a terminal node represents a **word**.  
Each node also keeps a flag indicating if it’s the **end of a word**.

```cpp
struct TrieNode {
    TrieNode* child[26];
    bool end;
    TrieNode() {
        end = false;
        for (int i = 0; i < 26; i++) child[i] = nullptr;
    }
};
```

## Recursive implementation

```cpp
void insertRec(TrieNode* node, const string& word, int i = 0) {
    if (i == word.size()) {
        node->end = true;
        return;
    }
    int idx = word[i] - 'a';
    if (!node->child[idx])
        node->child[idx] = new TrieNode();
    insertRec(node->child[idx], word, i + 1);
}
```

search

```cpp
bool searchRec(TrieNode* node, const string& word, int i = 0) {
    if (!node) return false;
    if (i == word.size()) return node->end;
    int idx = word[i] - 'a';
    return searchRec(node->child[idx], word, i + 1);
}
```

delete 

```cpp
bool deleteRec(TrieNode* node, const string& word, int i = 0) {
    if (!node) return false;
    if (i == word.size()) {
        if (!node->end) return false;
        node->end = false;
        for (int j = 0; j < 26; j++)
            if (node->child[j]) return false; // still has children
        return true; // delete this node
    }
    int idx = word[i] - 'a';
    if (deleteRec(node->child[idx], word, i + 1)) {
        delete node->child[idx];
        node->child[idx] = nullptr;
        if (!node->end)
            for (int j = 0; j < 26; j++)
                if (node->child[j]) return false;
        return true;
    }
    return false;
}
```

Sometimes we can maintian the additional data with each node for example number of nodes ending at certain point. Maintaining this kind of datastrucrture is often easy to maintain and use in problems. 

## Iterative

```cpp
void insertIter(TrieNode* root, const string& word) {
    TrieNode* cur = root;
    for (char c : word) {
        int idx = c - 'a';
        if (!cur->child[idx])
            cur->child[idx] = new TrieNode();
        cur = cur->child[idx];
    }
    cur->end = true;
}
```

```cpp
bool searchIter(TrieNode* root, const string& word) {
    TrieNode* cur = root;
    for (char c : word) {
        int idx = c - 'a';
        if (!cur->child[idx]) return false;
        cur = cur->child[idx];
    }
    return cur->end;
}
```

## Usage of tries in xor operations

Tries, or prefix trees, play a powerful role in bit manipulation problems, especially when dealing with binary representations of numbers. In this context, a trie is typically a binary tree where each node represents a bit — either `0` or `1`. This structure allows efficient storage and querying of numbers at the bit level, enabling solutions to problems involving maximum XOR pairs, subset XOR, and related bitwise operations.

The core idea is to insert each number’s binary form into the trie, where the most significant bit (MSB) is stored near the root and the least significant bit (LSB) near the leaf.

y using a binary trie, one can efficiently search for the number that gives the best XOR result with the current number. As you traverse each bit from MSB to LSB, you always try to move to the branch representing the opposite bit (i.e., if the current bit is `0`, you prefer to go to the `1` branch) because `1 ^ 0 = 1` gives a higher XOR. If such a branch doesn’t exist, you move along the same bit branch. This greedy approach, enabled by the trie structure, allows computation of maximum XOR in **O(n * log M)** time, where `M` is the maximum possible number value (and `log M` is the number of bits needed to represent it), which is far more efficient than the brute-force **O(n²)** approach.

```cpp
int search(int n){
	Node* curr = root;
	int ans=0;
	for(int b=31;b>=0;b--){
		int bit = ((n>>b)&1);
		if(curr->links[1-bit]==nullptr){
			curr = curr->links[bit];
		}else{
			curr = curr->links[1-bit];
			ans|=(1<<b);
		}
	}
	return ans;
}
```

One thing to node about trie is that don't visualize the trie node as holding some character. Always try to think in terms of paths and linkages. 

### Problems

One good problem can be to enable wild card matching. For example `.` can match any character so how do one support that. We can implement recursive solution to do that. When we reach any `.` we can recursively check any of the branches down. 
In the worst case this will have time complexity of all going through all nodes of tree. However if there is restirction of atmost 1 `.` Then it will be almost same complexity. 

```cpp
bool search(int index,string &word,Node* curr){
	int n = word.size();
	if(index==n)return curr!=nullptr && curr->isEnding;
	if(word[index]=='.'){
		for(int i=0;i<26;i++){
			if(curr->links[i]!=nullptr){
				if(search(index+1,word,curr->links[i]))return true;
			}	
		}
		return false;	
	}else{
		if(curr->links[word[index]-'a']==nullptr)return false;	
		else return search(index+1,word,curr->links[word[index]-'a']);
	}
	return false;
}
```