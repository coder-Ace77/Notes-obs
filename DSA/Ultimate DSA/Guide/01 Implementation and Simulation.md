---
tags: [dsa, guide, implementation, parsing, simulation]
chapter: 1
sheet-section: A
---

# Chapter 1 · Implementation, Parsing & Simulation

> **Read this before you start the problems.** You don't need to have seen any of them. Every idea comes with a small example you can follow from scratch.

Back to [[00 Guide Index]] · Sheet section **A** in [[1. Ultime DSA 2026 calibration]]

---

## What makes these problems hard

In most chapters, a problem is hard because you don't know what to do. Here, you usually know what to do within a minute of reading. The difficulty is that one problem can carry ten or twelve separate requirements, and all of them have to be right at the same time.

That is more than you can comfortably track while you are also writing code. Some requirements slip out of view, and the ones that slip tend to be the ones covering unusual cases, which is what the hidden tests are made of. This is why a solution here can feel finished and still fail.

Most of the advice in this chapter comes down to two things: writing the requirements somewhere you can see them, and arranging your code so that each requirement lives in one identifiable place.

---

## What these problems look like

You will usually recognise this type on the first read, which is part of why they cost people so much time. Because the approach is obvious, it is tempting to start coding after two minutes, and the requirements you skimmed past only surface later as failed submissions.

Signs that you are in this chapter:

- The statement contains **rules with exceptions**, such as "words are separated by single spaces, except on the last line."
- The input is a **string that carries meaning**, such as a maths expression, a file path, or a chemical formula.
- There is a **process to run**, and the answer is wherever that process ends up.
- The process has **far more steps than you could execute**, such as a billion operations on an array of size 100.
- You have to **design a class**, and the grader will call your methods in an order you didn't anticipate.

These are the most common type of question in product and startup online assessments. They are also the type that most people practise least, because they look unglamorous next to graph and DP problems. That gap between frequency and preparation is the reason this block sits first on the sheet.

---

## Part 1 · Two kinds of difficulty

It is worth separating the two reasons a problem can be hard, because they call for opposite responses.

**Idea problems** are hard because you don't know what to do. There is some fact about the structure that makes the problem tractable, and until you find that fact you have nothing to write. Time spent thinking is productive here, because the missing piece is a piece of understanding.

**Detail problems** are hard because doing the obvious thing correctly is difficult. There is no hidden fact waiting to be found. The statement gives you a list of behaviours and asks you to produce all of them, and progress happens gradually as you handle more cases correctly rather than in a single moment of insight.

Sections B through X of the sheet are mostly idea problems. Section A is almost entirely detail problems.

This distinction matters because the habit that serves you well elsewhere works against you here. In an idea problem, pausing to think before writing is correct, since the bottleneck is understanding. In a detail problem your understanding is already complete, so additional thinking produces nothing. The bottleneck is how much you can hold in view at once, and the way to widen it is to start writing things down rather than to keep reasoning.

---

## Part 2 · The four pieces every problem here has

Whatever the surface of the problem looks like, it is built from the same four components:

| Piece         | What it means                                |
| ------------- | -------------------------------------------- |
| **State**     | what you are keeping track of                |
| **Invariant** | a rule that stays true the whole way through |
| **Steps**     | the operations that change the state         |
| **Output**    | turning the final state into an answer       |

Each of these has a reliable way to find it, which is more useful than being able to recognise it once someone points it out.

### Finding the state: the handover test

There is one question that works on any problem in this block, including problems you have never seen:

> **"I am halfway through. Someone else has to take over from here. What do I have to tell them?"**

Whatever you would have to tell them is your state, and anything you would not mention is not.

**Example.** You are given a list of words and a page width of 16 characters. You have to pack the words into lines and pad each line with spaces so that it is exactly 16 characters wide.

Suppose you stop halfway. Your replacement needs two things: the words sitting on the line you are currently building, and the total number of letters in those words, so that they can work out whether the next word will fit.

They do not need the lines you have already finished. Those lines are complete and cannot affect any future decision, so they are output rather than state.

That separation is worth being deliberate about. Some of what you hold is information you make decisions with, and some of it is material you are accumulating for the answer. Keeping them in the same mental bucket is what makes a problem with two real variables feel like a problem with six.

**A second example.** You are given a string describing a file tree, where tab characters indicate nesting depth:

```
dir
	subdir1
	subdir2
		file.txt
```

You need the length of the longest full path, which here is `dir/subdir2/file.txt`.

Ask the handover question again. Your replacement needs the running path length at every depth above the line currently being read. Depth 0 is `dir` at 3 characters, depth 1 is `dir/subdir2` at 11 characters, and the current line sits at depth 2.

That description is an array indexed by depth, which is the same thing as a stack.

The useful part is how you arrived there. You did not have to recall that this kind of problem uses a stack. You asked what a replacement would need, and the structure came out of the answer. That is why the handover test is worth making automatic: it works when you have no prior familiarity with the problem at all.

