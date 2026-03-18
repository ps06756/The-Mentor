# Example: Coding Interview (Arrays & HashMaps)

A realistic transcript showing the AI interviewer pushing beyond the first solution.

---

**Interviewer:** You're building a leaderboard system. Should you store scores in an array or a HashMap? What changes if you need the top 10?

**You:** HashMap, because you can look up any player's score in O(1).

**Interviewer:** Good for lookup. But how do you get the top 10 from a HashMap? Can you do that in O(1)?

**You:** No... I'd have to sort the values. That's O(n log n).

**Interviewer:** Right. So a HashMap is great for point lookups but bad for ranking. What data structure gives you *both*? Keep that thought. Let's move to a problem.

Here's your problem: Given an array of strings, group the anagrams together. For example, `["eat","tea","tan","ate","nat","bat"]` should return `[["eat","tea","ate"],["tan","nat"],["bat"]]`.

What's your plan?

**You:** I'll sort each string and use the sorted version as a key in a HashMap. All anagrams produce the same sorted string.

**Interviewer:** Walk me through the complexity before you code.

**You:** For each of the n strings, sorting takes O(k log k) where k is the string length. So total is O(n * k log k).

**Interviewer:** Good. Now code it up.

**You:**
```python
def group_anagrams(strs):
    groups = {}
    for s in strs:
        key = ''.join(sorted(s))
        if key not in groups:
            groups[key] = []
        groups[key].append(s)
    return list(groups.values())
```

**Interviewer:** Would you ship this? Two follow-ups. First — can you do better than O(k log k) per string? Sorting is convenient but not optimal. Second — what happens if the strings contain Unicode characters, not just lowercase English letters?

**You:** For the first... instead of sorting, I could count the frequency of each character and use that as the key?

**Interviewer:** Yes. That brings it down to O(n * k). Now, would you *actually* do that in production, or is the sorting approach fine? Think about readability vs performance.

**You:** In production, if k is small (like words in a dictionary), the sorting approach is more readable and the performance difference doesn't matter.

**Interviewer:** That's the answer I was looking for. Knowing the optimization *exists* but choosing readability when it doesn't matter — that's engineering judgment. Now let's move to something harder.

---

**Scorecard:**

| Area | Rating | Notes |
|------|--------|-------|
| Problem Understanding | Expert | Immediately identified the key insight (sorted string = anagram signature) |
| Solution Approach | Intermediate | Found O(n*k log k) but needed prompting for O(n*k) optimization |
| Code Quality | Intermediate | Clean and correct, could use defaultdict for conciseness |
| Edge Cases | Novice | Didn't consider Unicode or empty strings unprompted |
| Communication | Expert | Articulated complexity analysis and engineering trade-offs |
