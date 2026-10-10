# longest common prefix: stop guessing on LeetCode 14 — the vertical scan, the edge cases that break it, and how to drill the pattern before your next interview

Most people typing "longest common prefix" into a search box want one of two things. Either they're staring at LeetCode 14, which is tagged Easy, and their submission just failed on test case 3 of 127. Or they've hit the term somewhere else — a trie problem, an autocomplete feature, a "find the common prefix of these two arrays of numbers" variant — and they want the general idea, not a single solution.

This walks through both. The algorithm is small enough to hold in your head in about five minutes; the reason people still lose points on it is boundary handling, not cleverness. At the end there's a section on where the pattern fits into a structured prep curriculum, because "I can solve it once" and "I can reach for the right approach in 90 seconds under pressure" are different skills.

## What the term actually means

Given an array of strings, the longest common prefix is the longest string that appears at the start of every string in the array. If the first characters don't match, the answer is the empty string.

Two examples cover most of it:

- `["flower", "flow", "flight"]` → `"fl"`
- `["dog", "racecar", "car"]` → `""`

The LeetCode version constrains things tightly: 1 to 200 strings, each 0 to 200 characters, lowercase English letters only. Your solution shouldn't lean on the lowercase alphabet, though — real interviewers often tweak the constraint to see whether you assumed it.

Three properties do most of the work when you're reasoning about a solution:

1. The answer can never be longer than the shortest string in the array.
2. The answer never gets longer as you process more strings. It only shrinks or stays put.
3. Once it's empty, it stays empty. You can bail out immediately.

That second property is what makes horizontal scanning feel natural, and the third is what saves you from pointless work.

## The approaches worth knowing

There are more than four ways to solve this, and listing all of them is a waste of your time. These are the ones that come up in interviews.

### Vertical scanning: the default

Pick the first string as a reference. Walk through its characters by index. For each index `i`, check every other string: does it have a character at that position, and is it the same character?

The first time a string runs out of characters or disagrees, you return `strs[0][:i]`. If you get all the way through, the first string *is* the answer.

Why this is the default: it stops at the first disagreement across all strings simultaneously, and the early-exit condition is trivial to state out loud. AlgoMonster's own editorial for problem 14 uses exactly this method, with the first string as reference and a bounds check before every comparison.

### Horizontal scanning: shrinking a candidate

Start with `prefix = strs[0]`. For each following string, keep chopping the last character off `prefix` until the string starts with it. If `prefix` hits empty, return `""`.

Some learners find this easier to narrate because the variable visibly shrinks. It's also slightly easier to get subtly wrong — people forget the empty-prefix early return, and they don't guard against the candidate being shorter than the shortest string.

### Sort, then compare only the two ends

Sort the array. Compare `strs[0]` with `strs[-1]` character by character, and the shared beginning of those two is the answer for the whole array.

This looks like magic the first time. The argument is that after lexicographic sorting, the first and last strings are the most different pair in the set — so if they still share a prefix, every string sandwiched between them must share it too.

It's a good thing to mention if an interviewer asks for alternatives, and a bad thing to actually submit when the question is a single query, because you pay `O(N log N)` comparisons for nothing. It becomes attractive only if the same array gets queried repeatedly in a context where the sorting is already done.

### Trie: the version that scales past one array

Build a trie of all the strings, then walk from the root while the current node has exactly one child and isn't a terminal node. The path you walked is the prefix.

For a one-shot LeetCode 14 submission, a trie is overkill. It earns its place when the question stops being "solve this array once" and becomes "answer prefix queries on a growing set," or when the input is structured so a trie maps naturally — the numeric variant where you're asked for the *length* of the longest common prefix between arrays of integers is the standard example. Same idea, different data structure.

## Complexity, with actual numbers

Vertical and horizontal scanning both run in `O(N · M)` time, where N is the number of strings and M is the length of the shortest string (or of the prefix, depending on how you count the early exits). They use `O(1)` extra space beyond the string you return.

The sort approach is `O(N log N · M)` comparisons plus an `O(M)` scan, so it's strictly worse than scanning for a single call, and it mutates the input array unless you copy it — worth mentioning out loud, because interviewers notice when you quietly destroy the caller's data.

The trie builds in `O(total characters)` and walks in `O(M)`, with `O(total characters)` space.

If someone asks whether an `O(N · M)` answer is good enough here: with 200 strings of at most 200 characters, the worst case is 40,000 character comparisons. It's fine. Optimizing past that is solving a problem that doesn't exist.

## The edge cases that actually fail submissions

This is where most wrong answers live, and where an interviewer learns more about you than from the algorithm itself.

**Any string is empty.** `[""]` should return `""`. `["abc", ""]` should also return `""`. If you index into the first string blindly, this blows up or returns garbage.