### The invariant: the rule that holds between steps

An invariant is a statement that is true before each step and true again after it.

For the word-packing example, the invariant is that the words currently on the line, plus one space between each adjacent pair, fit within the page width. Written as a formula:

```
letterCount + (numberOfWords - 1) <= 16
```

Writing it as an actual formula rather than as a general feeling is worth the thirty seconds, for two reasons.

The first is that the invariant tells you when to act. You do not need a separate rule for when to start a new line, because the rule is simply that you start a new line at the point where adding the next word would break the invariant. The trigger is derived from the invariant rather than being another thing to remember.

The second is that it settles `<` against `<=` by arithmetic instead of by intuition. Take the smallest interesting case, a single word that is exactly 16 letters long, and substitute it: `16 + (1 - 1) = 16`. That word should fit on the line, so the comparison has to be `<=`. This check takes a few seconds and removes the most frequent source of off-by-one errors in this whole chapter.

There is also a broader reason to write invariants down. A bug in this kind of code is, almost by definition, a moment where your invariant has become false without you noticing. If you never stated the invariant, there is nothing concrete to check your code against, so you end up searching for a violation of a rule you never articulated. That is why these particular bugs feel so much harder to locate than algorithmic ones.

### Steps: separate deciding from building

This one is about how you organise your code, and it removes more errors than any other habit in this chapter.

> **The loop that decides *when* something is finished should not also be the code that *builds* the finished thing.**

Return to word-packing. The natural way to write it is as a single loop that collects words and, once the line is full, performs the space-padding arithmetic in place. That version ends up several levels deep in nested conditions, because the two exceptions — the last line, and a line holding only one word — get buried inside the padding logic. The code becomes hard to check not only because it is wrong, but because the structure hides where the wrongness is.

Splitting it looks like this:

```cpp
vector<string> result;
vector<string> line; int letters = 0;

for (string& w : words) {
    // would adding w break the invariant?
    if (letters + (int)line.size() + (int)w.size() > width) {
        result.push_back(render(line, letters, width, false));
        line.clear(); letters = 0;
    }
    line.push_back(w);
    letters += w.size();
}
result.push_back(render(line, letters, width, true));   // last line
```

The loop now answers a single question, which is when a line is complete. The `render` function answers a different single question, which is what a completed line should look like. Both exceptions live inside `render`, where they become two short conditions at the top level rather than two extra branches nested inside existing ones.

The general form of this habit is to keep the code that recognises a unit apart from the code that processes a unit. It applies wherever you are cutting a stream into pieces, which includes breaking text into lines, splitting a string into tokens, grouping records, and flushing a buffer.

### Output: where the exceptions tend to live

Notice what happened to the exceptions in that rewrite. Almost every "except…" clause from the statement ended up inside `render` rather than inside the loop.

This happens consistently, and there is a reason for it. Exceptions in these problems are usually about how a completed unit is presented, not about when a unit becomes complete. Knowing that in advance means you can file each rule into the right part of your program while you are still reading the statement, which is the subject of the next part.

---

## Part 3 · Reading the statement as a list of requirements

In an idea problem you read the statement to extract the underlying structure, and after that you rarely need to look at it again. In a detail problem the statement remains the specification for the entire time you are working, because every sentence in it corresponds to something a hidden test will check.

Because of that, it is worth reading it twice with different goals.

**The first read is for shape.** You are working out what the process is, what you are tracking, and what the answer looks like. Details can be skipped, since you are only trying to see the overall structure.

**The second read is for requirements.** Go through it a sentence at a time and write each requirement on its own line. For each one, record three things:

1. **Whether it is a main rule or an exception.** "Words are separated by single spaces" is a main rule; "except on the last line" is an exception to it.
2. **What it applies to** — the process as a whole, an individual step, or a completed unit. This tells you which part of your code it belongs in, using the split from Part 2.
3. **Whether it interacts with another rule, and which one takes priority if they conflict.**

The third question is the one that gets skipped, and it is where these problems tend to bite. The word-packing problem has two exceptions, one for the last line and one for a line holding a single word, and those two can occur together on a last line that has one word. Whether your code handles that combination is only a question you will think to ask if the two rules are written down next to each other.

Some problems consist almost entirely of this work. **LC 591 Tag Validator** gives you roughly ten rules about what makes a tag valid, covering the name format, a length limit, nesting, unclosed tags, CDATA sections, and a requirement that the whole string be wrapped in a single outer tag. Several of those rules interact. The way through is to enumerate them and verify each one individually, because there is no single observation that makes the others unnecessary.

One practical note: write the requirement list as a comment at the top of your code file rather than on paper. You will refer back to it several times while coding, and glancing up within the same window costs less attention than looking away from the screen. It also becomes the checklist you use during the verification pass in Part 6.

---

## Part 4 · The three ways these problems become hard

