# 15 — DSA Interview Problems (Java Solutions)

> **Level:** 3-4 Years Experience | **Priority:** 🟠 Very Important  
> Covers: Arrays, Strings, HashMap, LinkedList, Stack, Trees, Sorting, Recursion, Dynamic Programming, Sliding Window, Two Pointers

---

## 📌 Table of Contents

1. [Pattern Recognition Guide](#1-pattern-recognition-guide)
2. [Arrays](#2-arrays)
3. [Strings](#3-strings)
4. [HashMap / HashSet](#4-hashmap--hashset)
5. [Two Pointers](#5-two-pointers)
6. [Sliding Window](#6-sliding-window)
7. [Stack & Queue](#7-stack--queue)
8. [Linked List](#8-linked-list)
9. [Binary Tree & BST](#9-binary-tree--bst)
10. [Sorting & Searching](#10-sorting--searching)
11. [Recursion & Backtracking](#11-recursion--backtracking)
12. [Dynamic Programming](#12-dynamic-programming)
13. [Top 30 Must-Do Problems](#13-top-30-must-do-problems)

---

## 1. Pattern Recognition Guide

| Pattern | When to Use | Example |
|---------|------------|---------|
| Two Pointers | Sorted array, pair/triplet sum | Two Sum (sorted), 3Sum |
| Sliding Window | Subarray/substring of size k, max/min window | Max sum subarray, longest substring |
| HashMap | Frequency count, two sum, anagram | Two Sum, group anagrams |
| Binary Search | Sorted data, search space reduction | Search rotated array |
| Stack | Matching brackets, next greater element | Valid parentheses |
| BFS/DFS | Tree/Graph traversal | Level order, number of islands |
| Dynamic Programming | Overlapping subproblems, optimal substructure | Fibonacci, knapsack, LCS |
| Backtracking | Generate all combinations/permutations | Subsets, permutations |
| Greedy | Local optimal → global optimal | Activity selection, jump game |

---

## 2. Arrays

### 2.1 Two Sum
```java
// Given array, find two numbers that add up to target
// Return indices
// Time: O(n), Space: O(n)
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[]{map.get(complement), i};
        }
        map.put(nums[i], i);
    }
    return new int[]{};
}
```

### 2.2 Maximum Subarray (Kadane's Algorithm)
```java
// Find contiguous subarray with largest sum
// Time: O(n), Space: O(1)
public int maxSubArray(int[] nums) {
    int maxSum = nums[0];
    int currentSum = nums[0];

    for (int i = 1; i < nums.length; i++) {
        currentSum = Math.max(nums[i], currentSum + nums[i]);
        maxSum = Math.max(maxSum, currentSum);
    }
    return maxSum;
}
// Input: [-2,1,-3,4,-1,2,1,-5,4] → Output: 6 ([4,-1,2,1])
```

### 2.3 Merge Two Sorted Arrays
```java
// Merge into nums1 (has enough space)
// Time: O(m+n), Space: O(1) — fill from the end
public void merge(int[] nums1, int m, int[] nums2, int n) {
    int i = m - 1, j = n - 1, k = m + n - 1;

    while (i >= 0 && j >= 0) {
        if (nums1[i] > nums2[j]) {
            nums1[k--] = nums1[i--];
        } else {
            nums1[k--] = nums2[j--];
        }
    }
    while (j >= 0) {
        nums1[k--] = nums2[j--];
    }
}
```

### 2.4 Product of Array Except Self
```java
// Without using division, Time: O(n), Space: O(1) extra
public int[] productExceptSelf(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];

    // Left pass: result[i] = product of all elements to the left
    result[0] = 1;
    for (int i = 1; i < n; i++) {
        result[i] = result[i - 1] * nums[i - 1];
    }

    // Right pass: multiply by product of all elements to the right
    int rightProduct = 1;
    for (int i = n - 1; i >= 0; i--) {
        result[i] *= rightProduct;
        rightProduct *= nums[i];
    }
    return result;
}
// [1,2,3,4] → [24,12,8,6]
```

---

## 3. Strings

### 3.1 Valid Anagram
```java
// Time: O(n), Space: O(1) — fixed array of 26
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;

    int[] count = new int[26];
    for (int i = 0; i < s.length(); i++) {
        count[s.charAt(i) - 'a']++;
        count[t.charAt(i) - 'a']--;
    }
    for (int c : count) {
        if (c != 0) return false;
    }
    return true;
}
```

### 3.2 Longest Substring Without Repeating Characters
```java
// Sliding window approach
// Time: O(n), Space: O(min(m,n)) where m = charset size
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> map = new HashMap<>();
    int maxLen = 0;
    int left = 0;

    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (map.containsKey(c)) {
            left = Math.max(left, map.get(c) + 1);
        }
        map.put(c, right);
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
// "abcabcbb" → 3 ("abc")
```

### 3.3 Reverse Words in a String
```java
public String reverseWords(String s) {
    String[] words = s.trim().split("\\s+");
    StringBuilder sb = new StringBuilder();
    for (int i = words.length - 1; i >= 0; i--) {
        sb.append(words[i]);
        if (i > 0) sb.append(" ");
    }
    return sb.toString();
}
// "  the sky is blue  " → "blue is sky the"
```

### 3.4 Group Anagrams
```java
// Time: O(n * k log k) where k = max string length
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    for (String s : strs) {
        char[] arr = s.toCharArray();
        Arrays.sort(arr);
        String key = new String(arr);
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(map.values());
}
// ["eat","tea","tan","ate","nat","bat"]
// → [["eat","tea","ate"],["tan","nat"],["bat"]]
```

---

## 4. HashMap / HashSet

### 4.1 First Non-Repeating Character
```java
public int firstUniqChar(String s) {
    Map<Character, Integer> count = new LinkedHashMap<>();
    for (char c : s.toCharArray()) {
        count.merge(c, 1, Integer::sum);
    }
    for (int i = 0; i < s.length(); i++) {
        if (count.get(s.charAt(i)) == 1) return i;
    }
    return -1;
}
```

### 4.2 Contains Duplicate
```java
public boolean containsDuplicate(int[] nums) {
    Set<Integer> seen = new HashSet<>();
    for (int num : nums) {
        if (!seen.add(num)) return true;  // add returns false if already exists
    }
    return false;
}
```

### 4.3 Top K Frequent Elements
```java
// Time: O(n log k), Space: O(n)
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> count = new HashMap<>();
    for (int n : nums) count.merge(n, 1, Integer::sum);

    PriorityQueue<Integer> heap = new PriorityQueue<>(
        Comparator.comparingInt(count::get)  // min-heap by frequency
    );

    for (int key : count.keySet()) {
        heap.offer(key);
        if (heap.size() > k) heap.poll();   // remove least frequent
    }

    return heap.stream().mapToInt(i -> i).toArray();
}
// [1,1,1,2,2,3], k=2 → [1,2]
```

---

## 5. Two Pointers

### 5.1 3Sum (Find All Unique Triplets Summing to Zero)
```java
// Time: O(n²), Space: O(1) extra
public List<List<Integer>> threeSum(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    Arrays.sort(nums);

    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue;  // skip duplicates

        int left = i + 1, right = nums.length - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                while (left < right && nums[left] == nums[left + 1]) left++;   // skip dup
                while (left < right && nums[right] == nums[right - 1]) right--; // skip dup
                left++;
                right--;
            } else if (sum < 0) {
                left++;
            } else {
                right--;
            }
        }
    }
    return result;
}
```

### 5.2 Container With Most Water
```java
// Time: O(n), Space: O(1)
public int maxArea(int[] height) {
    int left = 0, right = height.length - 1;
    int maxWater = 0;

    while (left < right) {
        int water = Math.min(height[left], height[right]) * (right - left);
        maxWater = Math.max(maxWater, water);
        if (height[left] < height[right]) {
            left++;
        } else {
            right--;
        }
    }
    return maxWater;
}
```

### 5.3 Valid Palindrome
```java
public boolean isPalindrome(String s) {
    int left = 0, right = s.length() - 1;
    while (left < right) {
        while (left < right && !Character.isLetterOrDigit(s.charAt(left))) left++;
        while (left < right && !Character.isLetterOrDigit(s.charAt(right))) right--;
        if (Character.toLowerCase(s.charAt(left)) != Character.toLowerCase(s.charAt(right))) {
            return false;
        }
        left++;
        right--;
    }
    return true;
}
```

---

## 6. Sliding Window

### 6.1 Maximum Sum Subarray of Size K
```java
// Time: O(n), Space: O(1)
public int maxSumSubarray(int[] arr, int k) {
    int windowSum = 0;
    for (int i = 0; i < k; i++) {
        windowSum += arr[i];
    }

    int maxSum = windowSum;
    for (int i = k; i < arr.length; i++) {
        windowSum += arr[i] - arr[i - k];  // slide: add right, remove left
        maxSum = Math.max(maxSum, windowSum);
    }
    return maxSum;
}
```

### 6.2 Minimum Window Substring
```java
// Find smallest substring of s containing all characters of t
// Time: O(m + n), Space: O(m + n)
public String minWindow(String s, String t) {
    if (s.isEmpty() || t.isEmpty()) return "";

    Map<Character, Integer> need = new HashMap<>();
    for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);

    int required = need.size();
    int formed = 0;
    Map<Character, Integer> window = new HashMap<>();

    int left = 0, minLen = Integer.MAX_VALUE, minLeft = 0;

    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        window.merge(c, 1, Integer::sum);

        if (need.containsKey(c) && window.get(c).intValue() == need.get(c).intValue()) {
            formed++;
        }

        while (formed == required) {
            if (right - left + 1 < minLen) {
                minLen = right - left + 1;
                minLeft = left;
            }
            char leftChar = s.charAt(left);
            window.merge(leftChar, -1, Integer::sum);
            if (need.containsKey(leftChar) && window.get(leftChar) < need.get(leftChar)) {
                formed--;
            }
            left++;
        }
    }
    return minLen == Integer.MAX_VALUE ? "" : s.substring(minLeft, minLeft + minLen);
}
```

---

## 7. Stack & Queue

### 7.1 Valid Parentheses
```java
// Time: O(n), Space: O(n)
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> pairs = Map.of(')', '(', '}', '{', ']', '[');

    for (char c : s.toCharArray()) {
        if (pairs.containsValue(c)) {
            stack.push(c);
        } else if (pairs.containsKey(c)) {
            if (stack.isEmpty() || stack.pop() != pairs.get(c)) {
                return false;
            }
        }
    }
    return stack.isEmpty();
}
```

### 7.2 Next Greater Element
```java
// For each element, find the next greater element to its right
// Time: O(n), Space: O(n)
public int[] nextGreaterElement(int[] arr) {
    int n = arr.length;
    int[] result = new int[n];
    Deque<Integer> stack = new ArrayDeque<>();  // stores indices

    for (int i = n - 1; i >= 0; i--) {
        while (!stack.isEmpty() && stack.peek() <= arr[i]) {
            stack.pop();
        }
        result[i] = stack.isEmpty() ? -1 : stack.peek();
        stack.push(arr[i]);
    }
    return result;
}
// [4, 5, 2, 25] → [5, 25, 25, -1]
```

### 7.3 Min Stack (O(1) getMin)
```java
class MinStack {
    private Deque<int[]> stack = new ArrayDeque<>();  // [value, currentMin]

    public void push(int val) {
        int min = stack.isEmpty() ? val : Math.min(val, stack.peek()[1]);
        stack.push(new int[]{val, min});
    }

    public void pop() { stack.pop(); }
    public int top() { return stack.peek()[0]; }
    public int getMin() { return stack.peek()[1]; }
}
```

---

## 8. Linked List

### 8.1 Reverse Linked List
```java
// Iterative — Time: O(n), Space: O(1)
public ListNode reverseList(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}

// Recursive
public ListNode reverseListRecursive(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode newHead = reverseListRecursive(head.next);
    head.next.next = head;
    head.next = null;
    return newHead;
}
```

### 8.2 Detect Cycle (Floyd's Algorithm)
```java
// Time: O(n), Space: O(1)
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}

// Find the start of cycle
public ListNode detectCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {
            slow = head;
            while (slow != fast) {
                slow = slow.next;
                fast = fast.next;
            }
            return slow;  // start of cycle
        }
    }
    return null;
}
```

### 8.3 Merge Two Sorted Lists
```java
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode curr = dummy;

    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) {
            curr.next = l1;
            l1 = l1.next;
        } else {
            curr.next = l2;
            l2 = l2.next;
        }
        curr = curr.next;
    }
    curr.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
```

### 8.4 Find Middle of Linked List
```java
public ListNode middleNode(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;  // slow is at the middle
}
```

---

## 9. Binary Tree & BST

### 9.1 Binary Tree Traversals
```java
// Inorder (Left → Root → Right)  — gives sorted order for BST
public void inorder(TreeNode root, List<Integer> result) {
    if (root == null) return;
    inorder(root.left, result);
    result.add(root.val);
    inorder(root.right, result);
}

// Preorder (Root → Left → Right)
public void preorder(TreeNode root, List<Integer> result) {
    if (root == null) return;
    result.add(root.val);
    preorder(root.left, result);
    preorder(root.right, result);
}

// Level Order (BFS)
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;

    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);

    while (!queue.isEmpty()) {
        int size = queue.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(level);
    }
    return result;
}
```

### 9.2 Maximum Depth of Binary Tree
```java
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

### 9.3 Validate BST
```java
public boolean isValidBST(TreeNode root) {
    return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

private boolean validate(TreeNode node, long min, long max) {
    if (node == null) return true;
    if (node.val <= min || node.val >= max) return false;
    return validate(node.left, min, node.val) &&
           validate(node.right, node.val, max);
}
```

### 9.4 Lowest Common Ancestor (BST)
```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    if (root.val > p.val && root.val > q.val) {
        return lowestCommonAncestor(root.left, p, q);
    } else if (root.val < p.val && root.val < q.val) {
        return lowestCommonAncestor(root.right, p, q);
    }
    return root;  // split point
}
```

### 9.5 Diameter of Binary Tree
```java
int diameter = 0;

public int diameterOfBinaryTree(TreeNode root) {
    height(root);
    return diameter;
}

private int height(TreeNode node) {
    if (node == null) return 0;
    int left = height(node.left);
    int right = height(node.right);
    diameter = Math.max(diameter, left + right);
    return 1 + Math.max(left, right);
}
```

---

## 10. Sorting & Searching

### 10.1 Binary Search
```java
// Time: O(log n)
public int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;  // avoid overflow
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

### 10.2 Search in Rotated Sorted Array
```java
// Time: O(log n)
public int search(int[] nums, int target) {
    int left = 0, right = nums.length - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;

        // Left half is sorted
        if (nums[left] <= nums[mid]) {
            if (target >= nums[left] && target < nums[mid]) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        // Right half is sorted
        else {
            if (target > nums[mid] && target <= nums[right]) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
    }
    return -1;
}
```

### 10.3 Merge Sort
```java
// Time: O(n log n), Space: O(n) — stable sort
public void mergeSort(int[] arr, int left, int right) {
    if (left >= right) return;

    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}

private void merge(int[] arr, int left, int mid, int right) {
    int[] temp = new int[right - left + 1];
    int i = left, j = mid + 1, k = 0;

    while (i <= mid && j <= right) {
        if (arr[i] <= arr[j]) temp[k++] = arr[i++];
        else temp[k++] = arr[j++];
    }
    while (i <= mid) temp[k++] = arr[i++];
    while (j <= right) temp[k++] = arr[j++];

    System.arraycopy(temp, 0, arr, left, temp.length);
}
```

---

## 11. Recursion & Backtracking

### 11.1 Generate All Subsets
```java
// Time: O(2^n), Space: O(n)
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] nums, int start, List<Integer> current,
                       List<List<Integer>> result) {
    result.add(new ArrayList<>(current));  // ← add copy, not reference!
    for (int i = start; i < nums.length; i++) {
        current.add(nums[i]);
        backtrack(nums, i + 1, current, result);
        current.remove(current.size() - 1);  // undo choice
    }
}
// [1,2,3] → [[], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]]
```

### 11.2 Permutations
```java
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    boolean[] used = new boolean[nums.length];
    permuteHelper(nums, used, new ArrayList<>(), result);
    return result;
}

private void permuteHelper(int[] nums, boolean[] used, List<Integer> current,
                           List<List<Integer>> result) {
    if (current.size() == nums.length) {
        result.add(new ArrayList<>(current));
        return;
    }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        used[i] = true;
        current.add(nums[i]);
        permuteHelper(nums, used, current, result);
        current.remove(current.size() - 1);
        used[i] = false;
    }
}
```

### 11.3 Letter Combinations of Phone Number
```java
private static final String[] PHONE = {
    "", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"
};

public List<String> letterCombinations(String digits) {
    List<String> result = new ArrayList<>();
    if (digits.isEmpty()) return result;
    combine(digits, 0, new StringBuilder(), result);
    return result;
}

private void combine(String digits, int idx, StringBuilder sb, List<String> result) {
    if (idx == digits.length()) {
        result.add(sb.toString());
        return;
    }
    String letters = PHONE[digits.charAt(idx) - '0'];
    for (char c : letters.toCharArray()) {
        sb.append(c);
        combine(digits, idx + 1, sb, result);
        sb.deleteCharAt(sb.length() - 1);
    }
}
// "23" → ["ad","ae","af","bd","be","bf","cd","ce","cf"]
```

---

## 12. Dynamic Programming

### 12.1 Climbing Stairs
```java
// How many ways to climb n stairs (1 or 2 steps at a time)
// Time: O(n), Space: O(1)
public int climbStairs(int n) {
    if (n <= 2) return n;
    int prev2 = 1, prev1 = 2;
    for (int i = 3; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

### 12.2 Longest Common Subsequence
```java
// Time: O(m*n), Space: O(m*n)
public int longestCommonSubsequence(String text1, String text2) {
    int m = text1.length(), n = text2.length();
    int[][] dp = new int[m + 1][n + 1];

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                dp[i][j] = 1 + dp[i - 1][j - 1];
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    return dp[m][n];
}
// "abcde", "ace" → 3 ("ace")
```

### 12.3 0/1 Knapsack
```java
// Time: O(n*W), Space: O(n*W)
public int knapsack(int[] weights, int[] values, int W) {
    int n = weights.length;
    int[][] dp = new int[n + 1][W + 1];

    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            dp[i][w] = dp[i - 1][w];  // don't take item i
            if (weights[i - 1] <= w) {
                dp[i][w] = Math.max(dp[i][w],
                    dp[i - 1][w - weights[i - 1]] + values[i - 1]);  // take item i
            }
        }
    }
    return dp[n][W];
}
```

### 12.4 Coin Change
```java
// Minimum coins to make amount
// Time: O(amount * coins), Space: O(amount)
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);
    dp[0] = 0;

    for (int i = 1; i <= amount; i++) {
        for (int coin : coins) {
            if (coin <= i) {
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
            }
        }
    }
    return dp[amount] > amount ? -1 : dp[amount];
}
// coins=[1,5,11], amount=15 → 3 (5+5+5)
```

### 12.5 Longest Increasing Subsequence
```java
// Time: O(n log n) — using binary search with patience sort
public int lengthOfLIS(int[] nums) {
    List<Integer> tails = new ArrayList<>();

    for (int num : nums) {
        int pos = Collections.binarySearch(tails, num);
        if (pos < 0) pos = -(pos + 1);

        if (pos == tails.size()) {
            tails.add(num);
        } else {
            tails.set(pos, num);
        }
    }
    return tails.size();
}
// [10,9,2,5,3,7,101,18] → 4 ([2,3,7,101])
```

### 12.6 House Robber
```java
// Can't rob adjacent houses. Maximize theft.
// Time: O(n), Space: O(1)
public int rob(int[] nums) {
    int prev2 = 0, prev1 = 0;
    for (int num : nums) {
        int curr = Math.max(prev1, prev2 + num);
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
// [2,7,9,3,1] → 12 (2+9+1)
```

---

## 13. Top 30 Must-Do Problems

| # | Problem | Pattern | Difficulty |
|---|---------|---------|-----------|
| 1 | Two Sum | HashMap | Easy |
| 2 | Valid Parentheses | Stack | Easy |
| 3 | Merge Two Sorted Lists | Linked List | Easy |
| 4 | Best Time to Buy & Sell Stock | Greedy | Easy |
| 5 | Valid Palindrome | Two Pointers | Easy |
| 6 | Reverse Linked List | Linked List | Easy |
| 7 | Contains Duplicate | HashSet | Easy |
| 8 | Maximum Subarray (Kadane's) | DP / Greedy | Medium |
| 9 | Climbing Stairs | DP | Easy |
| 10 | Binary Search | Binary Search | Easy |
| 11 | 3Sum | Two Pointers | Medium |
| 12 | Container With Most Water | Two Pointers | Medium |
| 13 | Product of Array Except Self | Arrays | Medium |
| 14 | Longest Substring Without Repeating | Sliding Window | Medium |
| 15 | Group Anagrams | HashMap | Medium |
| 16 | Coin Change | DP | Medium |
| 17 | Top K Frequent Elements | Heap/HashMap | Medium |
| 18 | Search in Rotated Sorted Array | Binary Search | Medium |
| 19 | Letter Combinations of Phone | Backtracking | Medium |
| 20 | Validate BST | Tree / DFS | Medium |
| 21 | Level Order Traversal | Tree / BFS | Medium |
| 22 | LRU Cache | HashMap + DLL | Medium |
| 23 | Number of Islands | DFS/BFS | Medium |
| 24 | Longest Common Subsequence | DP | Medium |
| 25 | Merge Intervals | Sorting | Medium |
| 26 | Subsets | Backtracking | Medium |
| 27 | House Robber | DP | Medium |
| 28 | Word Search | Backtracking | Medium |
| 29 | Longest Increasing Subsequence | DP / Binary Search | Medium |
| 30 | Minimum Window Substring | Sliding Window | Hard |

### ⭐ Bonus Problems for 4 YOE

| Problem | Why It's Asked |
|---------|---------------|
| LRU Cache | Tests OOP + Data Structures |
| Design HashMap | Tests hashing knowledge |
| Implement Trie | Tests tree + string knowledge |
| Serialize/Deserialize Binary Tree | Tests recursion + design |
| Merge K Sorted Lists | Tests heap + linked list |

---

### 💡 Interview Tips for DSA Rounds

1. **Think out loud** — explain your approach before coding
2. **Start with brute force** — then optimize
3. **Clarify edge cases** — empty input, single element, duplicates, negative numbers
4. **State time & space complexity** before and after optimizing
5. **Test with examples** — walk through your code with sample input
6. **Don't memorize** — understand patterns (two pointers, sliding window, BFS/DFS)
7. **Practice on paper/whiteboard** — many interviews don't have IDE auto-complete

---

> ← [14_Unix_Linux_Commands.md](14_Unix_Linux_Commands.md) | 🏠 [INDEX](00_INDEX.md)