**The shortest string is the entire answer.** `["abc", "ab"]` returns `"ab"` — not `"abc"`, not `"a"`. Your loop has to stop when it runs out of characters in *any* string, not just the reference.

**No match at index zero.** `["dog", "racecar", "car"]` returns `""` before you've compared anything meaningful. Return early instead of grinding through the rest.

**One string.** `["solo"]` returns `"solo"`. The whole first string is trivially a prefix of itself.

**The empty array.** LeetCode's constraints say at least one string, so this is technically out of scope. Interviewers ask anyway. Return `""` and say why.

**Reading past the end of a shorter string.** The single most common bug: comparing `first[i] != s[i]` without first checking `i < len(s)`. In languages with bounds checking you get an exception; in C++ you get undefined behavior, which is worse because it might pass.

A quick mental checklist before you say you're done: does it handle the empty string, does it handle unequal lengths, does it return on the first mismatch, and does it work when the answer is the entire shortest string?

## The code, vertical scanning

Python, written so the guards are obvious:

python
class Solution:
    def longestCommonPrefix(self, strs: list[str]) -> str:
        if not strs:
            return ""
        for i, ch in enumerate(strs[0]):
            for s in strs[1:]:
                if i == len(s) or s[i] != ch:
                    return strs[0][:i]
        return strs[0]


Java:

java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        if (strs.length == 0) return "";
        for (int i = 0; i < strs[0].length(); i++) {
            char c = strs[0].charAt(i);
            for (int j = 1; j < strs.length; j++) {
                if (i == strs[j].length() || strs[j].charAt(i) != c) {
                    return strs[0].substring(0, i);
                }
            }
        }
        return strs[0];
    }
}


C++:

cpp
class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        for (int i = 0; i < (int)strs[0].size(); i++) {
            for (int j = 1; j < (int)strs.size(); j++) {
                if (i == (int)strs[j].size() || strs[j][i] != strs[0][i])
                    return strs[0].substr(0, i);
            }
        }
        return strs[0];
    }
}


All three are the same eleven lines with different syntax. If you can write one, you can write the other two, which is exactly the point that AlgoMonster builds its curriculum around: template once, translate across languages.

