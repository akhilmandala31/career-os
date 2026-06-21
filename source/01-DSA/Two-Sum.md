# Two Sum

## Problem

Given an array of integers `nums` and a target integer `target`, return the **indices** of the two numbers that add up to the target. Exactly one solution exists, and you may not use the same element twice.

## Pattern

HashMap / Complement Lookup

## Brute Force

**Approach:** Two nested loops. For every element `i`, check every element `j` after it to see if `nums[i] + nums[j] == target`.

```java
for (int i = 0; i < nums.length; i++) {
    for (int j = i + 1; j < nums.length; j++) {
        if (nums[i] + nums[j] == target) return new int[]{i, j};
    }
}
```

**Time Complexity:** O(n²)

**Space Complexity:** O(1)

## Optimized Solution

**Approach:** Single pass with a HashMap. For each element, compute the complement (`target - nums[i]`). If the complement already exists as a key in the map, you've found the pair — return its stored index and the current index `i`. Otherwise, store `nums[i] → i` in the map.

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> pair = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int find = target - nums[i];
        if (pair.containsKey(find)) {
            return new int[]{pair.get(find), i};
        }
        pair.put(nums[i], i);
    }
    return new int[]{}; // problem guarantees this never hits
}
```

**Why store value → index?** Because you need to return indices, not values. The map is a reverse lookup: given a number, give me where it appeared.

**Why check before insert?** Prevents matching an element with itself. E.g. `nums = [3,3], target = 6` — at `i=0` the map is empty so 3 is not found, then inserted. At `i=1` the complement 3 is found at index 0 → returns `[0,1]`. ✅

**Time Complexity:** O(n)

**Space Complexity:** O(n) — worst case all elements stored before finding the pair

## Key Insight

You don't need to find both numbers at once. At each step, ask: *"Has the number I need already been seen?"* The HashMap turns that O(n) scan into an O(1) lookup.

## Mistakes Made

- **Returned `nums[i]` instead of `i`** — the problem asks for indices, not values. The map stores `value → index` precisely to avoid this. When in doubt, re-read what the return type is before coding.
- **Starting from index 0 returns the first occurrence** — this is correct and intentional. The complement check ensures you never use the same element twice (`j + 1` in brute force; in HashMap the current element isn't added to the map until after the check).

## Revision Needed

No

## Similar Problems

- Three Sum (extend this pattern with a fixed pointer + two pointers)
- Two Sum II — Input Array Is Sorted (two pointers instead of HashMap)
- Subarray Sum Equals K (prefix sum + HashMap — same complement idea)

## Interview Notes

**Clarify before coding:**
- Can the array contain duplicates?
- Can elements be negative?
- Is there always exactly one solution, or could there be zero or multiple?
- Can I modify the input array?

**Follow-up questions to be ready for:**

1. **What if the array is sorted?** → Two pointers, O(1) space, no HashMap needed.
2. **What if you need to return all pairs, not just one?** → HashMap approach needs adjustment to handle duplicates correctly.
3. **What if there are multiple valid answers?** → Return all pairs. How does your approach change?
4. **What if the input is a stream?** → Must use HashMap — two pointers won't work without the full array.
5. **Can you do it O(n log n) without extra space?** → Sort + two pointers. Trade space for time.
6. **What does your function return if no solution exists?** → Current code returns `new int[]{}` making intent explicit. Returning `[0,0]` would be wrong — looks like valid indices.

**The tricky case — `nums = [3,3], target = 6`:**
Works correctly because we check before inserting. At `i=0`, map is empty so 3 not found, inserted. At `i=1`, complement 3 found at index 0 → returns `[0,1]`. ✅

**Pattern reuse:** The complement-check HashMap pattern appears in Three Sum, Four Sum, Subarray Sum Equals K, and Longest Consecutive Sequence. Learn it once, apply everywhere.