Everything so far gets you to the point of correctly implementing the obvious approach. Some problems in this block are genuinely harder than that, but they become hard in a limited number of ways, and knowing which ways they are gives you something to check against when you are stuck.

There are three. Each has a question you can ask to identify it and a set of standard responses.

---

### Way 1 · The data is in the wrong shape

**The question to ask: are my operations awkward because of how I have chosen to store things?**

In this case the state itself is simple, but the representation you reached for first makes the operations you need expensive. The response is to change the representation so that the operations you actually perform become cheap.

**Example.** You are implementing a text editor with a cursor, supporting four operations: type a character, delete a character, move the cursor left, and move the cursor right.

The obvious representation is a string together with an integer holding the cursor position. Under that choice, typing in the middle of the document requires shifting every character after the cursor, which is slow when the document is large.

The useful observation is that every operation happens at the cursor and nothing ever happens far away from it. That suggests putting the cursor at the boundary between two stacks, with the top of each stack facing the cursor:

```
"hello|world"

  left  = h e l l o        (top is 'o', immediately left of the cursor)
  right = d l r o w        (top is 'w', immediately right of the cursor)
                ^ both tops face the cursor
```

Typing becomes a push onto `left`, deleting becomes a pop from `left`, and moving the cursor becomes a pop from one stack and a push onto the other. Each operation now costs an amount proportional to the size of the change rather than the size of the document.

The general form is that when all the activity in a problem happens at a single moving position, you should consider putting a stack on each side of that position. The same structure underlies undo and redo stacks, and the back and forward buttons in a browser.

Other representation changes worth having in mind:

- **Index by something other than position.** The file-path example is naturally indexed by depth rather than by line number.
- **Store differences instead of values**, or values instead of differences, depending on which one you query more often.
- **Store the reverse lookup.** If you keep asking where a particular item is, maintain a `position[item]` array alongside the main array.
- **Group items that behave identically** and track counts per group rather than tracking individuals.

A reliable way to notice this situation is that you find yourself writing a loop inside what ought to be a single operation. When that happens, the representation is usually the cause.

It is worth choosing the representation before you start typing, because changing it later means revisiting every line you have written. There is generally no time to do that inside a 35-minute budget.

---

### Way 2 · There are more steps than you can run

**The question to ask: the process runs for an enormous number of steps, but is the world it operates on small?**

When the number of possible situations is smaller than the number of steps, running every step was never the intended solution, and there will be structure in time that you can exploit instead.

There are three responses, worth considering in this order.

**(a) The process eventually repeats.** If there are fewer distinct situations than steps, then at some point a situation must recur, and once it recurs everything that follows recurs with it. You can therefore record each situation along with the step number at which you first saw it, and when a repeat appears you know the length of the cycle and can skip forward by whole cycles.

```cpp
unordered_map<State, long long, Hash> seen;
long long step = 0;
while (step < n) {
    if (seen.count(cur)) {
        long long loopLength = step - seen[cur];
        long long loops = (n - step) / loopLength;   // how many whole loops fit
        step += loops * loopLength;
        seen.clear();            // without this line, the loop never terminates
        if (step >= n) break;
    } else {
        seen[cur] = step;
    }
    cur = advance(cur);
    step++;
}
```

The `seen.clear()` call is easy to leave out and worth understanding rather than memorising. Without it, the next iteration detects the same repeat again, calculates that zero whole cycles now fit in the remaining distance, and makes no progress, so the loop runs forever.

The harder part of this technique is not detecting the repeat but deciding what to record. Consider **LC 466**, where you repeatedly run through a string `s1` and match characters of `s2` as you go. If you record the entire situation, including every character matched so far, nothing ever repeats, because you are continuously making progress. If instead you record only how far into `s2` you have got at the end of each complete copy of `s1`, that value is a number between 0 and the length of `s2`, so it can only take a limited number of values and must repeat quickly.

The skill is therefore in choosing a summary of the situation that is small enough to recur while still being sufficient to predict what happens next. That judgement has to be made afresh for each problem, and it is the part worth practising.

**(b) There is a closed form.** In some problems there is no simulation to shorten because there was never any need to simulate.

Suppose you are asked for the 100,000th digit of the infinite string `123456789101112...`. Rather than building the string, you can count in bands. The numbers 1 through 9 contribute 9 digits. The numbers 10 through 99 contribute 90 × 2 = 180 digits. The numbers 100 through 999 contribute 900 × 3 = 2700. Subtracting these in turn tells you which band your target falls into, after which you can compute the exact number and the exact digit directly.

That is CSES *Digit Queries*, and CSES *Number Spiral* applies the same counting approach in two dimensions.

**(c) Apply many steps at once.** If a block of identical operations can be collapsed into a single computation, that removes the step count from the complexity entirely. Matrix exponentiation is the general tool for this and is covered in chapter [[14 Dynamic Programming Core]].

---

### Way 3 · The process is easier to run in the other direction

**The question to ask: would this be simpler backwards?**