If you want to see the same problem annotated with per-line reasoning in Python, Java, and C++, 👉 [read AlgoMonster's editorial for LeetCode 14](https://algo.monster/liteproblems/14?fpr=li69) — it uses the vertical scan and calls out the bounds check explicitly.

## Why interviewers keep asking this

LeetCode 14 is one of the highest-frequency easy problems in interview loops, and it isn't because the algorithm is impressive. There's nothing impressive about it.

It's asked because it compresses four separate signals into ten minutes:

- **Do you check bounds before indexing?** This separates people who have shipped code from people who have only passed happy-path tests.
- **Do you explain a loop invariant?** "The prefix can only shrink" is a one-sentence justification that makes your whole solution obviously correct.
- **Do you know when to stop optimizing?** `O(N · M)` with N, M ≤ 200 is fine, and saying so out loud is a stronger signal than proposing a trie nobody asked for.
- **Can you move between languages?** Follow-ups like "write it in Java" are common when the job listing names a stack you didn't use.

The honest framing: this problem is a boundary-handling test wearing an algorithm costume. Candidates who've done 200 random problems without a system tend to get it *almost* right — right approach, one missing guard. Candidates who've learned the pattern under a name ("prefix scanning," "vertical traversal") tend to get it right the first time, because the guard is part of the template, not an afterthought.

## Practicing the pattern instead of the problem

Here's the trap with problems like this. You read the solution, you nod, you move on, and three weeks later in a phone screen you write `s[i]` without the length check. Recognition and recall are different skills, and grinding 500 random problems doesn't build either one efficiently — it builds familiarity with 500 specific answers.

AlgoMonster takes the opposite approach. Instead of a problem bank, it's organized around named patterns: each pattern gets a conceptual explanation, a code template in multiple languages, and a set of mapped problems where the template applies with a twist. String scanning and prefix-traversal problems get grouped together, so the bounds-check habit comes from the template rather than from memorizing a specific test case. Company question sets are refreshed monthly, and there's a Speedrun mode for timed review once you know the patterns and just need speed.

The tradeoffs are worth stating plainly, since several reviewers have flagged them:

- **Text and diagrams, no video.** 1,000+ illustrations across the lessons, but zero video walkthroughs. If you learn by watching someone code, this format will feel dry.
- **Smaller problem bank than LeetCode.** The lessons and problems are curated and mapped rather than vast. Most reviewers suggest pairing it with LeetCode for volume once the patterns are in place.
- **System design coverage is introductory.** Fine for junior and mid-level loops, not enough for senior rounds where system design is its own 45-minute interview.
- **No live mock interviews.** It's self-paced with no human feedback loop.

Where it does well is the thing this article is about: turning a scattered set of "easy" problems into a small number of named, reusable templates you can recall under pressure.

## AlgoMonster pricing: every plan on the page

AlgoMonster doesn't bill monthly. It's annual or one-time lifetime, plus a free tier you can use to judge whether the text-and-diagram format works for you before paying anything.

| Plan | What's included | Price | Billing | Get it |
| --- | --- | --- | --- | --- |
| **Free** | Limited sample access to patterns, lessons, and problems | $0 | No card required | [Start free](https://algo.monster?fpr=li69) |
| **Pro – Yearly** | All patterns, lessons and problems unlocked; unlimited AI assistant; unlimited Speedrun; real questions from top tech companies, updated monthly; 500+ lessons across 6 courses, 1,000+ illustrations, 3,000+ editorials; member discount on live bootcamps | $99/year ($8.25/mo equivalent) | Billed annually, cancel or upgrade anytime | [Get Pro Yearly](https://algo.monster/subscribe?fpr=li69) |
| **Pro – Lifetime** | Everything in the yearly plan, plus lifetime updates with no renewal | $189 | One-time payment | [Get Pro Lifetime](https://algo.monster/subscribe?fpr=li69) |

Two things to know before you check out.

First, the numbers move. AlgoMonster runs continuous promotions, and the subscribe page currently shows the plans at roughly half the listed original prices ($198/year and $378 lifetime). The exact total at checkout depends on the promotion that's live and the link you arrive through — the yearly plan has been displayed at both $99 and lower. Look at the actual number on the page rather than trusting a blog post's screenshot, including mine.

Second, coupon codes float around. A discount page run by a creator circulates a code that reportedly stacks an extra 10% on the current sale, and other sites name codes like `FAANG20`. I can't verify that any specific code still applies to your cart, so treat them as worth typing in and nothing more.

Two extras worth noting from the plans page: the subscription bundles a Foundation Course and a "Build an LLM" course at no extra cost, and some specialized courses developed with outside partners — Quant Interview Prep is the example they give — are sold separately.

## Yearly or lifetime?

The arithmetic is simple once you decide how long you'll care about this.

**Yearly at $99** is the right call if you have a loop starting in the next month or two. You'll use the platform intensively for four to eight weeks, then stop. Paying once and walking away costs less than committing to lifetime pricing.

**Lifetime at $189** costs less than two years of the annual plan. If you expect to interview again in two or three years — a promotion cycle, a job hop, a team that reorganizes — the lifetime tier is cheaper than renewing twice, and updates keep arriving. Reviewers make the same argument, and the math holds as long as you actually come back.

**The free tier** is not a trial in disguise. It's a genuinely limited slice of the content, enough to tell you whether you can learn from text and diagrams or whether you'll bounce off it. Spend a day there before paying. That's a better use of your time than reading a fifth comparison article.

One caveat before you buy lifetime: if your gap is system design or behavioral prep, no AlgoMonster plan fixes it. You'd be paying for a solution to a problem you don't have.

## FAQ

**What's the time complexity of longest common prefix?** `O(N · M)` for scanning, where N is the number of strings and M is the length of the shortest string. Extra space is `O(1)` not counting the string you return.

**Can the answer be longer than the shortest string?** No. The shortest string is an upper bound on the length, and a mismatch usually makes it shorter still.

**Is sorting faster?** No. Sorting the array adds `O(N log N)` string comparisons on top of the final scan. It's a neat observation to raise when asked for alternatives, not an optimization.

**What should the function return for an empty array?** `""`. It's outside LeetCode's stated constraints, but interviewers ask, and returning empty is the only answer that doesn't crash on the next line.

**Does the algorithm care about uppercase or Unicode?** Not really — the logic is character equality, and it works for any encoding as long as comparisons are consistent across strings. LeetCode just restricts the input to lowercase `a-z`.

**Is a trie the "best" solution?** Not for one query. A trie costs more to build than a scan costs to run. It's the right tool when the same string set gets queried repeatedly or when the input isn't plain strings.

## Where to go next

Write the vertical scan from memory, in whatever language you'd actually interview in. Then write it again without looking. Then deliberately break it four ways — empty string, empty array, unequal lengths, no common first character — and fix each break.

That's the whole problem. It's thirty lines of thinking and about ten minutes of practice, and it's worth doing today rather than the week before your loop, because the skills it tests (bounds guards, loop invariants, knowing when to stop) show up in every other array and string question you'll get.

If you'd rather practice it as part of a template you can reuse, 👉 [browse the pattern-based curriculum on AlgoMonster](https://algo.monster/subscribe?fpr=li69) and start with the free tier to see whether the format clicks. If it doesn't, you've lost an afternoon. If it does, you've got a template for the next twenty string problems.
