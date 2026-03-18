# Problem Bank: Problem Decomposition

These problems are intentionally NOT from the LeetCode Top 100. Each walkthrough demonstrates the PROCESS of solving, not just the final answer. Every walkthrough follows the Problem-Solving Funnel: Understand -> Plan -> Code -> Test.

---

## Problem 1: Parking Lot Spot Assignment

**Level**: SWE-I | **Patterns Tested**: OOP design, constraint satisfaction, heap
**Why This Problem**: Looks like a design question but is really about choosing the right data structure. Candidates who jump to code will miss critical requirements.

### Statement

Design a parking lot system:
- Spots come in three sizes: small, medium, large.
- Vehicles come in three sizes: motorcycle (any spot), car (medium or large), bus (large only).
- Assign the smallest valid spot with the lowest spot number.
- Implement: `park(vehicle) -> spot_number or -1` and `leave(spot_number) -> void`.

### Process Walkthrough

**UNDERSTAND**: Key clarifying questions candidates should ask:
- "Are spots numbered globally or per category?"
- "Is this single-threaded or concurrent?"
- "If a motorcycle takes a large spot, can a bus use it after the motorcycle leaves?"

**PLAN**:
- Brute Force: Scan all spots for smallest valid available. O(n) per park.
- Optimized: Min-heap per spot size. O(log n) per park and leave.
- Alternative: Sorted set per spot size. Same complexity, supports range queries.

**CODE** (optimized approach):
```python
import heapq

class ParkingLot:
    def __init__(self, small, medium, large):
        self.heaps = {
            'small': list(range(1, small + 1)),
            'medium': list(range(small + 1, small + medium + 1)),
            'large': list(range(small + medium + 1, small + medium + large + 1)),
        }
        self.spot_to_size = {}
        for size, heap in self.heaps.items():
            heapq.heapify(heap)
            for spot in heap:
                self.spot_to_size[spot] = size

    def park(self, vehicle_type):
        valid = {'motorcycle': ['small', 'medium', 'large'],
                 'car': ['medium', 'large'], 'bus': ['large']}
        for size in valid[vehicle_type]:
            if self.heaps[size]:
                return heapq.heappop(self.heaps[size])
        return -1

    def leave(self, spot_number):
        heapq.heappush(self.heaps[self.spot_to_size[spot_number]], spot_number)
```

**TEST**: `ParkingLot(2, 2, 2)` -- spots: small=[1,2], medium=[3,4], large=[5,6]
- `park('car')` -> 3, `park('motorcycle')` -> 1, `park('bus')` -> 5
- `park('car')` -> 4, `park('car')` -> 6 (overflow to large), `park('car')` -> -1
- `leave(3)`, `park('car')` -> 3 (freed spot reused)

**Follow-Ups**: Add concurrency. Add per-size billing. Add multiple floors (prefer lower).

---

## Problem 2: Meeting Free Slots

**Level**: Mid-Level | **Patterns Tested**: Interval merging, multi-source aggregation
**Why This Problem**: Requires combining intervals from N people, merging, and finding gaps. Tests multi-step decomposition.

### Statement

Given N people's schedules (lists of [start, end] busy intervals) and duration `d`, find all time slots where ALL people are free for at least `d` minutes. Working hours: 9:00-18:00 (540-1080 in minutes).

### Process Walkthrough

**UNDERSTAND**: Key clarifying questions:
- "Are individual schedules sorted? Can they overlap within one person?"
- "Are endpoints inclusive or exclusive?"
- "Return empty list if no common free slots?"

**PLAN**:
- Brute Force: Check every minute. O(W * N * M). Wasteful.
- Optimized: Flatten all intervals, sort, merge overlapping, find gaps >= d. O(T log T).
- Alternative: Sweep line with event counting. Same complexity, different style.

**CODE** (optimized approach):
```python
def find_free_slots(schedules, duration):
    WORK_START, WORK_END = 540, 1080
    all_busy = [iv for person in schedules for iv in person]
    all_busy.sort(key=lambda x: x[0])

    merged = []
    for start, end in all_busy:
        if merged and start <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])

    free_slots, prev_end = [], WORK_START
    for start, end in merged:
        if start > prev_end and start - prev_end >= duration:
            free_slots.append([max(prev_end, WORK_START), min(start, WORK_END)])
        prev_end = max(prev_end, end)
    if WORK_END - prev_end >= duration:
        free_slots.append([prev_end, WORK_END])
    return free_slots
```

**TEST**: Person A: [[540,600],[660,720]], Person B: [[570,630]], duration=30
- Merged: [[540,630],[660,720]]
- Gaps: [630,660] (30 min), [720,1080] (360 min) -- both valid
- Output: [[630,660],[720,1080]] (10:30-11:00, 12:00-18:00)

**Follow-Ups**: Different working hours per person. Find earliest slot only. Handle real-time schedule updates.

---

## Problem 3: File Path Compression

**Level**: Senior / Staff | **Patterns Tested**: Stack-based processing, string parsing
**Why This Problem**: The word "stack" never appears. Candidates must recognize the pattern from structure. Tests parsing discipline and edge case awareness.

### Statement

Simplify an absolute Unix path by resolving `.` (current dir), `..` (parent dir), and redundant slashes.
- `/a/b/../c/./d` -> `/a/c/d`
- `/a/b/../../c/` -> `/c`
- `////a///b//` -> `/a/b`
- `/..` -> `/`

### Process Walkthrough

**UNDERSTAND**: Key clarifying questions:
- "Can '..' go above root? Is '/../x' just '/x'?"
- "Is '...' (three dots) a valid directory name or special?"
- "Should trailing slashes be removed?"

**PLAN**:
- Brute Force: Repeated string replacements. Fragile, many edge cases.
- Optimized (Stack): Split by '/', process tokens with a stack. O(n) time and space.
- Alternative: Recursive with depth parameter. Same complexity, harder to debug.

**CODE** (stack approach):
```python
def simplify_path(path):
    stack = []
    for part in path.split('/'):
        if part == '' or part == '.':
            continue
        elif part == '..':
            if stack:
                stack.pop()
        else:
            stack.append(part)
    return '/' + '/'.join(stack)
```

**TEST**:
- `/a/b/../c/./d` -> push a,b; pop b; push c,d -> `/a/c/d`
- `/../` -> pop on empty (no-op) -> `/`
- `/a/.../b` -> push a, push ... (valid name!), push b -> `/a/.../b`

**Follow-Ups**: Support relative paths. Support symlinks. Handle 10^6-character paths.

---

## Edge Cases Cheat Sheet (Interviewer Reference)

**Parking Lot**: Full for one vehicle type but not others. Leave unassigned spot. Reuse freed spots.
**Meeting Free Slots**: No busy intervals. Meetings spanning full day. Adjacent meetings (end == start). Duration > working day.
**File Path Compression**: Path is just "/". Only ".." components. "..." as valid name. Trailing slashes.
