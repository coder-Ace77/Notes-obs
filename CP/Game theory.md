
---

Patterns

1. Argument based/ symmetric
2. DP - common
3. Nim games and its variations
4. Sparsh grundy values
5. Observation/Print and see types


## Combinatorial games:

### Definition

 It is a game that satisfies the following conditions.
1. There are two players.
2. There is a set, usually finite, of possible positions of the game.
3. The rules of the game specify for both players and each position which moves to other positions are legal moves. If the rules make no distinction between the players, that is if both players have the same options of moving from each position, the game is called impartial; otherwise, the game is called partizan.
4.  The players alternate moving.
5. The game ends when a position is reached from which no moves are possible for the player whose turn it is to move. Under the normal play rule, the last player to move wins. Under the mis`ere play rule the last player to move loses.
If the game never ends, it is declared a draw. However, we shall nearly always add the following condition, called the ending condition. This eliminates the possibility of a draw.
6. The game ends in a finite number of moves no matter how it is played.

It is important to note what is omitted in this definition. No random moves such as the rolling of dice or the dealing of cards are allowed. This rules out games like backgammon and poker.

A combinatorial game is a game of perfect information: simultaneous moves and hidden moves are not allowed.

### Backward induction

We analyze this game from the end back to the beginning. This method is sometimes called backward induction. So we simulate where it will end and move backward.

We call a position P in a combinatorial game if the conditions are winning for the Previous player(the player who just moved) and otherwise N position for Next player winning.

In impartial combinatorial games, one can find in principle which positions are P-positions and which are N-positions by (possibly transfinite) induction using the following labeling procedure starting at the terminal positions. We say a position in a game is a terminal position, if no moves from it are possible.

### Modelling

P-positions and N-positions are defined recursively by the following three statements.

1. All terminal positions are P-positions.
2. From every N-position, there is at least one move to a P-position.
3. From every P-position, every move is to an N-position

Or we canv define P postions as

A position is a P position if all the links outgoing from `P` are `N`. 

we can define N position as 

A position is N position if atleast one link from here is `P`

### Subtraction games

Let S be a set of positive integers. The subtraction game with subtraction set S is played as follows. From a pile
with a large number, say n, of chips, two players alternate moves. A move consists of removing s chips from the pile where `s in S`. Last player to move wins.

Solution: We can label the P and N positions for such games very easily by dp. However time complexity `O(S*n)` which may or may not be possible in the situation. Sometimes the set pickup becomes simplified and we can figure out a rule or a pattern that can allow us to solve it much more quickly in constant time. However this has to be seen by observation.

### General solution

Usually in substraction games is much harder to predict at which positions W or L will be rather this is more general observation - 

Suppose we build the string of positions by `P/N` or by `W/L`. We can define the substraction game by a string of T where every t in T is either W or L depending upon the position. This will be infinite string. 

Now here is the observation:

Given a set of moves in S. The string T is always periodic. With period always dependent on the set S.
Also the period has upper bound with `lcm(S)`. Thus for substraction games if `max(S)` is not very large we can always solve uptil `3*length of lcm of S` and then find the period using `kmp` algorithm. Once period is known it is bound to repeat and we can solve it pretty easily. 


### Theorum

Every natural number can be written as sum of fibonacci numbers.


## Nim games

It is assumed that the game is **finite**, i.e. after a certain number of moves, one of the players will end up in a losing position — from which they can't move to another position. On the other side, the player who set up this position for the opponent wins. Understandably, there are no draws in this game. Such games can be completely described by a _directed acyclic graph_: the vertices are game states and the edges are transitions (moves). A vertex without outgoing edges is a losing vertex (a player who must make a move from this vertex loses).

Since there are no draws, we can classify all game states as either **winning** or **losing**. Winning states are those from which there is a move that causes inevitable defeat of the other player, even with their best response. Losing states are those from which all moves lead to winning states for the other player. Summarizing, a state is winning if there is at least one transition to a losing state and is losing if there isn't at least one transition to a losing state.

Our task is to classify the states of a given game.

### Game description

There are several piles, each with several stones. In a move a player can take any positive number of stones from any one pile and throw them away. A player loses if they can't make a move, which happens when all the piles are empty.

The game state is unambiguously described by a multiset of positive integers. A move consists of strictly decreasing a chosen integer (if it becomes zero, it is removed from the set).

This game obeys the restrictions described above. Moreover, _any_ perfect-information impartial two-player game can be reduced to the game of Nim. Studying this game will allow us to solve all other similar games, but more on that later.

### Solution

**Theorem.** The current player has a winning strategy if and only if the xor-sum of the pile sizes is non-zero.

### Proof 

The proof is as follows observe that in the terminal case all the piles are zero and so xor sum is zero. Now from any non-zero state `W` we can make a choice to move to zero-state `L`.

For each `W` position there is a move to `L`. We can do this by looking at the largest pile and removing some stones from it so that xor sum becomes `0`. 

For  each `L` positions all the transitions lead to `W` positions. This concludes the proof.

Now with this said there are many variations of nim games which can be reduces to basic nim game and be solved. For instance consider this

### Problem 1

Suppose in nim game a player can add/remove stones to the pile. 

### Solution

Consider if there were only `reduction` of stones was possible then if this is a winning position `W` then removing stones makes sence and thus should be done. If however this is losing then just by the removal of stones even if this player adds the stones. In the next step other player will reduce exactly the same stones from same pile leading to player 1 being at the same spot. So a `L` state can't be converted to `W` by addition and `add` moves have no effect on the game. And final answer is again `xor sum`. 

### Problem 2

In a **misère game**, the goal of the game is opposite, so the player who removes the last stick loses the game. It turns out that the misère nim game can be optimally played almost like a standard nim game. The idea is to first play the misère game like the standard game, but change the strategy at the end of the game. The new strategy will be introduced in a situation where each heap would contain at most one stick after the next move. In the standard game, we should choose a move after which there is an even number of heaps with one stick. However, in the misère game,we choose a move so that there is an odd number of heaps with one stick. This strategy works because a state where the strategy changes always appears in the game, and this state is a winning state, because it contains exactly one heap that has more than one stick so the nim sum is not 0.