The idea underneath this is that a process described in one direction does not have to be executed in that direction. You are free to choose whichever direction makes each individual decision unambiguous.

Backwards tends to help specifically when later steps cover up the evidence of earlier ones.

**Example.** You have a rubber stamp carrying the word `abc`. You start with a blank strip written as `?????` and stamp it repeatedly, with later stamps overwriting earlier ones. Given the final strip, you have to produce a sequence of stamps that would create it.

Running forwards is difficult because the first stamp is the one most likely to have been overwritten, so you cannot tell where it went, and every guess multiplies the search.

Running backwards is straightforward. Start from the finished strip and undo stamps: find a window matching `abc`, treating `?` as matching anything, blank that whole window back to `?`, and repeat until the entire strip is `?`. Reversing your list of moves at the end gives the forward answer.

The reason backwards works is that the most recent stamp is still fully visible, because nothing was placed on top of it. There is therefore always at least one move you can identify with certainty, and a greedy choice is safe. Forwards, uncertainty is at its highest at the beginning; backwards, it is zero at the beginning.

That is LC 936, and the same reversal appears in several places on this sheet. Deletions become insertions when time is reversed, which is how chapter [[10 DSU Advanced]] handles bricks being knocked out of a wall and floodwater receding. The question "which do I do first" becomes "which do I do last" in chapter [[14 Dynamic Programming Core]], which is what makes Burst Balloons tractable.

Two related direction changes are worth knowing:

- **Working outward from the middle.** When one element is guaranteed to be included, growing outward from it is often simpler than scanning inward from an end.
- **Processing in an order other than time.** Sorting events by size, by value, or by depth and sweeping through that order is sometimes far easier than following the sequence given.

---

### Using the three together

When a problem in this block resists you, these are the three checks to run:

| What you notice | Which kind it is | The question to ask |
|---|---|---|
| operations feel clumsy, loops appearing inside single steps | wrong shape | which layout makes my common operations cheap? |
| more steps than could possibly be executed | too many steps | which small summary of the situation repeats? |
| every forward choice requires a guess | wrong direction | is the *last* move identifiable? |

If none of the three applies, the problem is probably not hard in any structural sense, and the difficulty is coming from the requirement-tracking issues described in Parts 2, 3 and 6.

---

## Part 5 · Parsing

Parsing appears more often than anything else in this block, and it also has the most regular structure of anything here. Because that structure barely changes between problems, learning it once gives you a reusable approach to roughly six of the problems on the list.

### What the state looks like in a parser

Parsing means reading a string that carries meaning, such as `2 * (3 + 4)`, and working out what it denotes.

Applying the handover test: you are partway through reading `2 * (3 + 4 * (5 - 1))`, and your replacement needs to know where you are in the string, together with the partially computed total at every bracket level you are currently inside.

That second item is a stack, holding one entry per open bracket.

You could maintain that stack yourself. However, your program already contains a stack that grows as you go deeper and shrinks as you come back out, which is the call stack. Writing the parser as a set of functions that call each other means the language maintains the bracket-level bookkeeping on your behalf, and this is the practical reason recursion is the right tool for parsing rather than any question of elegance.

### Operator precedence comes from how the functions are layered

The structure is three functions that call one another:

```
expr   := term   ( ('+' | '-') term   )*
term   := factor ( ('*' | '/') factor )*
factor := NUMBER | '(' expr ')' | '-' factor
```

If the notation is unfamiliar, `( ... )*` means "zero or more of whatever is inside the brackets."

In plain terms: an expression is a series of terms added or subtracted together; a term is a series of factors multiplied or divided together; and a factor is a number, a bracketed expression, or a minus sign in front of another factor.

This layering is what produces correct operator precedence. Because `expr` calls `term`, and `term` consumes as many multiplication and division operations as it can before returning, the input `2 + 3 * 4` is automatically grouped as `2 + (3 * 4)`. You never need a precedence table or a separate algorithm, and introducing an operator that binds more tightly than multiplication is a matter of adding a fourth function below `factor`.

```cpp
struct Parser {
    string s;
    int i = 0;                       // current position. There is only one of these.

    void skip()      { while (i < (int)s.size() && s[i] == ' ') i++; }
    char peek()      { skip(); return i < (int)s.size() ? s[i] : '\0'; }
    bool eat(char c) { if (peek() == c) { i++; return true; } return false; }

    long long expr() {
        long long v = term();
        while (true) {
            if (eat('+'))      v += term();
            else if (eat('-')) v -= term();
            else return v;
        }
    }
    long long term() {
        long long v = factor();
        while (true) {
            if (eat('*'))      v *= factor();
            else if (eat('/')) v /= factor();
            else return v;
        }
    }
    long long factor() {
        if (eat('(')) { long long v = expr(); eat(')'); return v; }
        if (eat('-')) return -factor();          // leading minus sign
        if (eat('+')) return  factor();
        long long v = 0;
        while (isdigit(peek())) { v = v * 10 + (s[i] - '0'); i++; }
        return v;
    }
};
```

