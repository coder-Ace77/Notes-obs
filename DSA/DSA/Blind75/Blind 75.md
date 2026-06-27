
---

## Non overlapping intervals

Given an array of intervals `intervals` where `intervals[i] = [starti, endi]`, return _the minimum number of intervals you need to remove to make the rest of the intervals non-overlapping_.

**Note** that intervals which only touch at a point are **non-overlapping**. For example, `[1, 2]` and `[2, 3]` are non-overlapping.

### Solution

This is one of the important interval problems and lies in the greedy paradigm. This is same as the movie problem of CSES. The idea is to sort based on end point of intervals. To prove this observe that it will be optimal for the first interval. As since each interval has equal profit while ending first has least chance of colliding with others. This argument can be extended to incorporate others as well. 

```cpp
class Solution {
public:
    int eraseOverlapIntervals(vector<vector<int>>& v){
        sort(v.begin(),v.end(),[&](vector<int> &a,vector<int> &b){
            if(a[1]==b[1])return a[0]<b[0];
            return a[1]<b[1];
        });
        int prev = -1e9,ans = 0,n=v.size();
        for(int i=0;i<n;i++){
            if(v[i][0]>=prev){
                prev=v[i][1];
                ans++;
            }
        }
        return n-ans;
    }
};
```

## Minimum number of arrows to burst ballons

There are some spherical balloons taped onto a flat wall that represents the XY-plane. The balloons are represented as a 2D integer array `points` where `points[i] = [xstart, xend]` denotes a balloon whose **horizontal diameter** stretches between `xstart` and `xend`. You do not know the exact y-coordinates of the balloons.

Arrows can be shot up **directly vertically** (in the positive y-direction) from different points along the x-axis. A balloon with `xstart` and `xend` is **burst** by an arrow shot at `x` if `xstart <= x <= xend`. There is **no limit** to the number of arrows that can be shot. A shot arrow keeps traveling up infinitely, bursting any balloons in its path.

Given the array `points`, return _the **minimum** number of arrows that must be shot to burst all balloons_.

### Solution

Again idea is to look at intervals now for any two intervals it will be optimal to put the arrow at the end of some interval. Now since the first interval (meaning ending first) has to get the arrow we sort by ending point. 

```cpp
class Solution {
public:
    int findMinArrowShots(vector<vector<int>>& v) {
        sort(v.begin(),v.end(),[&](vector<int> &a,vector<int> &b){
            if(a[1]==b[1])return a[0]<b[0];
            return a[1]<b[1];
        });
        long long ans=0,n=v.size(),prev=-1e18;
        for(int i=0;i<n;i++){
            if(prev<v[i][0]){
                ans++;
                prev=v[i][1];
            }
        }
        return ans;
    }
};
```


## Can place flowers?

You have a long flowerbed in which some of the plots are planted, and some are not. However, flowers cannot be planted in **adjacent** plots.

Given an integer array `flowerbed` containing `0`'s and `1`'s, where `0` means empty and `1` means not empty, and an integer `n`, return `true` _if_ `n` _new flowers can be planted in the_ `flowerbed` _without violating the no-adjacent-flowers rule and_ `false` _otherwise_.

### Solution

The idea is simply to trying to flowers. If we can do so we should and this way we can put maximum amount of flowers. Note that if we can place a flower we  should and that is the best strategy. 

```cpp
class Solution {
public:
    bool canPlaceFlowers(vector<int>& v, int n) {
        int ans=0;
        int m = v.size();
        for(int i=0;i<m;i++){
            if(v[i]==0){
                bool ok1 = false;
                bool ok2 = false;
                if(i==0 || v[i-1]==0)ok1=true;
                if(i==m-1 || v[i+1]==0)ok2=true;
                if(ok1 && ok2){v[i]=1;ans++;}
            }
        }
        return ans>=n;
    }
};
```

## Greatest common divisor of strings

For two strings `s` and `t`, we say "`t` divides `s`" if and only if `s = t + t + t + ... + t + t` (i.e., `t` is concatenated with itself one or more times).

Given two strings `str1` and `str2`, return _the largest string_ `x` _such that_ `x` _divides both_ `str1` _and_ `str2`.

The problem is just brute force if `O(n*m)` complexity is allowed. However we have to it in `O(n+m)`

### Solution 

First of all in these types of questions observe that whatever the greatest common divisor will be it has to be the prefix of both strings. Consider it as x. Now `s = x+x+x`  and `t=x+x+...x`. If we write `s+t` and `t+s` Both will be same if and only if there was x which made it reapeated. Now if that is not the case we can not get a gcd. Now for the final time we need to find the length of this gcd string. 

Consider the prefix `gcdBase` as the common prefix of strings such that `len(gcdBase)=gcd(len(s),len(t))`. Now consider if there is a length `x<gcdBase` then x must divide `gcdBase` by length property of gcd base. Clearly `gcdBase = x+x` meaning `gcdBase` is also a candidate. Since `gcdBase` has to be the maximal length larger length strings will fail the length property. It turns out that if exist `gcdBase` is the most optimal answer. 

```cpp
class Solution {
public:
    string gcdOfStrings(string str1, string str2) {
        if(str1+str2!=str2+str1){
            return "";
        }
        int g = gcd(str1.size(),str2.size());
        return str1.substr(0,g);
    }
};
```

## Increasing triplet sequence 

Given an integer array `nums`, return `true` _if there exists a triple of indices_ `(i, j, k)` _such that_ `i < j < k` _and_ `nums[i] < nums[j] < nums[k]`. If no such indices exists, return `false`.

### Solution 

The idea is to maintain the smallest two eleements. For each element we check if its smaller or equal to first then we update first. Note that since its smaller or equalt to first we can use it to update second. And if that is not the case it means its larger than first than we can use it to update second since it might be better for next index somewhere. Note that updating first if we get very small element is ok as anyway first and second have to maintain the sorted order. 

```cpp
class Solution {
public:
    bool increasingTriplet(vector<int>& nums) {
        long long first = 1e18 , second = 1e18;
        int n = nums.size();
        for(int i=0;i<n;i++){
            if(nums[i]>second)return true;
            if(nums[i]<=first){
                first = nums[i];
            }else if(nums[i]<=second){
                second = nums[i];
            }
        }
        return false;
    }
};
```

##