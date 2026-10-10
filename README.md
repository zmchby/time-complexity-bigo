# How to Calculate Time Complexity: Count Loops, Solve Recurrences, and Get Big-O Right in Interviews

Most people who search for this aren't confused about what Big-O means. They know `O(n log n)` is better than `O(n²)`. What they're stuck on is the mechanical part: someone shows you a loop, a recursive function, or a chunk of code with a hidden `sort()` call somewhere, and asks "what's the time complexity?" — and you're supposed to produce an answer in under a minute, out loud, while someone watches.

That's a different skill from reading a definition. Below is the working method: how to pick `n`, how to count the work, how to handle recursion, and the specific places where people get it wrong even when they understand the theory.

## Why you can't just measure seconds

Run the same function on a laptop and a phone and you'll get different numbers. Run it twice on the same machine and cache behaviour will change the result. Hardware, compiler, and runtime all shift the wall-clock time, which makes it useless for comparing algorithms.

So analysis uses two simplifications that have held up for decades:

1. Express the work as a function of the **input size**, usually called `n`.
2. Count **basic operations** — comparisons, arithmetic, assignments, array accesses — rather than real time.

You end up with something like `T(n) = 3n² + 7n + 23`. From there you drop the lower-order terms and the constant multiplier, and you're left with `O(n²)`. The reason this isn't cheating: for large `n`, multiplicative constants stop mattering. `100n` beats `n²` once `n` passes 100, and the gap only widens from there.

That's the whole idea. Everything below is the execution.

## Step 1: Decide what `n` actually is

Most wrong answers start here. "This is `O(n)`" means nothing until you say what `n` counts.

| Problem type | Natural size parameter |
| --- | --- |
| Searching or iterating an array | number of elements |
| Multiplying two `n × n` matrices | the matrix dimension (there are `n²` entries) |
| Processing a string | number of characters |
| Graph traversal | number of vertices `V` and edges `E` |
| Integer arithmetic | number of bits, not the numeric value |

The matrix case is the trap. A triple nested loop over an `n × n` matrix is `O(n³)`, not `O(n)` — because `n` was defined as the dimension, and the loop runs `n³` times. Say the parameter out loud in an interview and half the ambiguity disappears.

## Step 2: Count the work

For iterative code, the count is mostly about loops. Read the loop, figure out how many times the body runs, multiply by the cost of the body.

| Code shape | Complexity | Why |
| --- | --- | --- |
| Straight-line statements | `O(1)` | Fixed number of steps regardless of input |
| Single loop over `n` items | `O(n)` | Body runs `n` times |
| `while (n > 1) n = n / 2` | `O(log n)` | Halving until you hit the base |
| `for (j = 1; j < n; j *= 2)` | `O(log n)` | Doubling is the same count in reverse |
| Loop over `n`, inner loop over `n` | `O(n²)` | Iterations multiply |
| Outer `n`, inner runs `i+1 … n` | `O(n²)` | `n(n-1)/2` still scales as `n²` |
| Outer `n`, inner `log n` | `O(n log n)` | Multiplication again |
| `n` iterations, each doing an `O(n)` operation | `O(n²)` | The hidden-cost case |

The last row is where beginners lose points. Consider:

java
for (int i = 0; i < n; i++) {
    if (list.contains(target)) { ... }   // O(n) scan for ArrayList
}


The loop looks linear. It isn't — each `contains` walks the list, so the real answer is `O(n²)`. Same problem shows up with `sublist` or slicing inside a loop, string concatenation in a loop, and `sort()` called inside an iteration.

For a loop that doesn't obviously count to `n` or `log n`, ask: *how many times does the counter change before the loop condition fails?* That number is the iteration count, and the body's cost multiplies it.

## Step 3: Know where the ladder becomes unusable

Once you have the complexity class, it tells you what input sizes are even feasible. These thresholds come up constantly in interview constraints — and on AlgoMonster's runtime overview lesson, which maps each class to the coding patterns that produce it.

| Complexity | Typical `n` limit | Common source |
| --- | --- | --- |
| `O(1)` | any | Hash lookup, array index, arithmetic |
| `O(log n)` | very large | Binary search, halving loops |
| `O(n)` | up to ~10⁷–10⁸ | Single pass, prefix sums |
| `O(n log n)` | ~10⁶ | Sorting, divide and conquer |
| `O(n²)` | ~3000 | Nested loops, pairwise comparison |
| `O(2ⁿ)` | ~20 | Subsets, naive recursion with two branches |
| `O(n!)` | ~12 | Permutations, brute-force ordering |