Two details in that code are worth adopting generally.

The position variable is a member of the struct, so there is exactly one of them. If you instead pass an index into a function and return an updated copy, you end up with two representations of where you are, and keeping them synchronised becomes an additional thing to get wrong.

The functions `peek` and `eat` are deliberately separate, with `peek` reporting what comes next and `eat` consuming it. Keeping inspection apart from consumption prevents the error where you accidentally consume a character you only meant to examine. Putting the whitespace handling inside `peek` also means that rule exists in exactly one location rather than being repeated wherever characters are read.

### The structure stays fixed while the values change

The most useful thing about this skeleton is how little of it varies between problems.

Compare four problems that look quite different on the surface:

- **LC 224** asks you to evaluate an arithmetic expression such as `2 * (3 + 4)`.
- **LC 726** asks you to read a chemical formula such as `K4(ON(SO3)2)2` and count each element.
- **LC 1096** asks you to expand `{a,b}{c,d}` into every string it can produce, giving `ac, ad, bc, bd`.
- **LC 770** asks you to evaluate an expression containing variables and return a formula.

The three functions are the same in all four. What differs is only the type of value a `factor` produces, and what the two operators do to values of that type.

| Problem | A factor produces | What "+" means | What "×" means |
|---|---|---|---|
| LC 224 Basic Calculator | a number | add the numbers | multiply the numbers |
| LC 726 Number of Atoms | a map of element to count | merge the maps, adding counts | multiply every count by `k` |
| LC 1096 Brace Expansion | a set of strings | union the sets, written as `,` | concatenate every pair |
| LC 770 Calculator IV | a polynomial | add the polynomials | multiply the polynomials |

Because of that, these are better treated as one problem with four settings than as four problems. When you meet a new expression-parsing question, the useful thing to work out first is what kind of value a factor should produce and what the operators do to it. Once you have those two answers, the skeleton can be filled in directly.

**LC 736 Parse Lisp** uses the same approach with the operators written in front of their arguments, as in `(add 1 2)` rather than `1 + 2`, and adds variable scoping. The scoping is simply one more piece of state: a stack of variable values, pushed when entering a `let` and popped when leaving it. The grammar has a different shape, but the method of working is unchanged.

---

## Part 6 · Checking your work

In an idea problem you can gain confidence by re-deriving the central idea and confirming it still holds. Detail problems offer no equivalent, since there is no single idea to re-derive, so confidence has to come from a routine you perform instead.

There are three techniques, listed from cheapest to most expensive.

**Turn the invariant into an assertion.** You already wrote it down in Part 2, so putting it into the code costs one line:

```cpp
assert(letters + (int)line.size() - 1 <= width);
```

The benefit is that a violation is reported at the step where it happens rather than at submission time, which usually identifies the faulty transition immediately.

**Point at the code that implements each requirement.** Take the requirement list from Part 3 and, for each entry, locate the specific line or function responsible for it. If you cannot find one, that requirement has not been implemented, and it is almost certainly the test you are about to fail. This pass takes around ninety seconds and reliably catches the case where an exception was read but never coded.

Then run the standard edge cases against those specific requirements rather than generically: a single item, empty input, all values identical, the largest permitted input, and any input where two requirements overlap.

**Compare against a deliberately slow version.** When you have a few spare minutes and real doubt, write the obvious solution that simply performs every step or tries every possibility, then compare the two on small random inputs.

```cpp
for (int t = 0; t < 10000; t++) {
    auto input = randomSmallInput();
    if (fast(input) != slow(input)) { print(input); break; }
}
```

This works particularly well in this chapter, because the slow version is correct by construction. It is the process itself, executed literally, so comparing against it is comparing against the specification. For problems where the difficulty was too many steps, the slow version is exactly the loop you were trying to avoid writing, which makes it almost free to produce.

Within a 35-minute budget you will not always reach the third technique. The first two together take under two minutes, so they are worth performing every time rather than treating as a judgement call.

---

## Part 7 · The method, in order

For any problem in this block:

1. **Read once for shape.** What is the process, and what does the answer look like?
2. **Read again and list the requirements.** Mark main rules, exceptions, and any conflicts between them. Keep the list in a comment.
3. **Run the handover test** to name your state, separating what you make decisions with from what you are accumulating.
4. **Write the invariant as a formula**, and derive your action trigger from it.
5. **Check the three ways it could be hard** — wrong shape, too many steps, wrong direction — and settle on your representation before writing code.
6. **Write the code with deciding and building kept apart**, using one loop to recognise units and separate functions to construct them.
7. **Assert the invariant, point at each requirement, and run the edge cases.**
8. Submit.

Steps 1 through 5 take five or six minutes, which feels like a significant portion of a 35-minute budget. The reason it is still worth doing is that those minutes are being traded against the time you would otherwise spend on the second, third and fourth submissions, which in practice costs considerably more.

---

