---
layout: default
title: Coding Design Patterns
---

# Coding Design Patterns

The recurring patterns behind most array / linked-list interview problems.

- [1. Two Pointers](#1-two-pointers)
- [2. Prefix Sum](#2-prefix-sum)
- [3. Sliding Window](#3-sliding-window)
- [4. Slow / Fast Pointers](#4-slow--fast-pointers)
- [5. Linked List Reversal](#5-linked-list-reversal)
- [6. Monotonic Stack](#6-monotonic-stack)

---

## 1. Two Pointers

Two usages:

1. A **start** pointer at the beginning of an array and an **end** pointer at the end,
   converging to find a target sum across pairs (dichotomy style).
2. A **slow** pointer and a **fast** pointer incrementing e.g. twice as fast, used to
   locate a cycle in a linked list.

### Two Sum II

```python
def twoSum(self, numbers, target):
    """
    :type numbers: List[int]
    :type target: int
    :rtype: List[int]
    """
    size = len(numbers) - 1

    left = 0
    right = size

    while left < right:
        sum = numbers[right] + numbers[left]
        if sum == target:
            return list((left + 1, right + 1))
        # Smaller than the target, move forward
        elif sum < target:
            left += 1
        # Over the target, step back
        else:
            right -= 1
    return None
```

### Remove duplicates

```python
def removeDuplicates(self, nums):
    """
    :type nums: List[int]
    :rtype: int
    """
    size = len(nums) - 1
    next = 0
    for i in range(size):
        next = i + 1
        while nums[next] == nums[i] and nums[i] is not None:
            nums.pop(next)
            nums.append(None)
    while None in nums:
        nums.pop()
```

### Palindrome

```python
def isPalindrome(self, x):
    """
    :type x: int
    :rtype: bool
    """
    n = str(x)
    size = len(n)
    left = 0
    right = size - 1

    while left != right and left < size:
        if n[left] != n[right]:
            return False
        left += 1
        right -= 1
    return True
```

### Move zeroes

```python
def moveZeroes(self, nums):
    """
    :type nums: List[int]
    :rtype: None Do not return anything, modify nums in-place instead.
    """
    # Right pointer iterates over the array
    # Left pointer increments when there is no zero to swap
    size = len(nums)
    left = 0
    for right in range(size):
        if nums[right] != 0:
            (nums[right], nums[left]) = (nums[left], nums[right])
            left += 1
```

The idea is to swap every non-zero value and park it on the left of the array.

- The **left** pointer increments after a swap, ready for the next one.
- The **right** pointer is the iterator: as soon as it finds a non-zero value,
  it swaps it with the element pointed by the left pointer.

---

## 2. Prefix Sum

**Idea:** build a cumulative sum array while walking the input array, so that any
range query becomes a single subtraction.

```python
class NumArray(object):
    def __init__(self, nums):
        """
        :type nums: List[int]
        """
        self.prefixArray = []
        self.nums = list(nums)
        self.prefix()

    def prefix(self):
        acc = 0
        for i in range(len(self.nums)):
            acc += self.nums[i]
            self.prefixArray.append(acc)

    def sumRange(self, left, right):
        """
        :type left: int
        :type right: int
        :rtype: int
        """
        if left > 0:
            return self.prefixArray[right] - self.prefixArray[left - 1]
        else:
            return self.prefixArray[right]
```

If `left` is zero, reading `prefixArray[right]` directly gives the cumulated value.
Otherwise subtract the element just before the one pointed by `left`.

### Subarray sum equals K

```python
class Solution:
    def subarraySum(self, nums, k):
        sum = 0
        count = 0
        map = defaultdict(int)
        map[0] = 1

        for num in nums:
            sum += num
            rem = sum - k

            if rem in map:
                count += map[rem]
            map[sum] += 1

        return count
```

1. **Initialization**
   - `sum` tracks the cumulative sum of the elements seen so far.
   - `count` stores the number of subarrays whose sum equals `k`.
   - The hash map stores the frequency of each cumulative sum, initialized with
     `{0: 1}` — a sum of 0 has occurred once.
2. **Iterating through the array** — walk `nums` left to right, updating `sum`.
3. **Calculating the remainder** — `rem = sum - k` is what is needed to reach `k`.
4. **Checking for subarrays** — if `rem` is in the map, a subarray ending at the
   current index sums to `k`; increment `count` by the frequency of `rem`.
5. **Updating the frequency map** — bump the count of the current `sum`.
6. **Return the count.**

### Minimum value to get a positive step-by-step sum

```python
class Solution(object):
    def minStartValue(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        acc = 0
        start = 1

        for num in nums:
            acc += num
            start = max(start, 1 - acc)
        return start
```

---

## 3. Sliding Window

**Example:** you want to borrow 5 consecutive books out of N, and you want the most
expensive group of 5. Sum the first 5, shift the range by one, and keep the best.

No need to recompute everything on each shift: take the previous sum, add the book
that entered and remove the one that left.

```python
def toto(prices, k):
    if len(prices) < k:
        return 0
    tot = sum(prices[:k])
    maxtot = tot
    for i in range(len(prices) - k):
        # remove the element leaving the window
        tot -= prices[i]
        # add the new element
        tot += prices[i + k]
        maxtot = max(maxtot, tot)
    return maxtot
```

### Longest substring without repeating characters

```python
class Solution(object):
    def lengthOfLongestSubstring(self, s):
        """
        :type s: str
        :rtype: int
        """
        visited = set()
        left = 0
        maxtot = 0
        for right in range(len(s)):
            while s[right] in visited:
                visited.remove(s[left])
                left += 1
            visited.add(s[right])
            maxtot = max(maxtot, right - left + 1)
        return maxtot
```

A mix of Two Pointers and Sliding Window: when an already-seen character shows up,
shift the left pointer while dropping the character it was pointing at.

### Find max average

```python
class Solution:
    def findMaxAverage(self, nums: List[int], k: int) -> float:
        if len(nums) < k:
            return 0
        tot = sum(nums[:k])
        max_average = tot / k
        for i in range(len(nums) - k):
            # remove the element leaving the window
            tot -= nums[i]
            # add the new element
            tot += nums[i + k]
            max_average = max(max_average, tot / k)
        return max_average
```

### Minimum window substring

```python
class Solution:
    def minWindow(self, s: str, t: str) -> str:
        tmp_list = {}
        for left in range(len(s)):
            chars = list(t)
            tmp = ''
            for right in range(left, len(s)):
                if len(chars) == 0:
                    break
                if s[right] in chars:
                    chars.remove(s[right])
                tmp += s[right]
            if len(chars) == 0:
                tmp_list[len(tmp)] = tmp

        return tmp_list[min(tmp_list.keys())] if tmp_list else ""
```

---

## 4. Slow / Fast Pointers

### Linked list cycle

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def hasCycle(self, head: Optional[ListNode]) -> bool:
        if not head:
            return False
        slow = head
        fast = head
        while fast.next and fast.next.next:
            slow = slow.next
            fast = fast.next.next
            if slow == fast:
                return True
        return False
```

### Find the duplicate number

```python
class Solution:
    def findDuplicate(self, nums):
        slow = nums[nums[0]]
        fast = nums[nums[nums[0]]]

        while slow != fast:
            slow = nums[slow]
            fast = nums[nums[fast]]

        slow = nums[0]
        while slow != fast:
            slow = nums[slow]
            fast = nums[fast]
        return slow
```

### Happy number

```python
def nex(n):
    number = list(str(n))
    ope = sum(int(digit) ** 2 for digit in number)
    return ope

class Solution:
    def isHappy(self, n):
        slow = nex(n)
        fast = nex(nex(n))

        while slow != fast and fast != 1:
            slow = nex(slow)
            fast = nex(nex(fast))
        return fast == 1
```

---

## 5. Linked List Reversal

```python
def reverseList(self, head: Optional[ListNode]) -> Optional[ListNode]:
    current = head
    prev = None

    # Iterate over the length of the linked list
    while current is not None:
        # The node after the current one
        nextNode = current.next
        # The current node now points to the previous one
        current.next = prev
        # Next iteration's previous will be the current node
        prev = current
        # Next iteration's current will be the following node
        current = nextNode

    return prev
```

### Same thing, but on a sub-list

```python
def reverseBetween(self, head: Optional[ListNode], left: int, right: int) -> Optional[ListNode]:
    if left == right:
        return head
    # Create a zero node pointing at head, and use it as prev
    out = ListNode(0, head)
    prev = out

    # Advance prev until reaching left
    for _ in range(left - 1):
        prev = prev.next

    current = prev.next
    for _ in range(right - left):
        tmp = current.next
        current.next = tmp.next
        tmp.next = prev.next
        prev.next = tmp

    return out.next
```

---

## 6. Monotonic Stack

> TODO — source note is empty.

[Back to Algorithms & Design Patterns](./)