A quadratic solution isn't automatically wrong. For `n` under 1000 it will pass most tests on modern hardware. The issue is that interviewers expect you to notice when something better exists — which is usually the entire point of the question.

## Step 4: Recursion — write the recurrence first

Recursive functions don't have a loop count, so you write a recurrence instead. The general divide-and-conquer shape is:


T(n) = a·T(n/b) + f(n)


where `a` is how many recursive calls you make, `n/b` is how much the problem shrinks each time, and `f(n)` is the work outside the recursive calls (splitting, merging, or nothing).

Then you solve it. Three methods show up in coursework, and the recursion tree is the one worth internalising because it lets you *see* the answer:

- **Recursion tree**: draw the calls, compute the work at each level, multiply by the number of levels.
- **Substitution**: guess the bound, prove it by induction.
- **Master Theorem**: apply the formula when the recurrence matches the required shape.

Merge sort is the canonical tree example. At every level, the total work across all subproblems is `c·n`. The tree has `log₂ n` levels before subproblems hit size 1. Multiply: `Θ(n log n)`. No algebra required, just level cost × height.

Naive recursive Fibonacci, `T(n) = T(n-1) + T(n-2) + O(1)`, branches into roughly two calls per node with almost no work at each one. That's `O(2ⁿ)`, which is why the memoised version drops to `O(n)`.

Recurrences worth recognising on sight:

| Recurrence | Solution | Where it appears |
| --- | --- | --- |
| `T(n) = T(n/2) + O(1)` | `O(log n)` | Binary search |
| `T(n) = T(n-1) + O(1)` | `O(n)` | Linear recursion, factorial |
| `T(n) = 2T(n/2) + O(n)` | `O(n log n)` | Merge sort |
| `T(n) = 2T(n-1) + O(1)` | `O(2ⁿ)` | Subsets, naive branching |
| `T(n) = T(n-1) + O(n)` | `O(n²)` | Quicksort worst case |
| `T(n) = 3T(n/2) + O(n)` | `O(n^1.585)` | Karatsuba multiplication |

## Master Theorem in three lines

If `T(n) = a·T(n/b) + O(n^d)`, compare `log_b a` to `d`:

- `log_b a < d` → `T(n) = O(n^d)`. The non-recursive work dominates.
- `log_b a = d` → `T(n) = O(n^d log n)`. The two are comparable, and you pay a log factor.
- `log_b a > d` → `T(n) = O(n^(log_b a))`. The recursion dominates.

Binary search: `a = 1`, `b = 2`, `d = 0`. Since `log₂ 1 = 0 = d`, that's case two — `O(log n)`.

Merge sort: `a = 2`, `b = 2`, `d = 1`. `log₂ 2 = 1 = d`, case two again — `O(n log n)`.

`T(n) = 3T(n/4) + cn²`: `log₄ 3 ≈ 0.79`, which is less than `d = 2`, so case three applies and the answer is `O(n²)`. The recursion tree shows why: each level does `3/16` of the work above it, so the series converges and the root dominates.

One caution the theorem's own conditions make explicit: it needs `f(n)` to be polynomial. Something like `T(n) = 2T(n/2) + 2ⁿ` doesn't fit, and you fall back to the tree or substitution method.

## Amortized time: one expensive step, spread out

Some operations are cheap almost always and expensive occasionally. Appending to a dynamically sized array is the standard case: most appends are `O(1)`, but when the array fills up, it copies everything into a bigger buffer — `O(n)` for that single call.

Because the resize happens rarely and the buffer doubles, the *average* cost per append stays constant. That's what amortized `O(1)` means: the total cost of `n` appends is `O(n)`, so each one amortizes to `O(1)`, even though one individual call can be expensive.

The distinction matters in interviews. If you say "worst case `O(n)`" for `push_back`, you've answered a different question than the one being asked. Amortized is about the pattern over many operations, not any single one.

## Six mistakes that cost the most points

**Adding instead of multiplying nested iterations.** Sequential loops add (`O(n) + O(n) = O(n)`); nested loops multiply.

