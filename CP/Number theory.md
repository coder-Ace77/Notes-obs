
---

## Gcd(greatest common divisor)

Gcd of two number a and b is the largest number g that divides both a and b. If one number is zero gcd by definition is second number.

1. We call `b` divisor of `a` if `a = kb` for some `k`.
2. `gcd(a,b) = a if b=0 gcd(b,a mod b) otherwise`

```cpp
int gcd (int a, int b) {
    if (b == 0)
        return a;
    else
        return gcd (b, a % b);
}
```

Proof

Let a>b

Suppose g is gcd , since if g divides a and b both g divides a - b. So we can remove b from a multiple times and remaining will be `a%b`. which consludes proof. This algorithm has time complexity of `log(n)`.

3. `lcm(a,b) = a*b/gcd(a,b)`

4. Elementry theorum of division - for any two numbers `a,b` there will always exist unique numbers `m and r` such that `a = m*b+r`

5. `a mod k` creates equivalence relation where `a == b mod k` if a and b both yield same remainder on division by `k`
6. Two numbers `a and b` are called coprime if `gcd(a,b)=1`
7. Let a divides `b*c` and `gcd(a,c)=1` then a divides b.
8. In general if a divides `b*c` and `gcd(a,c)=g` then `a / g divides b`

Proof 

By definition g divides a and c.

`gcd(a,c)=g` or `gcd(a/g,c/g)=1`
Now given ,
`b*c = a*k => b*g*c1 = g*a1*k => b*c1 = a1*k` `gcd(a1,c1)=1`
So , `a1 divides b` by theorum 7.

9. Suppose we have a function `f(x) = ax mod n`. and `gcd(a,n)=1`. Then when we apply all the possible inputs for x ranging from `1 to n` then we get all possible outputs `0, ... , n-1` exactly once. Think of it as a perfect **permutation** or "shuffle" of the numbers. The reason is that we can prove that no two numbers `f(ax1) = f(ax2) if x1!=x2`. Observe that f(x) is repetivive additon of a in `mod n ` universe

10. Now if `gcd(a,b)=1` then there exist `x` such that `ax =1 mod b`. Here x is called multiplicative inverse of `a mod b`. 
11. Fermat theorum - Lets look at the multiplicative addition `f(x) = a^x mod n`. If `gcd(a,n)=1` then `a^x` is also coprime with `n`
12. Linear diophantine equations are equations in the general form `ax+by=c`. Bezout theorum says that `ax+by` has a solution if and only if c is the multiple of gcd(a,b) in the other words only the multiple of gcd(a,b) can be produced by `ax+by`. Consider `g = gcd(a,b)`. First we will find the solution to `ax+by=g` 

```cpp
int gcd(int a, int b, int& x, int& y) {
    if (b == 0) {
        x = 1;
        y = 0;
        return a;
    }
    int x1, y1;
    int d = gcd(b, a % b, x1, y1);
    x = y1;
    y = x1 - y1 * (a / b);
    return d;
}

bool find_any_solution(int a, int b, int c, int &x0, int &y0, int &g) {
    g = gcd(abs(a), abs(b), x0, y0);
    if (c % g) {
        return false;
    }

    x0 *= c / g;
    y0 *= c / g;
    if (a < 0) x0 = -x0;
    if (b < 0) y0 = -y0;
    return true;
}
```


## Primes

A prime number is a number with only two divisors 1 and itself. A non prime number can always be written as the products of primes. `a = p1*p2*..` this is what is called as fundamental theorum of arithematic. 

Every even integer bigger than 2 can be split into two prime numbers, such as 6 = 3 + 3 or 8 = 3 + 5.

Sieve of Eratosthenes is an algorithm for finding all the prime numbers in a segment  `[1;n]`  using  `O(n \log \log n)` operations.

The algorithm is very simple: at the beginning we write down all numbers between 2 and  $n$ . We mark all proper multiples of 2 (since 2 is the smallest prime number) as composite. Observe that we are starting from `j = i*i` reason being `i*(i-1)` will already being marked earlier. 

```cpp
int n;
vector<bool> is_prime(n+1, true);
is_prime[0] = is_prime[1] = false;
for (int i = 2; i <= n; i++) {
    if (is_prime[i] && (long long)i * i <= n) {
        for (int j = i * i; j <= n; j += i)
            is_prime[j] = false;
    }
}
```

Distinct prime factors of a number n is very small. Reason being products multiply very fast. For instance every number in range `1e18` can have at most `8` prime factors. 

However the prime numbers in a given range can be large - which is in range `1,n` there are approx `n / log(n)` prime numbers.

The prime factorization sequence which is `a = p1*p2...pn` will be of length `log(a)`. Which is uaually very small. 

Now using sieve we can find the prime factorization of numbers `lpf` least prime factor accray which is simple modification of sieve.

```cpp
int n;
vector<int> lpf(n+1);
for(int i=2;i<=n;i++){
	lpf[i] = i;
}
lpf[0] = lpf[1] = false;
for (int i = 2; i <= n; i++) {
    if (lpf[i] && (long long)i * i <= n) {
        for (int j = i * i; j <= n; j += i)
	        if(lpf[j]==j)lpf[j] = i;
    }
}
```

Since it stores the smallest prime factor for any number. Then we can simply use

```cpp
while(n>1){
	cout<<lpf[n]<<endl; // prime factors
	n/=lpf[n];
}
```

Tip: Sometimes it is justified to store the prime factorization as map of factors as keys and freq as values.
Othertimes we convert a number into its reduced form where we store just minimal information. 
However if only the distinct prime factors are needed we can simply store them in a number itself.

```cpp
int trace = 1; // calcs all the distinct primes
while(n>1){
	int fac = lpf[n];
	if(trace%fac==0)trace*=fac;
	n/=fac;
}
```

Number of divisor of number can simply be found using the formula

If a is a number with `a = p1^x1*p2^x2...` then 

`d(a) = number of divisors of a are = (x1+1)*(x2+1)*...` This is becaouse to form a number out of a we need to pick some numbers from this. 

Note if we want to do the prime factorization using brute force we have `sqrt(n)` algorithm. 

```cpp
for(int i=2;i*i<=n;i++){
	if(n%i==0){
		cout<<i<<" "<<endl;
	}
}
```

## Binary exponentiation


```cpp
long long binpow(long long a, long long b) {
    if (b == 0)
        return 1;
    long long res = binpow(a, b / 2);
    if (b % 2)
        return res * res * a;
    else
        return res * res;
}
```

## Matrix exponentiation

The idea is to calculate large powers of matrices very quickly. We again use the idea of binary exponentiation. So if we need to calcualte the power of `M^x` then it can be recusively solved as `M^(x/2)*M^(x/2)` and combining the results. 

```
```

Now Matrix exponentiation is used to solve the linear recurrence with very fast speed. Linear recurrences cna be written useually as - 

`T.State(n-1) = State(n)` So if formula can be written as some relation to itself then we can build the matrix our of it and solve it. 

For instance fibonnaci serices can be solved as -  

![[Pasted image 20251030141513.png]]

here unknow matrix by relation finding is 

![[Pasted image 20251030141537.png]]