## A full example: LC 726 Number of Atoms

Working through one problem completely is more useful than reading the method in the abstract, so this section applies all eight steps to a single question.

**The problem.** Given a chemical formula such as `K4(ON(SO3)2)2`, count how many atoms of each element it contains, and output them sorted by element name, giving `K4N2O14S4`.

---

**Step 1, the shape.** There is no process unfolding over time. The input is a nested structure that has to be read and totalled, so this is a parsing problem and Part 5 applies.

---

**Step 2, the requirement list.**

```
R1  An element is one uppercase letter followed by any number of lowercase letters.
R2  An element or a group may be followed by a number. Absence of a number means 1.
R3  Brackets group a sub-formula.
R4  A number after ')' multiplies everything inside the brackets.
R5  Items written next to each other are added together.
R6  Output is sorted by element name, and a count of 1 is written as nothing.
```

Six requirements, with no exceptions and no conflicts between them. That tells you in advance that the difficulty will be in choosing the representation rather than in reconciling rules, which is the opposite of a problem like LC 591 where the interactions between rules consume most of the time.

---

**Step 3, the handover test.** Partway through reading the formula, a replacement would need to know the current position in the string, and the counts collected so far at each bracket level currently open.

The second of those is a stack, and from Part 5 the recursion provides it. That leaves one explicit piece of state, the position.

---

**Step 4, the invariant.** For a recursive parser the invariant concerns the contract between the functions rather than a numerical relationship:

> **Every parse function leaves the position pointer immediately after the construct it has just read.**

This is worth writing down because most bugs in recursive parsers are a single function breaking this contract by one character, and the resulting failure appears somewhere unrelated.

---

**Step 5, which kind of hard is it?**

It is not a matter of too many steps, since there is a single pass over a short string, and it is not a direction problem, since reading left to right presents no ambiguity. It is a representation problem, and that is where the whole difficulty sits.

So the question from Part 5 applies: what kind of value should a factor produce? The answer is a map from element name to count. Once that is settled, the two operators follow. Requirement R5, items written next to each other, is the addition operator, and it means merging two maps while summing their counts. Requirement R4, a number following a group, is the multiplication operator, and it means multiplying every count in the map by that number.

One convenient consequence is that this grammar is simpler than arithmetic, because there is no addition symbol written anywhere in the input. Adjacency performs the addition, so two levels of function are sufficient rather than three.

---

**Step 6, the code.**

```cpp
using Counts = map<string,int>;

struct Parser {
    string s; int i = 0;

    // A run of factors, merged together. This is the "+" operator.
    Counts parse() {
        Counts result;
        while (i < (int)s.size() && s[i] != ')') {
            Counts f = factor();
            for (auto& [name, n] : f) result[name] += n;
        }
        return result;
    }

    // One element or bracketed group, scaled by its number. This is the "×" operator.
    Counts factor() {
        Counts cur;
        if (s[i] == '(') {
            i++;                 // consume '('
            cur = parse();
            i++;                 // consume ')'
        } else {
            string name(1, s[i++]);                              // R1
            while (i < (int)s.size() && islower(s[i])) name += s[i++];
            cur[name] = 1;
        }
        int mult = number();                                     // R2
        for (auto& [name, n] : cur) n *= mult;
        return cur;
    }

    int number() {
        int n = 0; bool found = false;
        while (i < (int)s.size() && isdigit(s[i])) { n = n*10 + (s[i++]-'0'); found = true; }
        return found ? n : 1;    // R2: no digits means 1
    }
};
```

The separation from Part 2 is visible here. The `parse` function decides where each factor begins and ends, `factor` constructs one, and `number` handles requirement R2 in a single place. Requirement R6 belongs to a separate output function that never touches the parser.

---

**Step 7, checking it.**

Each requirement can be pointed at: R1 is the `islower` loop, R2 is the `found` flag in `number`, R3 and R4 are the bracket branch together with the multiplication loop, R5 is the merge inside `parse`, and R6 is the output function. All six are accounted for.

The invariant holds if every function leaves the position immediately after what it read, and the bracket branch does so because it consumes the opening bracket, delegates, and then consumes the closing bracket.

The edge cases worth running are a single element with no number such as `H`, nested brackets such as `K4(ON(SO3)2)2`, and a multi-digit count such as `H12`, which the loop inside `number` handles.

---

What made this a fifteen-minute problem rather than a forty-minute one was step 5, where the value type was decided before any code was written. Starting to type with an integer in mind leads to discovering the problem around minute twelve, at which point most of what has been written has to be replaced. That is the practical reason representation is chosen early rather than discovered along the way.

---

## The ideas worth carrying forward

1. **These are detail problems.** The difficulty comes from the number of requirements rather than from any missing idea, so when you are stuck the productive response is to write things down rather than to think for longer.

2. **The handover test finds your state.** Asking what a replacement would need to know produces the right variables, and often the right data structure, without needing to recognise the problem type first.