**Leaving two terms in the answer.** If the question is "what's the time complexity," `O(n² + n log n)` is not an answer. `O(n²)` is.

**Analysing the best case.** Big-O describes an upper bound, so assume the worst input unless told otherwise. A loop whose inner count depends on a value found in the data is treated as running the full `n` times.

**Missing the work inside library calls.** Sorting inside a loop, `in` checks on a list, slicing, or string building are all real costs. Know what your standard library actually does.

**Ignoring space.** Recursive solutions carry a stack. Depth `n` means `O(n)` space, even if the function itself allocates nothing.

**Treating the log base as significant.** `log₂ n` and `log₁₀ n` differ by a constant factor, so both are `O(log n)`. The base never changes the complexity class.

## Where practice actually changes the outcome

There's a gap between understanding this article and being able to produce an analysis in real time under pressure. Reading a worked example feels like learning; doing it yourself from scratch on unfamiliar code is what builds the reflex.

That's essentially the training loop AlgoMonster is built around: its problems ask you to identify the pattern, sketch the approach, implement it, and then state the time and space complexity — the last step is part of the exercise, not an afterthought. The platform keeps a dedicated runtime overview lesson with practice questions on exactly the categories above (nested loops, exponential recursion, amortized cost), plus editorials that walk through complexity analysis alongside the solution.

It's text and diagrams rather than video, which outside reviews consistently flag as the main trade-off — good if you like reading and working through code, less good if you learn primarily by watching someone else solve problems. Curriculum length is designed for roughly 4–8 weeks at 1–2 hours a day.

👉 [Start with the free AlgoMonster content and work through the runtime lesson yourself](https://algo.monster?fpr=li69)

## AlgoMonster plans and what they cost

AlgoMonster runs on two paid plans and a free tier. There's no monthly option and no team or enterprise plan — it's built for individuals preparing on their own. Checkout currently shows a 50% promotional rate on both paid plans; third-party deal trackers list the pre-discount prices as $198/year and $378 one-time.

| Plan | What you get | Price | Billing | Get it |
| --- | --- | --- | --- | --- |
| Free | Limited access to a subset of patterns and problems — enough to test whether the text-and-diagram format works for you | $0 | — | [Try the free tier](https://algo.monster?fpr=li69) |
| Pro — Yearly | All patterns, lessons and problems unlocked · Unlimited AI assistant · Unlimited Speedrun · 500+ lessons across 6 courses, 1,000+ illustrations, 3,000+ editorials · Real interview questions from top tech companies, updated monthly · Member discount on live bootcamps · Bonus Foundation and Build-an-LLM courses · 12 months of access | $99/year ($8.25/month billed annually) | Annual, cancel or upgrade anytime | [See the yearly plan](https://algo.monster?fpr=li69) |
| Pro — Lifetime | Everything in the annual plan · Lifetime content updates · One-time payment, no renewal | $189 | One-time | [Check the lifetime plan](https://algo.monster?fpr=li69) |

A few details worth knowing before you buy. Partner-developed courses such as Quant Interview Prep are sold separately. The free tier is a preview rather than a free trial — there's no time limit on it, but it doesn't cover the full curriculum. And the 50% rate is promotional, which means the $99 and $189 figures aren't guaranteed to stay put if you're planning to buy later.

Which one makes sense depends on how many job searches you expect. If you're prepping for one interview cycle over the next couple of months, the yearly plan covers it with money left over. If you expect to switch jobs every two or three years, lifetime pays for itself at the second search. The break-even sits at roughly two years of yearly pricing.

👉 [Compare both plans on the AlgoMonster pricing page](https://algo.monster?fpr=li69)

## Quick answers to the questions that come up

**Does the programming language change the complexity?** No. The same algorithm has the same Big-O in Python or C++. What changes is the constant factor and what counts as a single operation.

**What's the difference between O, Θ, and Ω?** `O` is an upper bound, `Ω` is a lower bound, `Θ` is both at once — a tight bound. In interviews, "what's the time complexity" almost always means the tightest upper bound you can justify.

**How do I get faster at this?** Do the analysis on code you've already written and know the answers to, before checking. The bottleneck isn't conceptual understanding, it's speed of recognition, and that only comes from repetition on problems where you state the complexity out loud rather than skimming a solution.
