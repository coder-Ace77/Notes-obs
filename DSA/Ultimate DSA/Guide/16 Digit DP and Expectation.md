
---

## What makes these problems hard

Expectation and probability are difficult for a different reason. The mathematics involved is genuinely elementary, but there is a real decision to make about which direction a computation should run, forward from the start or backward from the end, and choosing the wrong one produces a recursion that either does not terminate or computes something other than what was intended. Getting this decision right consistently requires having seen the reasoning worked through carefully at least once, since it is not something that becomes obvious from formulas alone.

## Part 1 · Digit dynamic programming

### What these problems look like

The trigger is close to unmistakable: a request to count integers within a range, where the upper end of that range may have up to eighteen or so digits, so that no loop could ever iterate through all of them directly.

Secondary signals include constraints on the digits themselves, such as forbidding two consecutive equal digits, restricting to a particular set of allowed digits, or bounding the number of distinct digits used; constraints involving divisibility by something related to the digit sum; and questions phrased in terms of bits rather than digits, which is the same technique in base two.

The first move, in every case, is the same: reduce a two-sided range to a one-sided one, since counting valid numbers from zero up to some bound `x` is a single well-defined subproblem, and the answer for a range from `L` to `R` is obtained by computing that subproblem at `R` and subtracting the result at `L` minus one. A two-sided digit dynamic program is never the right thing to write directly.

### The template

```cpp
string S;                                     // the upper bound written as a string
vector<vector<vector<long long>>> memo;       // indexed by [position][started][state]

long long go(int pos, bool tight, bool started, int state) {
    if (pos == (int)S.size()) return accept(state, started) ? 1 : 0;
    if (!tight && memo[pos][started][state] != -1) return memo[pos][started][state];

    int lim = tight ? S[pos] - '0' : 9;
    long long res = 0;
    for (int d = 0; d <= lim; d++) {
        bool nTight   = tight && (d == lim);
        bool nStarted = started || (d > 0);
        int  nState   = nStarted ? transition(state, d) : state;
        res += go(pos + 1, nTight, nStarted, nState);
    }
    if (!tight) memo[pos][started][state] = res;
    return res;
}
```

Four parts of this are worth understanding rather than simply typing out.

**The `tight` flag** records whether every digit placed so far has exactly matched the corresponding digit of the bound. While this flag is true, the next digit placed cannot exceed the bound's own digit at that position, and the instant a smaller digit is placed, the flag becomes false and stays false for the remainder of that recursive branch, since the number being built is now guaranteed to be smaller than the bound regardless of what follows. Exactly one path through the whole recursion stays tight the entire way, namely the bound itself, which is why states where `tight` is true are never memoised: memoising them would be actively incorrect unless `tight` were included as part of the memoisation key, since the same combination of position and state means something different depending on it.

**The `started` flag** tracks whether a non-zero digit has been placed yet, which matters whenever leading zeros need to be treated differently from a genuine digit of zero appearing within the number. Whether this flag is needed at all depends on the problem: counting numbers by digit sum does not need it, since a leading zero contributes nothing to a sum regardless of whether it is "real", but counting numbers with all distinct digits does need it, since without it a leading zero would be treated as occupying the digit slot for zero even in numbers that do not actually contain a zero. Deciding this explicitly, rather than guessing, is the single most common source of digit dynamic programming bugs.

**The `state`** is whatever the specific problem's constraint requires: a running digit sum, a remainder modulo some number, a bitmask of digits used so far, the previous digit placed, or anything else needed to decide validity at the end.

**The memoisation is indexed by position, `started`, and `state`, deliberately excluding `tight`.** Reusing the same memoised table across the two calls needed for `f(R)` and `f(L - 1)` is incorrect, since the memoisation was built against a different bound; the table must either be cleared between the two calls or the bound must itself be part of the key.

### When a direct formula is faster than the template

Some digit problems have a closed-form or purely combinatorial answer that is shorter to derive than the full template.

LC 233 Number of Digit One can be solved by considering each digit position in turn and counting, using a split into the digits above, at, and below that position, how many numbers up to the bound have a one there. This is the same "count in bands" idea used for CSES *Digit Queries* in chapter [[01 Implementation and Simulation]], applied to a different quantity, and it is worth deriving once even though the digit dynamic programming template would also work.

LC 357 Count Numbers with Unique Digits is pure counting, since the number of `k`-digit numbers with all distinct digits is a simple product of decreasing choices.

LC 1012 Numbers With Repeated Digits is most easily solved by counting the complement, meaning numbers with entirely distinct digits, using the combinatorial method above, and subtracting from the total. Counting the complement of a condition rather than the condition itself is a move that recurs constantly and is the central idea of chapter [[19 Combinatorics and Number Theory]].