3. **Separate decision state from accumulated output.** Information you use to make choices and material you are collecting for the answer are different things, and holding them in the same place is what makes small problems feel large.

4. **Write the invariant as a formula.** It supplies your action trigger for free, and it lets you settle `<` against `<=` by substituting the smallest case rather than by guessing.

5. **A bug here is a moment where the invariant became false without you noticing.** If the invariant was never stated, there is nothing concrete to check against, which is why these bugs are hard to locate.

6. **Keep deciding when a unit is complete apart from building the unit.** Exceptions in these problems are usually about how a finished unit is presented, so separating the two moves them into one place.

7. **Treat the statement as a requirement list.** Record each rule, mark whether it is a main rule or an exception, and note which takes priority when two of them overlap.

8. **There are three ways these get hard:** the data is in the wrong shape, there are more steps than you can run, or the process is easier in the other direction.

9. **When all activity happens at one moving position, consider a stack on each side of it.** This is the structure behind text editors, undo stacks, and browser history.

10. **Choose your representation before you type.** Changing it later means revisiting everything already written, which does not fit inside the time budget.

11. **Too many steps with a small world means something repeats.** Detecting the repeat is mechanical; choosing a summary small enough to recur but sufficient to predict the future is the part that requires judgement.

12. **Run the process backwards when later steps obscure earlier ones**, because the most recent step is always still visible and therefore unambiguous.

13. **A grammar needs a stack, and recursion provides it.** That is why recursive descent is the standard approach to parsing.

14. **Precedence comes from how the parsing functions are layered**, not from a precedence table, so adding a tighter-binding operator means adding a function below the existing ones.

15. **The three parsing functions do not change between problems.** Only the value type and the meaning of the two operators change.

16. **Use one position variable, and keep `peek` separate from `eat`**, so that inspecting a character cannot accidentally consume it.

17. **Verify with a routine rather than by re-deriving:** assert the invariant, locate the code for every requirement, and where time allows compare against a deliberately slow version.

---

## Where people lose these problems

**Starting to type before the requirement list exists.** This is the underlying cause of most of the entries below, because it is what allows requirements to go missing without being noticed.

**Keeping requirements in your head.** A statement with ten requirements exceeds what anyone can track while writing code, and the ones that drop out are usually the exceptions.

**Mixing decision state with collected output.** This inflates the apparent size of the problem and makes it harder to see which variables actually matter.

**Handling whitespace in several different places.** If the check appears more than once, one of the copies will eventually be missing. Centralising it in `peek()` removes the possibility.

**Building output inside the loop that decides when to flush.** The visible symptom is deep nesting, and the underlying cause is that two separate responsibilities have been merged.

**Missing the leading minus sign.** Inputs such as `-2+1` and `3-(-2)` account for a large share of Basic Calculator failures, because the case is easy to overlook when the examples do not include it.

**Integer overflow.** Intermediate values in expression evaluation can exceed 32 bits, so using `long long` throughout a parser is worth doing as a default since it costs nothing.

**Choosing the representation late.** By the time the problem becomes visible, most of the code depends on the choice, and there is no time to redo it.

**Omitting `seen.clear()` after detecting a repeat.** The loop then finds the same repeat again, computes a skip of zero, and makes no further progress.

**Not testing an awkward call order in design problems.** A grader will call something like `deleteText(1000)` on an empty editor, so any operation that takes a count needs to be bounded by what is actually available.

**Two exceptions occurring together.** A last line that also contains only one word satisfies both exceptions at once, and this combination is only checked if both rules were written down adjacently.

---

## Working through the problem list

The order below is arranged by what each problem teaches rather than by difficulty, so that each block exercises one part of the method. A short description of what each problem asks is included so that this section can be read before attempting anything.

### Block 1 · Building the routine on easy content

The ideas in these are straightforward, which makes them a good place to practise the process while nothing else is competing for attention.

- **CSES Palindrome Reorder** — *rearrange a string into a palindrome, or report that it is impossible.* A single counting rule. Write the condition down before coding even though it is short.
- **CSES Creating Strings** — *print all distinct rearrangements of a string.*
- **CSES Gray Code** — *list all n-bit numbers so that consecutive entries differ in exactly one bit.* The answer is `i ^ (i >> 1)`, but work it out from n = 1, 2, 3 before looking it up.
- **CSES Number Spiral** — *numbers are laid out in a spiral grid; report the value at a given row and column.* There is a formula, so no simulation is needed.
- **CSES Digit Queries** — *find the k-th digit of the string 123456789101112...* This is the counting-in-bands technique from Part 4.

### Block 2 · State and the deciding/building split

