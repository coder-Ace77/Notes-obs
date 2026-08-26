---
tags: [dsa, guide, index]
type: moc
status: active
---

# The Ultimate DSA Guide — Index

Companion to [[1. Ultime DSA 2026 calibration]].

The sheet tells you **what** to solve. This guide tells you **why each block exists, what the underlying machine is, and what the three or four genuine insights are** that make the whole block collapse into something you can do in 35 minutes.

---

## How to read a chapter

Every chapter has the same seven parts. This is deliberate — after two or three chapters your brain stops parsing structure and starts absorbing content.

| Part                 | What it does                                                                                      |
| -------------------- | ------------------------------------------------------------------------------------------------- |
| **The thesis**       | One sentence. If you remember nothing else, remember this.                                        |
| **Where it hides**   | How the topic is disguised in a real OA statement. Pattern-matching fuel.                         |
| **Ground floor**     | The theory, built from nothing, in prose. No "recall that…".                                      |
| **The templates**    | Code you should be able to type without thinking. Not reference material — muscle memory targets. |
| **Aha moments**      | The numbered insights. These are the actual payload of the chapter.                               |
| **Failure modes**    | The specific ways people lose 35 minutes here.                                                    |
| **Running the list** | The sheet's problems, re-ordered into warm-up → core → boss, with what each one teaches.          |

At the end of each chapter there's a **self-check**: questions you should be able to answer out loud. If you can't, the chapter didn't land, and no amount of extra problems will fix that.

---

## Chapter map

### Foundation — the blocks everything else sits on

| # | Chapter | Sheet | Why now |
|---|---|---|---|
| 01 | [[01 Implementation and Simulation]] | A | Your error rate lives here. Measure it before anything else. |
| 14 | [[14 Dynamic Programming Core]] | N | EDPC is 26 problems and 26 distinct paradigms. Best DP-per-hour in existence. |
| 08 | [[08 Segment Trees]] | H | The merge is the problem, not the update. This is the infra-OA core. |

### The 2026 hard core — where most of your time goes

| # | Chapter | Sheet |
|---|---|---|
| 02 | [[02 Intervals and Sweep Line]] | B |
| 04 | [[04 Binary Search on the Answer]] | D |
| 07 | [[07 Monotonic Stacks and Deques]] | G |
| 09 | [[09 Fenwick Offline and Mos]] | I |
| 10 | [[10 DSU Advanced]] | J |
| 11 | [[11 Shortest Paths and State Space]] | K |
| 15 | [[15 Bitmask DP]] | O |
| 16 | [[16 Digit DP and Expectation]] | P |
| 17 | [[17 DP Optimization]] | Q |
| 18 | [[18 Strings]] | R |
| 24 | [[24 Bitwise and XOR Basis]] | X |

### Breadth and tail — closing gaps, not building foundations

| # | Chapter | Sheet |
|---|---|---|
| 03 | [[03 Greedy and Exchange Arguments]] | C |
| 05 | [[05 Sliding Window and Two Pointers]] | E |
| 06 | [[06 Prefix Sums and Difference Arrays]] | F |
| 12 | [[12 Graph Structure SCC 2SAT Bridges]] | L |
| 13 | [[13 Trees]] | M |
| 19 | [[19 Combinatorics and Number Theory]] | S |
| 20 | [[20 Flows and Matching]] | T |
| 21 | [[21 Geometry]] | U |
| 22 | [[22 Constructive Invariants and Games]] | V |
| 23 | [[23 Data Structure Design]] | W |

---

## The method, restated

The sheet's rules of engagement are good but they're stated as rules. Here is the reasoning behind them, because rules you understand are rules you actually follow.

### Why 35 minutes and not 60

An OA gives you roughly 35–45 minutes per hard problem including reading, coding and debugging. Practising with a 90-minute budget trains a completely different skill: the skill of eventually getting there. That skill does not transfer. The thing you need to train is **fast paradigm identification**, and the only way to train it is to make identification the bottleneck.