### Automata dynamic programming

Digit dynamic programming is really dynamic programming over an automaton whose alphabet happens to be the ten digits and whose state happens to track a numeric constraint. Replacing the alphabet and the automaton with something else produces a whole family of related techniques.

CSES *Required Substring* asks for the count of strings of a given length, over some alphabet, that avoid containing a specified pattern. Building the failure automaton for that pattern, using the technique from chapter [[18 Strings]], and then running a dynamic program where the state is the current automaton state reached after each character, counting only paths that never reach the pattern's accepting state, answers this directly. **Combining a string-matching automaton with a dynamic program is a genuinely recurring technique**, and it is also how LC 3213, which combines an Aho-Corasick automaton with a dynamic program, works one level further.

---

## Part 2 · Probability and expectation

In the real world we have some things happen for an example a dice is rolled. A trial or experiment refers to the single action or procedure with an uncertail result. Each trial has many possible outcome however when a trail is done only one outcome will come regardless. Sample space refers to the set of all possible outcomes of a trial. For example sample space for dice is `{1,2,3,4,5,6}`. 

Event is defined as the specific outcome or group/subset of outcomes that you care about. Note the thing event can have multiple outcomes in that sence it is actually different from word `event` itself. Rather a better analogy would be to think out some subset of outcomes that we are interested in for an example even role on dice is collection of three outcomes `{2,4,6}`. 

However till now we have no maths involved we can apply some logic but no maths. Random variable were actually introduced in mathematics to bring world of uncertanities to the mathematical domain. 

A random Variable(`X`) is not a variable in the traditional sence rather its a function which maps every outcome of sample space to some value. For an example $X$ could be defined as "the score on the die", or $X = 1$ if you roll an even number and $X = 0$ if odd. Variable here does not mean traditional mathematical variables rather it refers to the fact that its value varies from outcome to outcome. 

Now we can define probability function as either events or equivalently as random variable value since random variable value has to be one to one mapping with actual events. 

Finally its worth mentioning what this notation means `X=x` means where `X` is some random variable and `x` is some real number. `X=x` actually defines a set of outcomes when random variable achieves value x which means `X=x` is actually set of outcomes or more formally event. 

#### Probability 

Probability is a measure of how likely an event is to happen, quantified between $0$ (impossible) and $1$ (certain). There are three ways to think about it. A probability is a function that maps the set of events to its respective likely hood a value between `0` and `1`. 

Now in the sea of all possible outcomes for an event to happen it is enough that any one of the outcome happens for example if event is dice score is even its enough that any one of the {2,4,6} is achieved. This takes us to the primitive definition of probability. 

```
P[Event]=(Number of favourable outcomes / All outcomes)
```

- Imagine the set of all possible outcomes as a total area equal to $1$. The probability of an event is simply the "area" or "weight" that event occupies within that total space.
- note that `P(X=x)` is the prob of an event when outcomes achieve random variable value x. 

Note that we can also define following `X>=x` or `X<=x` or `X=x or X>=y` which means just set of outcomes where random variable achieves value atleast `x` or `atmost x`. 

- Input of probability function is an event `A`
- Output is a real number. 
- `P(A)>=0`
- `P(S)=1` probablity of event which means any out of sample space is `1` (guaranteed). 
- For two mutually exclusive events `P(AUB)=P(A)+P(B)` either A or B happens
- `P(A U B) = P(A)+P(B)-P(A intersection B)`. 

#### Expectation 

Expectation is used to predict future we think of it as follows. In the ideal world if some event has prob of comming 1/2 we expect half the time A to come and half time `B`. However for small sample we can get lucky and get A multiple times. 

Expected value is simple the sum of all numerial values `x` that `X` can take and multiplied by thier probab of event. 

```
E[X] = sum x.P(X=x) 
```

Note that we actually finding the mean of random variable in a way or another. For example if rv is rolled on dice. Then expection is (1+2+3+4+5+6)/6 = 21/6 = 3.5 

Its not the median value. Note note that to solve the expectation problem via above formulation require us to list all possible outcomes. Here we learn `Linearity of Expectation` - 

For any random variables `X_1,X_2,...,X_n` weather they depend on each other or not. 

```
E[X1+X2+....+Xn]=E[X1]+E[X2]+...+E[Xn]
```

To just clarify what is meaning of `X1+X2` - 

Since a random variable $X$ is a function that maps an outcome $\omega$ in our sample space $S$ to a real number, adding two random variables $X_1$ and $X_2$ creates a **new function** $Y = X_1 + X_2$.