- **LC 388 Longest Absolute File Path** — *given a tab-indented file listing, find the length of the longest full path.* The handover test produces a structure indexed by depth. Note that `\t` is a single character rather than four spaces.
- **LC 68 Text Justification** ✔ — *the word-packing problem used as the running example in this chapter.*
- **LC 65 Valid Number** ✔ — *decide whether a string is a valid number, allowing signs, decimal points and exponents.* Worth attempting as an explicit state machine: draw the states as circles and the characters as arrows, then write the transition table. The twelve special cases collapse into a single table lookup, which demonstrates directly how writing the state down eliminates cases.
- **LC 591 Tag Validator** — *decide whether a string of custom XML-like tags is valid.* Around ten interacting rules, where enumerating them is the substance of the solution.

### Block 3 · Parsing, changing one thing at a time

These are best done in order, since each changes exactly one aspect of the previous one.

- **LC 224 Basic Calculator** ✔ — *evaluate an expression containing `+`, `-` and brackets.* If your original solution used a hand-built stack, rewriting it with the three-function skeleton is a useful comparison.
- **LC 640 Solve the Equation** — *solve an equation such as `x+5-3+x=6+x+2` for x.* The value type becomes a pair holding the coefficient of x and the constant, which is the smallest possible change from the arithmetic version.
- **LC 726 Number of Atoms** — *count the elements in a chemical formula.* The value type becomes a map. This is the problem where the pattern becomes visible, so it is worth doing carefully. It is worked through in full above, but attempt it first.
- **LC 1096 Brace Expansion II** — *expand an expression such as `{a,b}{c,d}` into every string it can produce.* The value type becomes a set of strings, where the comma is union and adjacency is concatenation.
- **LC 736 Parse Lisp Expression** — *evaluate a Lisp-style expression using `let`, `add` and `mult`.* Operators come before their arguments, and variable scoping adds one stack.
- **LC 770 Basic Calculator IV** — *evaluate an expression containing variables and return a formula.* The value type becomes a polynomial. Worth attempting only once 726 and 1096 feel routine.

### Block 4 · Too many steps

- **LC 481 Magical String** — *a string that describes its own run lengths; find the n-th character.* Generating it requires trusting a rule whose consequences you cannot yet see in full.
- **LC 466 Count The Repetitions** — *how many times does one string appear inside another string repeated n times?* The repeat-detection template, and more importantly the question of what to record.
- **CF 1560E Polycarp and String Transformation** — *given the output of a letter-removal process, reconstruct the original string.* Deduce the process from its result, then confirm by simulating.

### Block 5 · The wrong direction

- **LC 936 Stamping The Sequence** — *the rubber stamp problem from Part 4.* The reasoning transfers to more problems than the question itself.
- **LC 899 Orderly Queue** — *repeatedly move one of the first k characters to the end, and find the smallest string reachable.* With `k ≥ 2` any arrangement is reachable, so the answer is the sorted string; with `k = 1` only rotations are possible, so all n rotations can be tried. The short answer behind a complicated statement is a reminder to read statements for what they actually permit.

### Block 6 · The wrong data shape

- **LC 2296 Design a Text Editor** — *the two-stack cursor described in Part 4.*
- **LC 761 Special Binary String** — *rearrange a balanced binary string to be lexicographically largest.* Decompose into balanced blocks, sort them, and recurse. Genuinely difficult and best treated as a stretch problem.
- **LC 2019 The Score of Students Solving Math Expression** — *given incorrect student answers, determine which could result from evaluating the expression in a valid but wrong order.* Combines parsing with interval DP over sets of reachable values, so it is best attempted after chapter [[14 Dynamic Programming Core]].

---

**The target for this block is more than 80% of submissions passing on the first attempt.**

That figure is realistic rather than aspirational, because these problems do not contain ideas you are missing. A result below it indicates a gap in process rather than in knowledge, which means that solving additional problems will not close it on its own. Re-running the block while applying Part 7 strictly is the more effective response.

**What to record in your log.** For each failure, note which step of Part 7 would have prevented it. If four entries point at step 2, the conclusion is that you are not writing the requirement list, which is something you can change directly. A note saying that you make careless mistakes does not offer the same.

---

## Check yourself

Answer these out loud before moving on.

1. What distinguishes an idea problem from a detail problem, and why does that change what you should do when stuck?
2. State the handover test, then apply it to this task: group a stream of numbers into batches whose total is at most 100. What is the state, and what is not?
3. Why does writing the invariant as a formula settle the choice between `<` and `<=`?
4. Complete this sentence: a bug in this kind of code is a moment where ___.
5. What is the deciding/building split, and which problem shows it moving the exceptions out of the main loop?
6. What three things do you record against each requirement when reading the statement?
7. Name the three ways these problems become hard, along with the question you ask to identify each one.
8. All the activity in a problem happens at one moving position. Which representation should you consider?
9. A billion steps over a small world. Which part of the technique requires judgement, and why?
10. When does running the process backwards help, and what makes each backwards move unambiguous?
11. Why is recursion the appropriate tool for parsing, in terms of what it keeps track of?
12. What is the only thing that differs between LC 224, LC 726 and LC 1096?
13. Name the three ways to check your work, and say which two are worth doing every time.