Concretely: if you have not said out loud "this is a sweep line with a multiset" or "this is binary search on the answer where the check is a greedy" within **five minutes**, stop. Don't code. Write down what you *think* it might be, look at the answer, and log it. That five-minute gap is the entire signal you're collecting.

### Why first-submission accuracy and not solve count

Hidden tests. You get one shot in the sense that matters — either your submission passes the full suite or you burn ten minutes you don't have discovering that `n=1` breaks your loop. Solve count measures whether you eventually understood. First-submission accuracy measures whether you'd have passed.

Track it per block. A block under 70% is a block you re-run, not a block you move past. This is the single highest-leverage rule on the whole sheet and it is the one everybody ignores.

### The hand-trace ritual

Before you hit submit, spend ninety seconds on this list. Every single time. It feels like a waste and it is the highest-ROI ninety seconds in your practice.

- **n = 1.** Does your two-pointer loop even enter? Does `r - 1` underflow?
- **All equal.** Monotonic stacks with `<` vs `<=` live or die here. So does deduplication logic.
- **All distinct / strictly increasing / strictly decreasing.** The degenerate case for stacks, deques and greedy.
- **Maximum constraints.** `n = 10^5` with values up to `10^9` — does your sum overflow 32 bits? Multiply two of them and it definitely does.
- **Empty input / empty result.** Does your answer initialise to something that survives "no valid answer exists"?
- **Duplicates.** Does your binary search find *a* position or *the leftmost* position? Do you need `lower_bound` or `upper_bound`?
- **Negatives and zero.** Kadane, prefix sums, and every "product" problem break here.

### The per-problem loop

```
Read the statement twice. The second read is where you catch the constraint.
        ↓
Say the paradigm out loud in ONE sentence.        ← 5-minute hard gate
        ↓
If greedy: write the exchange argument. If DP: write the state and transition.
If a data structure: write what one node stores and how two nodes merge.
        ↓
Code.
        ↓
Hand-trace the edge-case list above.
        ↓
Submit.
        ↓
Log: paradigm-call time, first-submission pass/fail, and the one-line reason if it failed.
```

That log is the actual artefact of this whole exercise. The solutions are disposable. The log tells you what to practise next.

---

## A note on language

The sheet is heavily CSES / Codeforces / AtCoder, so the templates in this guide are written in **C++**. Two reasons, neither of them snobbery:

1. Iterator-based ordered containers (`std::map`, `std::multiset`, `lower_bound` on them) make roughly a third of this sheet — interval maps, sweep lines, order-statistics — dramatically shorter to write. `TreeMap` in Java is the closest equivalent and is fine.
2. `__int128`, `__builtin_popcount`, `__builtin_clz` and raw `long long` arithmetic remove a whole class of overflow bugs from your working memory.

If you're doing OAs in Java or Python, read the templates as pseudocode with a precise contract. Every template here notes the equivalent construct where it isn't obvious. Python users: for anything in chapters 08, 09, 17 and 18, be aware that constant factors will bite you on CSES and Codeforces even when the complexity is right.

---

## Suggested schedule

The sheet's three-pass ordering is correct. Here's how it maps onto chapters:

**Pass 1 — Calibration (~3 weeks).** Chapters [[01 Implementation and Simulation]], [[14 Dynamic Programming Core]] (the EDPC half only), [[08 Segment Trees]] (sections H1–H2 only).
You are not trying to finish these. You are trying to *measure* three things: your implementation error rate, which DP paradigms are rusty, and whether your segment tree is reflexive or reconstructed-from-memory. Everything after this is scheduled off the numbers you get here.

**Pass 2 — Depth.** Chapters 02, 04, 07, 08 (H3–H5), 09, 10, 11, 15, 16, 17, 18, 24.
This is the bulk. Expect it to take months, not weeks, and expect roughly 60% of your total time on the sheet to land here.

**Pass 3 — Breadth and tail.** Chapters 03, 05, 06, 12, 13, 19, 20, 21, 22, 23.
Either you're already competent here, or the topic is rare enough that it's a closing gap rather than a foundation. Do not start here because it feels comfortable. That instinct is exactly why the depth pass never gets done.