This new function $Y$ operates by taking an outcome $\omega$ and adding the outputs of $X_1$ and $X_2$ for that specific outcome:

$$Y(\omega) = (X_1 + X_2)(\omega) = X_1(\omega) + X_2(\omega)$$

It just means for any outcome the values get added for respective rvs. Similarly we can define 

```
X1*X2(x) = X1(x)*X2(x)
X1-X2(x) = X1(x) - X2(x)
X1/X2(x) = X1(x)/X2(x)
```

In this sence now rv can be considered like any other function just notations are changed. 

Multiplication is especially neat when working with **indicator variables** (which take values $0$ or $1$):

Let:

- $I_A = 1$ if event $A$ happens, $0$ otherwise.
- $I_B = 1$ if event $B$ happens, $0$ otherwise.

If we create the new random variable $Y = I_A \cdot I_B$:

- If $A$ happens and $B$ happens $\implies Y = 1 \cdot 1 = 1$.
- If either $A$ or $B$ (or both) does NOT happen $\implies Y = 0$.

So $I_A \cdot I_B$ is itself an indicator variable for the joint event $A \cap B$ (both $A$ and $B$ happening)!

Finally multiplication also follows 

```
E[X1*X2] = E[X1]*E[X2]
```

**Linearity of expectation** states that the expected value of a sum of random quantities equals the sum of their individual expected values, and critically, this holds regardless of whether those quantities are independent of one another.

#### Law of total expectation

Law of total probability just says that - 

$$E[X] = \sum P(\text{Outcome}_i) \cdot E[X \mid \text{Outcome}_i]$$

Meaning calculating expected value for some random variable `X` can actually be calcualted by fixing the some ith outcome and then taking weighted average of all outcomes. This is the formula we most use while calculating the expectation by dynamic programming. 

### Calculation implementation

When a process unfolds in steps through different states (like moving on a grid or removing items), we use **Dynamic Programming** based on the **Law of Total Expectation**. 

In pull dp to calculate the `dp` value for state `i` we find `dp[i]` from all the states where it could have came from. 

In forward or push dp when we reach state `i` , `dp[i]` is already calculated so we push the values forward and that's about it. 

```cpp
// Pull (backward)
dp[0] = 1;
for (int i = 1; i <= n; i++) {
    dp[i] = dp[i-1] + (i >= 2 ? dp[i-2] : 0);
}

// Push (forward)
dp[0] = 1;
for (int i = 0; i < n; i++) {
    dp[i+1] += dp[i];
    if (i + 2 <= n) dp[i+2] += dp[i];
}
```


**Push** is easier when you can list a state's outgoing moves but its incoming ones are messy. Examples are probability DP, "from this state I can jump to these states," and DP over a graph's edges.

**Pull** is easier when the predecessors form a clean range.

For probability we use forward dp since most of the time our state looks like 

```
dp[s] = prob of being in state s
```

Now we can simple move to next states via set of allowed moves say we can go to s1 from s. Then prob of comming upto `s` is known so for going to `s1`

```
dp[s1]+=dp[s]*P(next step we go from s to s1)
```

Now to understand why this is the case first we need to make sure that the randoms variable for the each move is addititive

```
X = X1+X2+X3... +Xn
```

Here meaning is that total random variable is actually additive in nature which is mostly the case. Now since each choice at any step is independent it follows that probability function itself is addititve finally we use concept of conditional probability which here can be used to fix a choice and then move forward. And that's pretty much it the explainer of dp formulation. 

We can also state it like this since comming to an state is like additive given multiple independent choices we know prob to reach a state is just sum of probs of reaching earlier states however we multiply by prob of reaching from here to there. 

```
P(state) = sum of prob(state_prev)*P(prob of reaching from state_prev to state)
```

Finally keep in mind base case say we start from `a` then `dp[a]=1` at 0 step since we start at `a`.
Now let's think about the expection usually we will be req to find some expected value when traversing some graph or something like that. So we mostly make state as 

```
dp[s]=Expected remaining cost or steps to reach the goal from state s.
```
Here we can pull value from prev if we can reach s via a and b then to reach s we can reach via either a or by either b. Meaning Expected value for the s will be average of both. Finally what will be `dp[goal]` is will be 0 since expected cost to reach goal is 0. 

TO calculate expectation we use law of total expectation.

Now lets think about this problem say at each step you will be given either `1` or `2` what is expeted number of steps needed to reach `x`

So with law of total expectation. 

```
dp[i] = remaining points needed
dp[i] = (dp[i-1]*1/2+dp[i-2]*1/2)
```

