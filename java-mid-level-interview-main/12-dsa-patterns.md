# 🧠 DSA Patterns - FAANG Interview Preparation

## Índice
1. [Sliding Window](#1-sliding-window)
2. [Two Pointers](#2-two-pointers)
3. [Fast & Slow Pointers](#3-fast--slow-pointers)
4. [Merge Intervals](#4-merge-intervals)
5. [Cyclic Sort](#5-cyclic-sort)
6. [In-place Reversal of LinkedList](#6-in-place-reversal-of-linkedlist)
7. [Binary Search](#7-binary-search)
8. [Tree BFS](#8-tree-bfs)
9. [Tree DFS](#9-tree-dfs)
10. [Graphs (BFS/DFS)](#10-graphs-bfsdfs)
11. [Backtracking](#11-backtracking)
12. [Dynamic Programming](#12-dynamic-programming)
13. [Greedy Algorithms](#13-greedy-algorithms)
14. [Topological Sort](#14-topological-sort)
15. [Monotonic Stack](#15-monotonic-stack)
16. [Trie](#16-trie)
17. [Union Find (Disjoint Set)](#17-union-find-disjoint-set)
18. [Bit Manipulation](#18-bit-manipulation)

---

## 1. Sliding Window

### 📌 Concepto

**Cuándo usar**: Problemas que involucran subarrays/substrings contiguos con condición específica.

**Señales de identificación**:
- "Maximum/minimum subarray/substring de tamaño K"
- "Longest/shortest subarray con condición X"
- "Subarray/substring con sum/product específico"

**Pattern core**:
```java
int left = 0;
for (int right = 0; right < n; right++) {
    // Expandir ventana: agregar arr[right]
    
    while (window_condition_violated) {
        // Contraer ventana: remover arr[left]
        left++;
    }
    
    // Actualizar resultado con ventana válida [left, right]
}
```

### 💡 Ejemplos Prácticos

#### **Ejemplo 1: Maximum Sum Subarray of Size K**

```java
/**
 * Problema: Dado un array, encontrar el subarray de tamaño K con suma máxima.
 * 
 * Input: [2, 1, 5, 1, 3, 2], K=3
 * Output: 9 (subarray [5, 1, 3])
 */
public class MaxSumSubarray {
    
    // ❌ Brute Force: O(n*k)
    public static int maxSumBruteForce(int[] arr, int k) {
        int maxSum = Integer.MIN_VALUE;
        
        for (int i = 0; i <= arr.length - k; i++) {
            int sum = 0;
            for (int j = i; j < i + k; j++) {
                sum += arr[j];
            }
            maxSum = Math.max(maxSum, sum);
        }
        
        return maxSum;
    }
    
    // ✅ Sliding Window: O(n)
    public static int maxSumSlidingWindow(int[] arr, int k) {
        if (arr.length < k) return -1;
        
        // Calcular sum de primera ventana
        int windowSum = 0;
        for (int i = 0; i < k; i++) {
            windowSum += arr[i];
        }
        
        int maxSum = windowSum;
        
        // Deslizar ventana
        for (int i = k; i < arr.length; i++) {
            windowSum += arr[i] - arr[i - k]; // Add new, remove old
            maxSum = Math.max(maxSum, windowSum);
        }
        
        return maxSum;
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(1)

#### **Ejemplo 2: Longest Substring Without Repeating Characters**

```java
/**
 * LeetCode 3: Longest Substring Without Repeating Characters
 * 
 * Input: "abcabcbb"
 * Output: 3 ("abc")
 */
public class LongestSubstringNoRepeat {
    
    public int lengthOfLongestSubstring(String s) {
        Map<Character, Integer> charIndex = new HashMap<>();
        int maxLength = 0;
        int left = 0;
        
        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);
            
            // Si el char ya está en ventana, mover left
            if (charIndex.containsKey(c)) {
                // Importante: left solo avanza, nunca retrocede
                left = Math.max(left, charIndex.get(c) + 1);
            }
            
            charIndex.put(c, right);
            maxLength = Math.max(maxLength, right - left + 1);
        }
        
        return maxLength;
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(min(n, m)) donde m = tamaño del charset

#### **Ejemplo 3: Minimum Window Substring**

```java
/**
 * LeetCode 76: Minimum Window Substring (HARD)
 * 
 * Input: s = "ADOBECODEBANC", t = "ABC"
 * Output: "BANC"
 */
public class MinimumWindowSubstring {
    
    public String minWindow(String s, String t) {
        if (s.length() < t.length()) return "";
        
        // Frecuencias de chars necesarios
        Map<Character, Integer> required = new HashMap<>();
        for (char c : t.toCharArray()) {
            required.put(c, required.getOrDefault(c, 0) + 1);
        }
        
        Map<Character, Integer> window = new HashMap<>();
        int left = 0, right = 0;
        int formed = 0; // Cuántos chars únicos tienen frecuencia correcta
        int requiredChars = required.size();
        
        // Resultado: [length, left, right]
        int[] result = {Integer.MAX_VALUE, 0, 0};
        
        while (right < s.length()) {
            // Expandir ventana
            char c = s.charAt(right);
            window.put(c, window.getOrDefault(c, 0) + 1);
            
            // Check si este char cumple su frecuencia requerida
            if (required.containsKey(c) && 
                window.get(c).intValue() == required.get(c).intValue()) {
                formed++;
            }
            
            // Contraer ventana mientras sea válida
            while (left <= right && formed == requiredChars) {
                // Actualizar resultado si es más pequeño
                if (right - left + 1 < result[0]) {
                    result[0] = right - left + 1;
                    result[1] = left;
                    result[2] = right;
                }
                
                // Remover char izquierdo
                char leftChar = s.charAt(left);
                window.put(leftChar, window.get(leftChar) - 1);
                
                if (required.containsKey(leftChar) &&
                    window.get(leftChar) < required.get(leftChar)) {
                    formed--;
                }
                
                left++;
            }
            
            right++;
        }
        
        return result[0] == Integer.MAX_VALUE ? "" 
            : s.substring(result[1], result[2] + 1);
    }
}
```

**Complejidad**:
- Time: O(|S| + |T|)
- Space: O(|S| + |T|)

### 📚 Cheat Sheet - LeetCode Problems

| Problema | Dificultad | Técnica |
|----------|-----------|---------|
| [3. Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Medium | HashMap + Window |
| [76. Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | Hard | Frequency map + Shrinking |
| [209. Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/) | Medium | Sum + Shrinking |
| [438. Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Medium | Fixed window + Freq |
| [567. Permutation in String](https://leetcode.com/problems/permutation-in-string/) | Medium | Fixed window |
| [239. Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) | Hard | Deque + Window |
| [1004. Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/) | Medium | K flips allowed |

---

## 2. Two Pointers

### 📌 Concepto

**Cuándo usar**: Arrays/lists ordenados, pares con suma específica, remover duplicados.

**Señales de identificación**:
- Array/string **ordenado**
- "Find pair with sum/difference X"
- "Remove duplicates in-place"
- "Reverse array/string"

**Patterns**:
1. **Pointers en extremos** (converging): left=0, right=n-1
2. **Pointers en mismo inicio** (slow/fast): ambos en 0
3. **Pointers en diferentes arrays**: merge sorted arrays

### 💡 Ejemplos Prácticos

#### **Ejemplo 1: Two Sum II (Sorted Array)**

```java
/**
 * LeetCode 167: Two Sum II - Input Array Is Sorted
 * 
 * Input: numbers = [2,7,11,15], target = 9
 * Output: [1,2] (1-indexed)
 */
public class TwoSumSorted {
    
    public int[] twoSum(int[] numbers, int target) {
        int left = 0;
        int right = numbers.length - 1;
        
        while (left < right) {
            int sum = numbers[left] + numbers[right];
            
            if (sum == target) {
                return new int[]{left + 1, right + 1}; // 1-indexed
            } else if (sum < target) {
                left++; // Need larger sum
            } else {
                right--; // Need smaller sum
            }
        }
        
        return new int[]{-1, -1}; // No solution
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(1)

#### **Ejemplo 2: 3Sum**

```java
/**
 * LeetCode 15: 3Sum (HARD variant)
 * 
 * Input: [-1,0,1,2,-1,-4]
 * Output: [[-1,-1,2],[-1,0,1]]
 */
public class ThreeSum {
    
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        Arrays.sort(nums); // O(n log n)
        
        for (int i = 0; i < nums.length - 2; i++) {
            // Skip duplicates para primer número
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            
            int target = -nums[i];
            int left = i + 1;
            int right = nums.length - 1;
            
            while (left < right) {
                int sum = nums[left] + nums[right];
                
                if (sum == target) {
                    result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                    
                    // Skip duplicates para segundo número
                    while (left < right && nums[left] == nums[left + 1]) left++;
                    // Skip duplicates para tercer número
                    while (left < right && nums[right] == nums[right - 1]) right--;
                    
                    left++;
                    right--;
                } else if (sum < target) {
                    left++;
                } else {
                    right--;
                }
            }
        }
        
        return result;
    }
}
```

**Complejidad**:
- Time: O(n²) - O(n log n) sort + O(n²) two pointers
- Space: O(1) excluding output

#### **Ejemplo 3: Remove Duplicates from Sorted Array**

```java
/**
 * LeetCode 26: Remove Duplicates from Sorted Array
 * 
 * Input: [0,0,1,1,1,2,2,3,3,4]
 * Output: 5, nums = [0,1,2,3,4,_,_,_,_,_]
 */
public class RemoveDuplicates {
    
    public int removeDuplicates(int[] nums) {
        if (nums.length == 0) return 0;
        
        int slow = 0; // Position for next unique element
        
        for (int fast = 1; fast < nums.length; fast++) {
            if (nums[fast] != nums[slow]) {
                slow++;
                nums[slow] = nums[fast];
            }
        }
        
        return slow + 1; // Length of unique elements
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(1)

#### **Ejemplo 4: Container With Most Water**

```java
/**
 * LeetCode 11: Container With Most Water
 * 
 * Input: height = [1,8,6,2,5,4,8,3,7]
 * Output: 49
 */
public class ContainerWithMostWater {
    
    public int maxArea(int[] height) {
        int left = 0;
        int right = height.length - 1;
        int maxArea = 0;
        
        while (left < right) {
            // Area = width * min(height[left], height[right])
            int width = right - left;
            int h = Math.min(height[left], height[right]);
            maxArea = Math.max(maxArea, width * h);
            
            // Move pointer with smaller height (greedy choice)
            if (height[left] < height[right]) {
                left++;
            } else {
                right--;
            }
        }
        
        return maxArea;
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(1)

**Intuición**: Movemos el puntero con altura menor porque:
- Width siempre decrece al mover cualquier puntero
- Solo podemos aumentar área si encontramos mayor altura
- El lado más bajo limita la altura del container

### 📚 Cheat Sheet - LeetCode Problems

| Problema | Dificultad | Patrón |
|----------|-----------|--------|
| [167. Two Sum II](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) | Medium | Converging pointers |
| [15. 3Sum](https://leetcode.com/problems/3sum/) | Medium | Sort + Two pointers |
| [16. 3Sum Closest](https://leetcode.com/problems/3sum-closest/) | Medium | Variant of 3Sum |
| [26. Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) | Easy | Slow/Fast |
| [27. Remove Element](https://leetcode.com/problems/remove-element/) | Easy | Slow/Fast |
| [11. Container With Most Water](https://leetcode.com/problems/container-with-most-water/) | Medium | Greedy converging |
| [42. Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) | Hard | Two pointers + max tracking |
| [125. Valid Palindrome](https://leetcode.com/problems/valid-palindrome/) | Easy | Converging |

---

## 3. Fast & Slow Pointers

### 📌 Concepto

**Cuándo usar**: Detección de ciclos, encontrar middle element, problemas de LinkedList.

**Señales de identificación**:
- "Detect cycle in linked list"
- "Find middle of linked list"
- "Find start of cycle"
- "Check if linked list is palindrome"

**Core pattern**:
```java
ListNode slow = head;
ListNode fast = head;

while (fast != null && fast.next != null) {
    slow = slow.next;        // Move 1 step
    fast = fast.next.next;   // Move 2 steps
    
    if (slow == fast) {
        // Cycle detected or middle found
    }
}
```

### 💡 Ejemplos Prácticos

#### **Ejemplo 1: Linked List Cycle Detection**

```java
/**
 * LeetCode 141: Linked List Cycle
 * 
 * Detectar si hay ciclo en linked list
 */
public class LinkedListCycle {
    
    public boolean hasCycle(ListNode head) {
        if (head == null) return false;
        
        ListNode slow = head;
        ListNode fast = head;
        
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            
            if (slow == fast) {
                return true; // Cycle detected
            }
        }
        
        return false; // No cycle
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(1)

**Proof**: Si hay ciclo, fast alcanzará a slow en máximo n iteraciones.

#### **Ejemplo 2: Find Start of Cycle**

```java
/**
 * LeetCode 142: Linked List Cycle II
 * 
 * Encontrar nodo donde empieza el ciclo
 */
public class LinkedListCycleII {
    
    public ListNode detectCycle(ListNode head) {
        if (head == null) return null;
        
        // Phase 1: Detect cycle
        ListNode slow = head;
        ListNode fast = head;
        boolean hasCycle = false;
        
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            
            if (slow == fast) {
                hasCycle = true;
                break;
            }
        }
        
        if (!hasCycle) return null;
        
        // Phase 2: Find start of cycle
        // Mover slow a head, luego ambos avanzan 1 step
        // Se encuentran en el inicio del ciclo
        slow = head;
        while (slow != fast) {
            slow = slow.next;
            fast = fast.next;
        }
        
        return slow;
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(1)

**Matemática detrás**:
```
Sea:
- L = distancia de head al inicio del ciclo
- C = longitud del ciclo
- K = distancia del meeting point al inicio del ciclo

Cuando se encuentran:
- Slow recorrió: L + K
- Fast recorrió: L + K + nC (n vueltas completas)

Como Fast va 2x más rápido:
2(L + K) = L + K + nC
L + K = nC
L = nC - K

Por lo tanto, desde meeting point, avanzar C-K steps = avanzar L steps desde head
```

#### **Ejemplo 3: Middle of Linked List**

```java
/**
 * LeetCode 876: Middle of the Linked List
 * 
 * Input: 1->2->3->4->5
 * Output: 3->4->5
 */
public class MiddleOfLinkedList {
    
    public ListNode middleNode(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        
        // Si lista tiene par de nodos, devuelve el segundo middle
        return slow;
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(1)

#### **Ejemplo 4: Happy Number**

```java
/**
 * LeetCode 202: Happy Number
 * 
 * Un número es "happy" si al sumar cuadrados de sus dígitos
 * repetidamente, eventualmente llegas a 1.
 * 
 * Input: 19
 * 1² + 9² = 82
 * 8² + 2² = 68
 * 6² + 8² = 100
 * 1² + 0² + 0² = 1 ✓
 */
public class HappyNumber {
    
    private int getNext(int n) {
        int sum = 0;
        while (n > 0) {
            int digit = n % 10;
            sum += digit * digit;
            n /= 10;
        }
        return sum;
    }
    
    public boolean isHappy(int n) {
        int slow = n;
        int fast = getNext(n);
        
        // Si entra en ciclo (no happy), slow == fast
        // Si es happy, fast llegará a 1
        while (fast != 1 && slow != fast) {
            slow = getNext(slow);
            fast = getNext(getNext(fast));
        }
        
        return fast == 1;
    }
}
```

**Complejidad**:
- Time: O(log n) - número de dígitos disminuye
- Space: O(1)

### 📚 Cheat Sheet - LeetCode Problems

| Problema | Dificultad | Uso |
|----------|-----------|-----|
| [141. Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/) | Easy | Cycle detection |
| [142. Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/) | Medium | Find cycle start |
| [876. Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/) | Easy | Find middle |
| [234. Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/) | Easy | Find middle + reverse |
| [202. Happy Number](https://leetcode.com/problems/happy-number/) | Easy | Cycle in sequence |
| [143. Reorder List](https://leetcode.com/problems/reorder-list/) | Medium | Middle + reverse + merge |

---

## 4. Merge Intervals

### 📌 Concepto

**Cuándo usar**: Problemas con rangos/intervalos que se solapan o tocan.

**Señales de identificación**:
- "Merge overlapping intervals"
- "Insert interval into sorted list"
- "Find free time slots"
- Arrays de [start, end]

**Core pattern**:
```java
// 1. Sort intervals por start time
Arrays.sort(intervals, (a, b) -> a[0] - b[0]);

// 2. Iterate y merge si overlapping
for (int[] interval : intervals) {
    if (merged.isEmpty() || merged.getLast()[1] < interval[0]) {
        merged.add(interval); // No overlap
    } else {
        merged.getLast()[1] = Math.max(merged.getLast()[1], interval[1]); // Merge
    }
}
```

### 💡 Ejemplos Prácticos

#### **Ejemplo 1: Merge Intervals**

```java
/**
 * LeetCode 56: Merge Intervals
 * 
 * Input: [[1,3],[2,6],[8,10],[15,18]]
 * Output: [[1,6],[8,10],[15,18]]
 */
public class MergeIntervals {
    
    public int[][] merge(int[][] intervals) {
        if (intervals.length <= 1) return intervals;
        
        // Sort por start time
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
        
        List<int[]> merged = new ArrayList<>();
        merged.add(intervals[0]);
        
        for (int i = 1; i < intervals.length; i++) {
            int[] current = intervals[i];
            int[] last = merged.get(merged.size() - 1);
            
            // Check if overlapping: current.start <= last.end
            if (current[0] <= last[1]) {
                // Merge: extend last interval's end
                last[1] = Math.max(last[1], current[1]);
            } else {
                // No overlap: add new interval
                merged.add(current);
            }
        }
        
        return merged.toArray(new int[merged.size()][]);
    }
}
```

**Complejidad**:
- Time: O(n log n) - dominated by sort
- Space: O(n) - for output

#### **Ejemplo 2: Insert Interval**

```java
/**
 * LeetCode 57: Insert Interval
 * 
 * Input: intervals = [[1,3],[6,9]], newInterval = [2,5]
 * Output: [[1,5],[6,9]]
 */
public class InsertInterval {
    
    public int[][] insert(int[][] intervals, int[] newInterval) {
        List<int[]> result = new ArrayList<>();
        int i = 0;
        int n = intervals.length;
        
        // 1. Add all intervals que terminan antes de newInterval
        while (i < n && intervals[i][1] < newInterval[0]) {
            result.add(intervals[i]);
            i++;
        }
        
        // 2. Merge overlapping intervals con newInterval
        while (i < n && intervals[i][0] <= newInterval[1]) {
            newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
            newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
            i++;
        }
        result.add(newInterval);
        
        // 3. Add remaining intervals
        while (i < n) {
            result.add(intervals[i]);
            i++;
        }
        
        return result.toArray(new int[result.size()][]);
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(n)

#### **Ejemplo 3: Meeting Rooms II**

```java
/**
 * LeetCode 253: Meeting Rooms II (Premium)
 * 
 * Encontrar número mínimo de salas de conferencias necesarias.
 * 
 * Input: [[0,30],[5,10],[15,20]]
 * Output: 2 (dos meetings simultáneos en [5,10])
 */
public class MeetingRoomsII {
    
    public int minMeetingRooms(int[][] intervals) {
        if (intervals.length == 0) return 0;
        
        // Min heap ordenado por end time
        PriorityQueue<Integer> heap = new PriorityQueue<>();
        
        // Sort por start time
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
        
        // Add first meeting's end time
        heap.offer(intervals[0][1]);
        
        for (int i = 1; i < intervals.length; i++) {
            // Si earliest ending meeting terminó antes de este meeting
            if (intervals[i][0] >= heap.peek()) {
                heap.poll(); // Reuse room
            }
            
            // Add current meeting's end time
            heap.offer(intervals[i][1]);
        }
        
        // Heap size = número de salas necesarias
        return heap.size();
    }
    
    // Alternativa: Two arrays approach
    public int minMeetingRoomsTwoArrays(int[][] intervals) {
        int n = intervals.length;
        int[] starts = new int[n];
        int[] ends = new int[n];
        
        for (int i = 0; i < n; i++) {
            starts[i] = intervals[i][0];
            ends[i] = intervals[i][1];
        }
        
        Arrays.sort(starts);
        Arrays.sort(ends);
        
        int rooms = 0;
        int endPtr = 0;
        
        for (int i = 0; i < n; i++) {
            // Si un meeting empieza antes de que termine el earliest
            if (starts[i] < ends[endPtr]) {
                rooms++; // Need new room
            } else {
                endPtr++; // Reuse room
            }
        }
        
        return rooms;
    }
}
```

**Complejidad**:
- Time: O(n log n) - sort + heap operations
- Space: O(n) - heap

### 📚 Cheat Sheet - LeetCode Problems

| Problema | Dificultad | Concepto |
|----------|-----------|----------|
| [56. Merge Intervals](https://leetcode.com/problems/merge-intervals/) | Medium | Basic merge |
| [57. Insert Interval](https://leetcode.com/problems/insert-interval/) | Medium | Insert + merge |
| [252. Meeting Rooms](https://leetcode.com/problems/meeting-rooms/) | Easy | Check overlap |
| [253. Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/) | Medium | Min resources |
| [435. Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) | Medium | Min removals |
| [452. Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/) | Medium | Greedy intervals |

---

## 5. Cyclic Sort

### 📌 Concepto

**Cuándo usar**: Array con números en rango [1, n] o [0, n-1], encontrar missing/duplicate.

**Señales de identificación**:
- "Numbers from 1 to n"
- "Find missing number"
- "Find duplicate number"
- In-place, O(n) time requerido

**Core idea**: Cada número `num` debe estar en index `num - 1` (1-indexed) o `num` (0-indexed).

### 💡 Ejemplos Prácticos

#### **Ejemplo 1: Cyclic Sort**

```java
/**
 * Sort array [1, n] in-place usando ciclos
 * 
 * Input: [3, 1, 5, 4, 2]
 * Output: [1, 2, 3, 4, 5]
 */
public class CyclicSort {
    
    public void cyclicSort(int[] nums) {
        int i = 0;
        
        while (i < nums.length) {
            int correctIndex = nums[i] - 1; // num debería estar en index num-1
            
            if (nums[i] != nums[correctIndex]) {
                // Swap a posición correcta
                swap(nums, i, correctIndex);
            } else {
                i++;
            }
        }
    }
    
    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

**Complejidad**:
- Time: O(n) - cada elemento swapea máximo 1 vez
- Space: O(1)

#### **Ejemplo 2: Find Missing Number**

```java
/**
 * LeetCode 268: Missing Number
 * 
 * Array de n números en rango [0, n], encontrar el faltante.
 * 
 * Input: [3,0,1]
 * Output: 2
 */
public class MissingNumber {
    
    // Solución 1: Cyclic Sort
    public int missingNumber(int[] nums) {
        int i = 0;
        int n = nums.length;
        
        // Cyclic sort: cada num va a index num
        while (i < n) {
            int correctIndex = nums[i];
            
            if (nums[i] < n && nums[i] != nums[correctIndex]) {
                swap(nums, i, correctIndex);
            } else {
                i++;
            }
        }
        
        // Find first index donde nums[i] != i
        for (i = 0; i < n; i++) {
            if (nums[i] != i) {
                return i;
            }
        }
        
        return n; // Si todos están, falta n
    }
    
    // Solución 2: XOR (más elegante)
    public int missingNumberXOR(int[] nums) {
        int xor = nums.length;
        
        for (int i = 0; i < nums.length; i++) {
            xor ^= i ^ nums[i];
        }
        
        return xor;
    }
    
    // Solución 3: Gauss sum
    public int missingNumberMath(int[] nums) {
        int n = nums.length;
        int expectedSum = n * (n + 1) / 2;
        int actualSum = 0;
        
        for (int num : nums) {
            actualSum += num;
        }
        
        return expectedSum - actualSum;
    }
    
    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

**Complejidades**:
- Cyclic Sort: Time O(n), Space O(1)
- XOR: Time O(n), Space O(1)
- Math: Time O(n), Space O(1)

#### **Ejemplo 3: Find All Duplicates**

```java
/**
 * LeetCode 442: Find All Duplicates in an Array
 * 
 * Array de n enteros donde 1 ≤ nums[i] ≤ n.
 * Algunos aparecen dos veces, otros una.
 * 
 * Input: [4,3,2,7,8,2,3,1]
 * Output: [2,3]
 */
public class FindDuplicates {
    
    // Solución 1: Cyclic Sort
    public List<Integer> findDuplicates(int[] nums) {
        List<Integer> duplicates = new ArrayList<>();
        int i = 0;
        
        // Cyclic sort
        while (i < nums.length) {
            int correctIndex = nums[i] - 1;
            
            if (nums[i] != nums[correctIndex]) {
                swap(nums, i, correctIndex);
            } else {
                i++;
            }
        }
        
        // Find duplicates: números que no están en su posición correcta
        for (i = 0; i < nums.length; i++) {
            if (nums[i] != i + 1) {
                duplicates.add(nums[i]);
            }
        }
        
        return duplicates;
    }
    
    // Solución 2: Negate values (in-place marker)
    public List<Integer> findDuplicatesNegate(int[] nums) {
        List<Integer> duplicates = new ArrayList<>();
        
        for (int i = 0; i < nums.length; i++) {
            int index = Math.abs(nums[i]) - 1;
            
            // Si ya es negativo, es duplicado
            if (nums[index] < 0) {
                duplicates.add(index + 1);
            } else {
                nums[index] = -nums[index]; // Mark as seen
            }
        }
        
        return duplicates;
    }
    
    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(1) excluding output

### 📚 Cheat Sheet - LeetCode Problems

| Problema | Dificultad | Variante |
|----------|-----------|----------|
| [268. Missing Number](https://leetcode.com/problems/missing-number/) | Easy | Find single missing |
| [448. Find All Numbers Disappeared in an Array](https://leetcode.com/problems/find-all-numbers-disappeared-in-an-array/) | Easy | Find all missing |
| [287. Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/) | Medium | Find single duplicate |
| [442. Find All Duplicates in an Array](https://leetcode.com/problems/find-all-duplicates-in-an-array/) | Medium | Find all duplicates |
| [41. First Missing Positive](https://leetcode.com/problems/first-missing-positive/) | Hard | First missing positive |

---

## 6. In-place Reversal of LinkedList

### 📌 Concepto

**Cuándo usar**: Reversal de linked list completo o parcial.

**Señales de identificación**:
- "Reverse linked list"
- "Reverse nodes in k-group"
- "Swap nodes in pairs"

**Core pattern**:
```java
ListNode prev = null;
ListNode current = head;

while (current != null) {
    ListNode next = current.next;  // Save next
    current.next = prev;           // Reverse link
    prev = current;                // Move prev
    current = next;                // Move current
}

return prev; // New head
```

### 💡 Ejemplos Prácticos

#### **Ejemplo 1: Reverse Linked List**

```java
/**
 * LeetCode 206: Reverse Linked List
 * 
 * Input: 1->2->3->4->5->NULL
 * Output: 5->4->3->2->1->NULL
 */
public class ReverseLinkedList {
    
    // Iterative: O(n) time, O(1) space
    public ListNode reverseList(ListNode head) {
        ListNode prev = null;
        ListNode current = head;
        
        while (current != null) {
            ListNode next = current.next;
            current.next = prev;
            prev = current;
            current = next;
        }
        
        return prev;
    }
    
    // Recursive: O(n) time, O(n) space (call stack)
    public ListNode reverseListRecursive(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        
        ListNode newHead = reverseListRecursive(head.next);
        
        // Reverse link: next.next points back to current
        head.next.next = head;
        head.next = null;
        
        return newHead;
    }
}
```

**Complejidad**:
- Iterative: Time O(n), Space O(1)
- Recursive: Time O(n), Space O(n)

#### **Ejemplo 2: Reverse Linked List II**

```java
/**
 * LeetCode 92: Reverse Linked List II
 * 
 * Reverse nodes from position left to right.
 * 
 * Input: head = [1,2,3,4,5], left = 2, right = 4
 * Output: [1,4,3,2,5]
 */
public class ReverseLinkedListII {
    
    public ListNode reverseBetween(ListNode head, int left, int right) {
        if (head == null || left == right) return head;
        
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        ListNode prev = dummy;
        
        // 1. Move to node before left position
        for (int i = 0; i < left - 1; i++) {
            prev = prev.next;
        }
        
        // 2. Reverse [left, right]
        ListNode current = prev.next;
        ListNode next;
        
        for (int i = 0; i < right - left; i++) {
            next = current.next;
            current.next = next.next;
            next.next = prev.next;
            prev.next = next;
        }
        
        return dummy.next;
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(1)

#### **Ejemplo 3: Reverse Nodes in k-Group**

```java
/**
 * LeetCode 25: Reverse Nodes in k-Group (HARD)
 * 
 * Reverse nodes en grupos de k. Si quedan < k al final, dejar como están.
 * 
 * Input: head = [1,2,3,4,5], k = 2
 * Output: [2,1,4,3,5]
 */
public class ReverseNodesKGroup {
    
    public ListNode reverseKGroup(ListNode head, int k) {
        if (head == null || k == 1) return head;
        
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        ListNode prevGroupEnd = dummy;
        
        while (true) {
            // Check si hay k nodes restantes
            ListNode kthNode = getKthNode(prevGroupEnd, k);
            if (kthNode == null) break;
            
            ListNode nextGroupStart = kthNode.next;
            
            // Reverse current group
            ListNode prev = nextGroupStart;
            ListNode current = prevGroupEnd.next;
            
            while (current != nextGroupStart) {
                ListNode next = current.next;
                current.next = prev;
                prev = current;
                current = next;
            }
            
            // Connect with previous group
            ListNode temp = prevGroupEnd.next;
            prevGroupEnd.next = kthNode;
            prevGroupEnd = temp;
        }
        
        return dummy.next;
    }
    
    private ListNode getKthNode(ListNode current, int k) {
        while (current != null && k > 0) {
            current = current.next;
            k--;
        }
        return current;
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(1)

### 📚 Cheat Sheet - LeetCode Problems

| Problema | Dificultad | Concepto |
|----------|-----------|----------|
| [206. Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) | Easy | Full reversal |
| [92. Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/) | Medium | Partial reversal |
| [25. Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/) | Hard | Group reversal |
| [24. Swap Nodes in Pairs](https://leetcode.com/problems/swap-nodes-in-pairs/) | Medium | k=2 special case |
| [61. Rotate List](https://leetcode.com/problems/rotate-list/) | Medium | Circular + reverse |

---

## 7. Binary Search

### 📌 Concepto

**Cuándo usar**: Array ordenado, búsqueda de elemento/condición en O(log n).

**Señales de identificación**:
- "Find element in sorted array"
- "Find first/last occurrence"
- "Search in rotated array"
- "Find peak element"
- Minimize/maximize algo con búsqueda

**Template básico**:
```java
int left = 0, right = arr.length - 1;

while (left <= right) {
    int mid = left + (right - left) / 2; // Avoid overflow
    
    if (condition(mid)) {
        return mid; // or update result
    } else if (target < arr[mid]) {
        right = mid - 1;
    } else {
        left = mid + 1;
    }
}
```

### 💡 Ejemplos Prácticos

#### **Ejemplo 1: Binary Search Básico**

```java
/**
 * LeetCode 704: Binary Search
 */
public class BinarySearchBasic {
    
    public int search(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;
        
        while (left <= right) {
            int mid = left + (right - left) / 2;
            
            if (nums[mid] == target) {
                return mid;
            } else if (nums[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        
        return -1;
    }
}
```

**Complejidad**:
- Time: O(log n)
- Space: O(1)

#### **Ejemplo 2: First and Last Position**

```java
/**
 * LeetCode 34: Find First and Last Position of Element in Sorted Array
 * 
 * Input: nums = [5,7,7,8,8,10], target = 8
 * Output: [3,4]
 */
public class FirstLastPosition {
    
    public int[] searchRange(int[] nums, int target) {
        int[] result = {-1, -1};
        
        // Find first position
        result[0] = findBound(nums, target, true);
        
        // Only search for last if first exists
        if (result[0] != -1) {
            result[1] = findBound(nums, target, false);
        }
        
        return result;
    }
    
    private int findBound(int[] nums, int target, boolean isFirst) {
        int left = 0;
        int right = nums.length - 1;
        int result = -1;
        
        while (left <= right) {
            int mid = left + (right - left) / 2;
            
            if (nums[mid] == target) {
                result = mid;
                
                if (isFirst) {
                    right = mid - 1; // Continue searching left
                } else {
                    left = mid + 1;  // Continue searching right
                }
            } else if (nums[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        
        return result;
    }
}
```

**Complejidad**:
- Time: O(log n)
- Space: O(1)

#### **Ejemplo 3: Search in Rotated Sorted Array**

```java
/**
 * LeetCode 33: Search in Rotated Sorted Array
 * 
 * Array sorted pero rotado en algún pivot.
 * Input: nums = [4,5,6,7,0,1,2], target = 0
 * Output: 4
 */
public class SearchRotatedArray {
    
    public int search(int[] nums, int target) {
        int left = 0;
        int right = nums.length - 1;
        
        while (left <= right) {
            int mid = left + (right - left) / 2;
            
            if (nums[mid] == target) {
                return mid;
            }
            
            // Determine cuál mitad está ordenada
            if (nums[left] <= nums[mid]) {
                // Left half is sorted
                if (target >= nums[left] && target < nums[mid]) {
                    right = mid - 1; // Target in left half
                } else {
                    left = mid + 1;  // Target in right half
                }
            } else {
                // Right half is sorted
                if (target > nums[mid] && target <= nums[right]) {
                    left = mid + 1;  // Target in right half
                } else {
                    right = mid - 1; // Target in left half
                }
            }
        }
        
        return -1;
    }
}
```

**Complejidad**:
- Time: O(log n)
- Space: O(1)

**Key insight**: En cada iteración, al menos una mitad está ordenada.

#### **Ejemplo 4: Search 2D Matrix**

```java
/**
 * LeetCode 74: Search a 2D Matrix
 * 
 * Matrix donde cada row está sorted, y first element de cada row
 * es mayor que last element de row anterior.
 */
public class Search2DMatrix {
    
    public boolean searchMatrix(int[][] matrix, int target) {
        if (matrix.length == 0 || matrix[0].length == 0) return false;
        
        int rows = matrix.length;
        int cols = matrix[0].length;
        
        // Treat 2D matrix como 1D sorted array
        int left = 0;
        int right = rows * cols - 1;
        
        while (left <= right) {
            int mid = left + (right - left) / 2;
            
            // Convert 1D index to 2D coordinates
            int row = mid / cols;
            int col = mid % cols;
            int midValue = matrix[row][col];
            
            if (midValue == target) {
                return true;
            } else if (midValue < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        
        return false;
    }
}
```

**Complejidad**:
- Time: O(log(m * n))
- Space: O(1)

#### **Ejemplo 5: Koko Eating Bananas (Binary Search on Answer)**

```java
/**
 * LeetCode 875: Koko Eating Bananas
 * 
 * Koko tiene H horas para comer N pilas de bananas.
 * Cada hora puede comer K bananas de una pila.
 * Encontrar K mínimo para terminar en H horas.
 * 
 * Input: piles = [3,6,7,11], h = 8
 * Output: 4
 */
public class KokoEatingBananas {
    
    public int minEatingSpeed(int[] piles, int h) {
        int left = 1;
        int right = getMaxPile(piles);
        
        while (left < right) {
            int mid = left + (right - left) / 2;
            
            if (canFinish(piles, mid, h)) {
                right = mid; // Try smaller K
            } else {
                left = mid + 1; // Need larger K
            }
        }
        
        return left;
    }
    
    private boolean canFinish(int[] piles, int k, int h) {
        int hours = 0;
        
        for (int pile : piles) {
            hours += (pile + k - 1) / k; // Ceiling division
            if (hours > h) return false;
        }
        
        return hours <= h;
    }
    
    private int getMaxPile(int[] piles) {
        int max = 0;
        for (int pile : piles) {
            max = Math.max(max, pile);
        }
        return max;
    }
}
```

**Complejidad**:
- Time: O(n log m) donde m = max pile
- Space: O(1)

**Pattern**: Binary search on answer space [1, max_pile].

### 📚 Cheat Sheet - LeetCode Problems

| Problema | Dificultad | Pattern |
|----------|-----------|---------|
| [704. Binary Search](https://leetcode.com/problems/binary-search/) | Easy | Basic template |
| [34. Find First and Last Position](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/) | Medium | Find bounds |
| [33. Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/) | Medium | Rotated array |
| [74. Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/) | Medium | 2D → 1D |
| [153. Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) | Medium | Find pivot |
| [875. Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/) | Medium | BS on answer |
| [4. Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/) | Hard | Binary search partition |

---

## 8. Tree BFS (Breadth-First Search)

### 📌 Concepto

**Cuándo usar**: Traverse tree por niveles, encontrar shortest path en tree, level-order problems.

**Señales de identificación**:
- "Level-order traversal"
- "Find depth/height of tree"
- "Zigzag traversal"
- "Right side view"

**Core pattern**:
```java
Queue<TreeNode> queue = new LinkedList<>();
queue.offer(root);

while (!queue.isEmpty()) {
    int levelSize = queue.size();
    
    for (int i = 0; i < levelSize; i++) {
        TreeNode node = queue.poll();
        // Process node
        
        if (node.left != null) queue.offer(node.left);
        if (node.right != null) queue.offer(node.right);
    }
}
```

### 💡 Ejemplos Prácticos

#### **Ejemplo 1: Binary Tree Level Order Traversal**

```java
/**
 * LeetCode 102: Binary Tree Level Order Traversal
 * 
 * Input: [3,9,20,null,null,15,7]
 * Output: [[3],[9,20],[15,7]]
 */
public class LevelOrderTraversal {
    
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if (root == null) return result;
        
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        
        while (!queue.isEmpty()) {
            int levelSize = queue.size();
            List<Integer> currentLevel = new ArrayList<>();
            
            for (int i = 0; i < levelSize; i++) {
                TreeNode node = queue.poll();
                currentLevel.add(node.val);
                
                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }
            
            result.add(currentLevel);
        }
        
        return result;
    }
}
```

**Complejidad**:
- Time: O(n) - visit each node once
- Space: O(w) donde w = max width of tree

#### **Ejemplo 2: Binary Tree Zigzag Level Order**

```java
/**
 * LeetCode 103: Binary Tree Zigzag Level Order Traversal
 * 
 * Alternar dirección por nivel: L→R, R→L, L→R...
 */
public class ZigzagLevelOrder {
    
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if (root == null) return result;
        
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        boolean leftToRight = true;
        
        while (!queue.isEmpty()) {
            int levelSize = queue.size();
            LinkedList<Integer> currentLevel = new LinkedList<>();
            
            for (int i = 0; i < levelSize; i++) {
                TreeNode node = queue.poll();
                
                // Add to front or back depending on direction
                if (leftToRight) {
                    currentLevel.addLast(node.val);
                } else {
                    currentLevel.addFirst(node.val);
                }
                
                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }
            
            result.add(currentLevel);
            leftToRight = !leftToRight;
        }
        
        return result;
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(w)

#### **Ejemplo 3: Binary Tree Right Side View**

```java
/**
 * LeetCode 199: Binary Tree Right Side View
 * 
 * Retornar último nodo de cada nivel (vista desde derecha).
 * 
 * Input: [1,2,3,null,5,null,4]
 * Output: [1,3,4]
 */
public class RightSideView {
    
    public List<Integer> rightSideView(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        if (root == null) return result;
        
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);
        
        while (!queue.isEmpty()) {
            int levelSize = queue.size();
            
            for (int i = 0; i < levelSize; i++) {
                TreeNode node = queue.poll();
                
                // Last node of level
                if (i == levelSize - 1) {
                    result.add(node.val);
                }
                
                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }
        }
        
        return result;
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(w)

### 📚 Cheat Sheet - LeetCode Problems

| Problema | Dificultad | Concepto |
|----------|-----------|----------|
| [102. Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/) | Medium | Basic BFS |
| [103. Binary Tree Zigzag Level Order](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/) | Medium | Alternate direction |
| [107. Binary Tree Level Order Traversal II](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/) | Medium | Bottom-up |
| [199. Binary Tree Right Side View](https://leetcode.com/problems/binary-tree-right-side-view/) | Medium | Last of level |
| [111. Minimum Depth of Binary Tree](https://leetcode.com/problems/minimum-depth-of-binary-tree/) | Easy | Shortest path |
| [637. Average of Levels in Binary Tree](https://leetcode.com/problems/average-of-levels-in-binary-tree/) | Easy | Level statistics |

---

## 9. Tree DFS (Depth-First Search)

### 📌 Concepto

**Cuándo usar**: Path problems, subtree problems, tree traversals.

**Señales de identificación**:
- "Path from root to leaf"
- "All paths with sum X"
- "Maximum depth"
- "Validate BST"

**Three traversal types**:
1. **Preorder**: root → left → right (DFS discovery)
2. **Inorder**: left → root → right (BST sorted order)
3. **Postorder**: left → right → root (bottom-up processing)

**Core pattern**:
```java
public void dfs(TreeNode root) {
    if (root == null) return;
    
    // Preorder: process here
    dfs(root.left);
    // Inorder: process here
    dfs(root.right);
    // Postorder: process here
}
```

### 💡 Ejemplos Prácticos

#### **Ejemplo 1: Maximum Depth of Binary Tree**

```java
/**
 * LeetCode 104: Maximum Depth of Binary Tree
 */
public class MaxDepth {
    
    public int maxDepth(TreeNode root) {
        if (root == null) return 0;
        
        int leftDepth = maxDepth(root.left);
        int rightDepth = maxDepth(root.right);
        
        return 1 + Math.max(leftDepth, rightDepth);
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(h) donde h = height (call stack)

#### **Ejemplo 2: Path Sum**

```java
/**
 * LeetCode 112: Path Sum
 * 
 * Verificar si existe path root-to-leaf con sum = targetSum
 */
public class PathSum {
    
    public boolean hasPathSum(TreeNode root, int targetSum) {
        if (root == null) return false;
        
        // Leaf node
        if (root.left == null && root.right == null) {
            return root.val == targetSum;
        }
        
        int remaining = targetSum - root.val;
        return hasPathSum(root.left, remaining) || 
               hasPathSum(root.right, remaining);
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(h)

#### **Ejemplo 3: All Paths With Sum**

```java
/**
 * LeetCode 113: Path Sum II
 * 
 * Encontrar TODOS los paths root-to-leaf con sum = targetSum
 */
public class PathSumII {
    
    public List<List<Integer>> pathSum(TreeNode root, int targetSum) {
        List<List<Integer>> result = new ArrayList<>();
        List<Integer> currentPath = new ArrayList<>();
        dfs(root, targetSum, currentPath, result);
        return result;
    }
    
    private void dfs(TreeNode node, int remaining, 
                     List<Integer> currentPath, 
                     List<List<Integer>> result) {
        if (node == null) return;
        
        // Add current node to path
        currentPath.add(node.val);
        
        // Check if leaf with target sum
        if (node.left == null && node.right == null && 
            remaining == node.val) {
            result.add(new ArrayList<>(currentPath)); // Deep copy
        }
        
        // Recurse
        dfs(node.left, remaining - node.val, currentPath, result);
        dfs(node.right, remaining - node.val, currentPath, result);
        
        // Backtrack: remove current node
        currentPath.remove(currentPath.size() - 1);
    }
}
```

**Complejidad**:
- Time: O(n²) - O(n) traversal + O(n) copy paths
- Space: O(h)

#### **Ejemplo 4: Validate Binary Search Tree**

```java
/**
 * LeetCode 98: Validate Binary Search Tree
 * 
 * Verificar si árbol cumple propiedad BST:
 * left < root < right (para TODOS los ancestros)
 */
public class ValidateBST {
    
    public boolean isValidBST(TreeNode root) {
        return validate(root, null, null);
    }
    
    private boolean validate(TreeNode node, Integer min, Integer max) {
        if (node == null) return true;
        
        // Check BST property
        if ((min != null && node.val <= min) ||
            (max != null && node.val >= max)) {
            return false;
        }
        
        // Left subtree: all values < node.val
        // Right subtree: all values > node.val
        return validate(node.left, min, node.val) &&
               validate(node.right, node.val, max);
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(h)

#### **Ejemplo 5: Lowest Common Ancestor**

```java
/**
 * LeetCode 236: Lowest Common Ancestor of a Binary Tree
 * 
 * Encontrar ancestro común más bajo de p y q.
 */
public class LowestCommonAncestor {
    
    public TreeNode lowestCommonAncestor(TreeNode root, 
                                         TreeNode p, 
                                         TreeNode q) {
        // Base cases
        if (root == null || root == p || root == q) {
            return root;
        }
        
        // Search in subtrees
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);
        
        // Both found in different subtrees → root is LCA
        if (left != null && right != null) {
            return root;
        }
        
        // Return non-null result
        return left != null ? left : right;
    }
}
```

**Complejidad**:
- Time: O(n)
- Space: O(h)

### 📚 Cheat Sheet - LeetCode Problems

| Problema | Dificultad | Pattern |
|----------|-----------|---------|
| [104. Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/) | Easy | Basic DFS |
| [112. Path Sum](https://leetcode.com/problems/path-sum/) | Easy | Root-to-leaf |
| [113. Path Sum II](https://leetcode.com/problems/path-sum-ii/) | Medium | All paths + backtrack |
| [124. Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/) | Hard | Postorder max |
| [98. Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/) | Medium | BST property |
| [236. Lowest Common Ancestor](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/) | Medium | Postorder LCA |
| [543. Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/) | Easy | Postorder diameter |

---

## 10. Graphs (BFS/DFS)

### 📌 Concepto

**Cuándo usar**: Connectivity problems, shortest path, cycle detection, component counting.

**Señales de identificación**:
- "Number of islands"
- "Shortest path in graph"
- "Detect cycle"
- "Find all connected components"

**Graph representations**:
```java
// 1. Adjacency Matrix
int[][] graph = new int[n][n];

// 2. Adjacency List (preferred)
List<List<Integer>> graph = new ArrayList<>();

// 3. Edge List
int[][] edges = {{0,1}, {1,2}, {2,3}};
```

### 💡 Ejemplos Prácticos

#### **Ejemplo 1: Number of Islands (DFS)**

```java
/**
 * LeetCode 200: Number of Islands
 * 
 * Grid de '1' (land) y '0' (water).
 * Contar número de islas (conectadas horizontalmente/verticalmente).
 * 
 * Input:
 * 11110
 * 11010
 * 11000
 * 00000
 * 
 * Output: 1
 */
public class NumberOfIslands {
    
    private static final int[][] DIRECTIONS = {{0,1}, {1,0}, {0,-1}, {-1,0}};
    
    public int numIslands(char[][] grid) {
        if (grid == null || grid.length == 0) return 0;
        
        int rows = grid.length;
        int cols = grid[0].length;
        int count = 0;
        
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == '1') {
                    count++;
                    dfs(grid, r, c);
                }
            }
        }
        
        return count;
    }
    
    private void dfs(char[][] grid, int r, int c) {
        // Bounds check
        if (r < 0 || r >= grid.length || 
            c < 0 || c >= grid[0].length || 
            grid[r][c] != '1') {
            return;
        }
        
        // Mark as visited
        grid[r][c] = '0';
        
        // Visit all 4 directions
        for (int[] dir : DIRECTIONS) {
            dfs(grid, r + dir[0], c + dir[1]);
        }
    }
}
```

**Complejidad**:
- Time: O(m × n)
- Space: O(m × n) worst case (call stack)

#### **Ejemplo 2: Clone Graph**

```java
/**
 * LeetCode 133: Clone Graph
 * 
 * Deep clone de un grafo undirected.
 */
class Node {
    public int val;
    public List<Node> neighbors;
    
    public Node(int val) {
        this.val = val;
        this.neighbors = new ArrayList<>();
    }
}

public class CloneGraph {
    
    private Map<Node, Node> visited = new HashMap<>();
    
    public Node cloneGraph(Node node) {
        if (node == null) return null;
        
        // If already cloned, return clone
        if (visited.containsKey(node)) {
            return visited.get(node);
        }
        
        // Create clone
        Node clone = new Node(node.val);
        visited.put(node, clone);
        
        // Clone neighbors recursively
        for (Node neighbor : node.neighbors) {
            clone.neighbors.add(cloneGraph(neighbor));
        }
        
        return clone;
    }
}
```

**Complejidad**:
- Time: O(V + E)
- Space: O(V)

#### **Ejemplo 3: Course Schedule (Cycle Detection)**

```java
/**
 * LeetCode 207: Course Schedule
 * 
 * Detectar si es posible completar todos los cursos
 * (no hay ciclos en dependency graph).
 * 
 * Input: numCourses = 2, prerequisites = [[1,0]]
 * Output: true (tomar curso 0 primero, luego 1)
 */
public class CourseSchedule {
    
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        // Build adjacency list
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < numCourses; i++) {
            graph.add(new ArrayList<>());
        }
        
        for (int[] prereq : prerequisites) {
            int course = prereq[0];
            int prerequisite = prereq[1];
            graph.get(course).add(prerequisite);
        }
        
        // 0 = unvisited, 1 = visiting, 2 = visited
        int[] state = new int[numCourses];
        
        for (int i = 0; i < numCourses; i++) {
            if (hasCycle(graph, i, state)) {
                return false; // Cycle detected
            }
        }
        
        return true;
    }
    
    private boolean hasCycle(List<List<Integer>> graph, 
                             int course, 
                             int[] state) {
        if (state[course] == 1) return true;  // Cycle!
        if (state[course] == 2) return false; // Already processed
        
        state[course] = 1; // Mark as visiting
        
        for (int prereq : graph.get(course)) {
            if (hasCycle(graph, prereq, state)) {
                return true;
            }
        }
        
        state[course] = 2; // Mark as visited
        return false;
    }
}
```

**Complejidad**:
- Time: O(V + E)
- Space: O(V + E)

#### **Ejemplo 4: Shortest Path in Binary Matrix (BFS)**

```java
/**
 * LeetCode 1091: Shortest Path in Binary Matrix
 * 
 * Encontrar shortest path desde (0,0) hasta (n-1,n-1).
 * Permitido moverse en 8 direcciones.
 */
public class ShortestPathBinaryMatrix {
    
    private static final int[][] DIRECTIONS = {
        {0,1}, {1,0}, {0,-1}, {-1,0},  // 4 cardinal
        {1,1}, {1,-1}, {-1,1}, {-1,-1} // 4 diagonal
    };
    
    public int shortestPathBinaryMatrix(int[][] grid) {
        int n = grid.length;
        
        if (grid[0][0] == 1 || grid[n-1][n-1] == 1) {
            return -1; // Start or end blocked
        }
        
        Queue<int[]> queue = new LinkedList<>();
        queue.offer(new int[]{0, 0, 1}); // {row, col, distance}
        grid[0][0] = 1; // Mark as visited
        
        while (!queue.isEmpty()) {
            int[] curr = queue.poll();
            int r = curr[0], c = curr[1], dist = curr[2];
            
            // Reached destination
            if (r == n - 1 && c == n - 1) {
                return dist;
            }
            
            // Explore 8 directions
            for (int[] dir : DIRECTIONS) {
                int nr = r + dir[0];
                int nc = c + dir[1];
                
                if (nr >= 0 && nr < n && nc >= 0 && nc < n && 
                    grid[nr][nc] == 0) {
                    queue.offer(new int[]{nr, nc, dist + 1});
                    grid[nr][nc] = 1; // Mark as visited
                }
            }
        }
        
        return -1; // No path found
    }
}
```

**Complejidad**:
- Time: O(n²)
- Space: O(n²)

### 📚 Cheat Sheet - LeetCode Problems

| Problema | Dificultad | Algoritmo |
|----------|-----------|-----------|
| [200. Number of Islands](https://leetcode.com/problems/number-of-islands/) | Medium | DFS/BFS flood fill |
| [133. Clone Graph](https://leetcode.com/problems/clone-graph/) | Medium | DFS + HashMap |
| [207. Course Schedule](https://leetcode.com/problems/course-schedule/) | Medium | Cycle detection |
| [210. Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) | Medium | Topological sort |
| [1091. Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | Medium | BFS shortest path |
| [994. Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) | Medium | Multi-source BFS |
| [417. Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/) | Medium | DFS from borders |

---

**[⬅️ Volver al índice principal](./README.md)** | **[Continúa: Backtracking & DP →](./12-dsa-patterns.md)**
