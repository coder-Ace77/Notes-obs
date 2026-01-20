
---

#### Basic difference array

```cpp
vector<int> arr(n);
vector<int> diff(n+1);
for(int i=1;i<n;i++){
	diff[i]=arr[i]-arr[i-1];
}
// upd
void upd(int i,int j,int x){
	diff[i]+=x;
	diff[j+1]-=x;
}
```

Instead of updating each element within a given range directly, the difference array allows these updates to be done in constant time. A difference array `diff[]` is constructed such that `diff[i] = arr[i] - arr[i-1]`. To increment all elements from index `l` to `r` by a value `val`, we simply do `diff[l] += val` and `diff[r+1] -= val`. When all updates are applied, we can reconstruct the updated array by taking the prefix sum of the difference array.

#### Multiply sums:

Query(l,r) to find - $arr[l]+arr[l+1]*2+arr[l+3]*3+....$

Solution is use two prefix sums:

mulpre[i] = arr[0]+arr[1]*2+arr[3]*2... 
pre[i] = arr[0]+arr[1]+... 

now any query q(l,r) = (mulpre[r]-mulpre[l-1]) - (l-1)*(pre[r]-pre[l-1]);

> [!NOTE]
> This is because mulpre[r] - mulpre[l-1] has all the values in form arr[l]*l + arr[l+1]*l+1.... 
> So we need to subtract the sum of the range by (l-1) times.

Query(l,r) -> $> arr[l]+arr[l+1]*k+arr[l+2]*k^2 +...$


Again create mulpre[i] = arr[0]+arr[1]*k+arr[2]*k^2 +...
$$
mulpre[r] - mulpre[l-1] = arr[l]*k^l+arr[l+1]*k^{l+1}+...
= k^l(arr[l]+arr[l+1]*k+arr[l+2]*k^2...)
$$
So we only need to divide by $k^l$

#### AP additon

Given an array of _N_ integers and _Q_ queries. Initially, all the elements of the array have value 0.
In each query 4 integers _A_, _D_, _L_, _R_ is given, which means perform AP addition

Arr[L]+=A
Arr[L+1]+=A+D
Arr[L+2]+=A+2*D
Arr[L+3]+=A+3*D

Solution:

Arr[i]=A+D×(i−L)
Arr[i]=A−DL+iD
Arr[i]=(A−DL)+iD
Arr[i]= constant for a query +iD+iD  
Arr[i]=C+iD

The `C` is constant across a particular query. So we can make use of partial sums for this this particular constant. Just add `C` at Arr[L] and subtract it from Arr[R+1].

> [!NOTE]
> For the variable part iD , we use another array arr2 and add `D` at arr2[L] and subtract it from arr2[R+1].  
> Finally we iterate through the arrays arr and arr2 and find their prefix sums as pf and pf2. To find the ans[i] we use the relation pf[i]+i×pf2[i], we multiply with `i` for the fact that the variable part is iD. The D part we took care from the prefix sum pf2 but we also need to account for the multiplication with the `i` factor here.



