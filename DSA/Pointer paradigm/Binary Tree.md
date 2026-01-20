

---

Binary trees are the data structure that have two pointer left and right. Basic questions are follows and are self explanatory--
The basic idea about binary trees is about traversal. In dfs recursive traversal there are three main variations - 

preorder, postorder and inorder--
in preorder -> current node then go left then right node
in inorder -> first left node then current node  then right
potorder -> right node then current then left

```cpp
void inorder(TreeNode* root) {
    if (!root) return;
    inorder(root->left);
    cout << root->val << " ";
    inorder(root->right);
}
```

Now the created sequence has some special properties - 

1. In preorder traversal root of the subtree is always the first val in the sequence.
2. Knowing **preorder + inorder** → allows **unique reconstruction** of the tree.
3. The **last element** in the postorder sequence is always the **root** of the tree.
4. Reflects **bottom-up structure** — you visit all descendants before the parent.
5. Reflects **tree structure in top-down order** — you visit each node before its subtrees.
6. **Each subtree corresponds to a continuous segment in the inorder traversal.**
7. We can split the inorder array into left and right subtrees easily during reconstruction.  (Because all left subtree nodes come before the root, and all right ones come after.)
8. In Preorder, subtrees usually form contiguous blocks,  but we can’t isolate left/right without knowing the inorder boundaries.
9. It is not possible to create tree if given preorder and postorder traversal.
#### Max depth:

Given the `root` of a binary tree, return _its maximum depth_.

A binary tree's **maximum depth** is the number of nodes along the longest path from the root node down to the farthest leaf node.

```cpp
int maxDepth(TreeNode* root) {
	if(root==nullptr)return 0;
	return 1+max(maxDepth(root->left),maxDepth(root->right));
}
```

#### Same tree:

Given the roots of two binary trees `p` and `q`, write a function to check if they are the same or not.
Two binary trees are considered the same if they are structurally identical, and the nodes have the same value.

```cpp
bool isSameTree(TreeNode* p, TreeNode* q) {
	if(p==nullptr && q==nullptr)return true;
	if(p==nullptr || q==nullptr)return false;
	if(p->val==q->val)return isSameTree(p->left,q->left) && isSameTree(p->right,q->right);
	return false;
}
```

#### Invert tree:

Given the `root` of a binary tree, invert the tree, and return _its root_.

```cpp
TreeNode* invertTree(TreeNode* root){
	if(root==nullptr)return root;
	invertTree(root->left);
	invertTree(root->right);
	swap(root->left,root->right);
	return root;
}
```

#### Symmetric tree:

Given the `root` of a binary tree, _check whether it is a mirror of itself_ (i.e., symmetric around its center).

```cpp
bool rec(TreeNode* a,TreeNode* b){
	if(a==nullptr && b==nullptr)return true;
	if(a==nullptr || b==nullptr)return false;
	if(a->val==b->val)return rec(a->left,b->right) && rec(a->right,b->left);
	return false;
}

bool isSymmetric(TreeNode* root) {
	if(root==nullptr)return true;
	return rec(root->left,root->right);
}
```

#### Contruct binary tree:

Given two integer arrays `inorder` and `postorder` where `inorder` is the inorder traversal of a binary tree and `postorder` is the postorder traversal of the same tree, construct and return _the binary tree_.

In the solution there are couple of caveat first is that all the values of the node hould be unique. Because we want to find the value in the inorder traversal. Second is that we can speed up solution by hashmapping the value with index and it will make solution O(n).

The given solution i based  on the idea that in preroder root node is the first node we have revere pre and now its last node for efficient remove 

```cpp
TreeNode* rec(int l,int r,vector<int> &in,vector<int> &pre){
	if(l>r || post.size()==0)return nullptr;
	int val = pre.back();
	pre.pop_back();
	int ind = 0;
	for(int i=l;i<=r;i++){
		if(in[i]==val){
			ind=i;
			break;
		}
	}
	TreeNode* tr = new TreeNode(val);
	tr->left = rec(l,ind-1,in,pre);
	tr->right = rec(ind+1,r,in,pre);
	return tr;
}

TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
	reverse(preorder.begin(),preorder.end());
	int n = inorder.size();
	return rec(0,n-1,inorder,preorder);
}
```

#### Populating the next pointer in binary tree:

Populate each next pointer to point to its next right node. If there is no next right node, the next pointer should be set to `NULL`.
Initially, all next pointers are set to `NULL`.

Very much easy to solve using bfs.

```cpp
class Solution {
public:
    Node* connect(Node* root) {
        queue<Node*> q;
        if(root)q.push(root);
        while(!q.empty()){
            int sz = q.size();
            Node* prev=nullptr; 
            for(int i=0;i<sz;i++){
                Node* node = q.front();
                q.pop();
                if(prev==nullptr)prev=node;
                else{
                    prev->next=node;
                    prev = node;
                }
                if(node->left)q.push(node->left);
                if(node->right)q.push(node->right);
            }
        }
        return root;
    }
};
```

### Flatten to a linked list:

Given the `root` of a binary tree, flatten the tree into a "linked list":

- The "linked list" should use the same `TreeNode` class where the `right` child pointer points to the next node in the list and the `left` child pointer is always `null`.
- The "linked list" should be in the same order as a [**pre-order** **traversal**](https://en.wikipedia.org/wiki/Tree_traversal#Pre-order,_NLR) of the binary tree.

Note that we can easily solve the problem if it is allowed to use extra space.

```cpp
class Solution {
public:
    void rec(TreeNode* root,vector<TreeNode*> &v){
        if(!root)return;
        v.push_back(root);
        rec(root->left,v);
        rec(root->right,v);
    }

    void flatten(TreeNode* root) {
        if(root==nullptr)return;
        vector<TreeNode*> v;
        rec(root,v);
        for(int i=0;i<v.size()-1;i++){
            v[i]->left=nullptr;
            v[i]->right=nullptr;
            v[i]->right=v[i+1];
        }
    
};
```

However problem is not so easy if it is not allowed to use extra space. Now observe the following idea
![[Pasted image 20251009093838.png]]

Observe that to make the necessary linkages we have to connect the rightmost node of left subtree to the first node of right subtree. Also right pointer of root must be changed to the root of left subtree as shown in the diagram.

```cpp
class Solution {
public:
    void flatten(TreeNode* root) {
        TreeNode* curr = root;
        while(curr!=nullptr){
            if(curr->left!=nullptr){
                TreeNode* prev = curr->left;
                while(prev->right){
                    prev = prev->right;
                }
                prev->right = curr->right;
                curr->right = curr->left;
                curr->left = nullptr;
            }
            curr = curr->right;
        }
    }
};
```

### Binary Tree maximum path sum:

A **path** in a binary tree is a sequence of nodes where each pair of adjacent nodes in the sequence has an edge connecting them. A node can only appear in the sequence **at most once**. Note that the path does not need to pass through the root.

The idea is to find the max path sum till the leaf node and then combine the left and right to find the value to max.

```cpp
class Solution{
public:
    int rec(TreeNode* root,int& res){
        if(!root)return 0;
        int left = max(0,rec(root->left,res));
        int right = max(0,rec(root->right,res));
        res = max(res,left+right+root->val);
        return max(left,right)+root->val;
    }

    int maxPathSum(TreeNode* root){
        int res = root->val;
        rec(root,res);
        return res;
    }
};
```

### Lowest common ancestor of binary tree:

First if any node is nullptr or it is equal to p or q then we can consider returning from here reason being answer can not lie below it. So we can prune down.

Note that the solution is incorrect if the question does not garenatee that both nodes will exist because then we will return some root but solution may or may not exist. 

```cpp
TreeNode* rec(TreeNode* root,TreeNode* p,TreeNode* q){
	if(!root || root==p || root==q)return root;
	TreeNode* left = rec(root->left,p,q);
	TreeNode* right = rec(root->right,p,q);
	if(left && right)return root;
	return left?left:right;
}
```

### Binary search tree iterator:(imp)

The idea is to give a data structrue with support of two functions. 
1. next() - it returns a element in the binary search tree order and also moves to new one.
2. hasNext() - it returns true or false based on if iteration is done or not.

In all it is used for implementation of traversal. 

Now problem is just inorder traversal. But we want to have smaller space complexity meaning `O(h)`
Now how to achieve this --

We maintain two things one a stack and another one a curr variable. 
Initially curr points to the root. Now when next is called we move to the extreme left of the tree and put the elements in the stack.  At the end we get the top most element of stack which represents the left most element in the stack now. Now what should be the next element it should be the right of next. Now two cases can occur if right is null then next time new element from stack will be taken out however if not then curr's subtree will be processed next time.

```cpp
class BSTIterator {
public:
    TreeNode* curr;
    stack<TreeNode*> st;
    BSTIterator(TreeNode* root){
        curr = root;
    }
    int next(){
        while(curr){
            st.push(curr);
            curr = curr->left;
        }
        TreeNode* x = st.top();st.pop();
        curr = x->right;
        return x->val;
    }

    bool hasNext(){
        return st.size()>0 || curr!=nullptr;
    }
};
```


### BFS in binary tree

Just like normal bst we can also do bfs wise traversal maintaining some datastructure for differnet problems for different solutions. 

### Right side view 

Given the `root` of a binary tree, imagine yourself standing on the **right side** of it, return _the values of the nodes you can see ordered from top to bottom_.

### Solution

```cpp
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        queue<TreeNode*> q;
        if(root)q.push(root);
        vector<int> ans;
        while(!q.empty()){
            int sz = q.size();
            for(int i=0;i<sz;i++){
                TreeNode* tr = q.front();
                q.pop();
                if(i==0)ans.push_back(tr->val); // pushing and updating last node
                else ans.back()=tr->val;
                if(tr->left)q.push(tr->left);
                if(tr->right)q.push(tr->right);
            }
        }
        return ans;
    }
};
```


---


## Binary search tree:

BST is a tree where each node follows binary search property - 
1. All nodes in the left subtree are less than node's value
2. All nodes in the right subtree are greater than node's value

It follows that grestest node in the left subtree is smaller than smallest node in the right subtree.

#### Basic operations:

There are three basic operations in a bst search, insert and delete. 

Idea is simple since each node follows bst property left node is smaller than right node meaning the key to search will either be found on left or on the right subtree. The time and space complexity is order of depth `O(h)`.

```cpp
bool search(TreeNode* root, int key) {
    if (!root) return false;
    if (root->val == key) return true;
    if (key < root->val) return search(root->left, key);
    return search(root->right, key);
}
```

insertion is again simple since there will be a defined position of a key we can search the postion first and then update. To simplyfy the operations we create a new node and reassign the pointers 

eg -- `root->left = insert(root->left,key)`

This works becasue if new node is not created we are returning older node. However we are returning new node if key is inserted.

```cpp
TreeNode* insert(TreeNode* root, int key) {
    if (!root) return new TreeNode(key);
    if (key < root->val)
        root->left = insert(root->left, key);
    else if (key > root->val)
        root->right = insert(root->right, key);
    return root;
}
```

Deleting a node is typical there can be three schenarios:

1. Node has no child - delete directly 
2. Node has one child - replace with the child
3. Node has two child - replace this node with the inorder successor

There are three kinds of patterns of traversal and each one of them has different use cases. For instance inorder is used to give out the sorted order of BST and post order is used to delete tree.

| Traversal Type | Order               | Output Pattern           |
| -------------- | ------------------- | ------------------------ |
| **Inorder**    | Left → Root → Right | Sorted order in BST      |
| **Preorder**   | Root → Left → Right | Used to clone/create BST |
| **Postorder**  | Left → Right → Root | Used to delete tree      |
```cpp
TreeNode* findMin(TreeNode* root) {
    while (root->left) root = root->left;
    return root;
}

TreeNode* deleteNode(TreeNode* root, int key) {
    if (!root) return nullptr;
    if (key < root->val) root->left = deleteNode(root->left, key);
    else if (key > root->val) root->right = deleteNode(root->right, key);
    else {
        if (!root->left) return root->right;
        if (!root->right) return root->left;
        TreeNode* minNode = findMin(root->right);
        root->val = minNode->val;
        root->right = deleteNode(root->right, minNode->val);
    }
    return root;
}
```

Diameter:

rec() returns the max depth of any node and ans keeps track of diamter of tree. diameter stacks the edges so its value edges = nodes on path -1

```java
private int ans=0;

public int diameterOfBinaryTree(TreeNode root){
	rec(root);
	return ans;
}

public int rec(TreeNode root){
	
	if(root==null) return 0;
	int lft = rec(root.left);	
	int rgt = rec(root.right);

	ans = Math.max(ans,lft+rgt);
	return 1+Math.max(lft,rgt);
}
```

inorder Traversal -> left,print(curr),right
Preorder Traversal -> print(curr),left,right
Postorder Traversal -> left,right,print(curr)

#### BST

Validating bst enough to alidate evety node. For this we can return max_value and min_value of every node recursively. 
More clever approach: Idea is to maintain the valid range of values for a given node. 
Obvously we will start with inf value of min=-inf and max=inf. If node is not the range then bst is invalid.

When we will go more in depth ranges will shrunk on both sides.

```java
public boolean check(TreeNode root,int mn,int mx){
	if(root==null){
		return true;
	}
	
	if(root.val<mn || root.val>mx)return false;
	return check(root.left,mn,(int)root.val-1) && check(root.right,(int)root.val+1,mx);
}
```

LCA in BST:

A general LCA in binary tree requires us to do level wise traversal. We can however solve the lca in bst more wisely.

If current node value is more than both nodes then lca must be in left. 
If current node value is less than both nodes than lca must be on right.
Current node is the answer if its value is larger than one node and smaller than other node.

```java
public TreeNode lowestCommonAncestorInBST(TreeNode root, TreeNode p, TreeNode q){
	if(root==null){
		return null;
	}
	if(root.val>p.val && root.val>q.val){
		return lowestCommonAncestorInBST(root.left,p,q);
	}
	if(root.val<p.val && root.val<q.val){
		return lowestCommonAncestorInBST(root.right,p,q);
	}
	return root;
}
```

Inorder successor:

```java
public int inorderSuccessor(TreeNode root,TreeNode p){
	succ = null;
	while(root!=null){
		if(p.val < root.val){
			succ = root;
			root = root.left;
		}else{
			root = root.right;
		}
	}
	if(succ==null)return -1;
	return (int)succ.val;
}
```

if current node has value more than target then we go left but this is possible solution.
Similarly if current node has value more than target then we go right.

Binary tree from inorder and postorder:

Done recursively->

we have start end index of current range. 
- Start from the last element in postorder (root).
- Find this root in inorder to divide into left and right subtrees.
- Recurse:
    - Build **right subtree first**, because in postorder, right subtree comes before left subtree (when going backward).
    - Then build **left subtree**.


```java
private TreeNode helper(int[] inorder,int[] postorder,int st,int end){
	if(st>end)return null;
	int val = postorder[idx--];
	TreeNode root = new TreeNode(val);
	int i = mp.get(val);
	root.right = helper(inorder,postorder,i+1,end);
	root.left = helper(inorder,postorder,st,i-1);
	return root;
}
```

Kth largest in BST:

We go to right subtree first and check there. If count was less than k that means there were less than k elements in right subtree.

Now we can move to left subtree. Here either this is kth largest node or we should find it in left subtree. 

```java
private int dfs(TreeNode root,int k){
	if(root==null)return -1;	
	int res = dfs(root.right,k);
	if(res!=-1)return res;
	cnt++;
	if(cnt==k){
		return (int)root.val;
	}
	return dfs(root.left,k);
}
```


