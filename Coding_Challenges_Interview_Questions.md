# Coding Challenges Interview Questions

This file contains programming problems organized by language and problem type.

---

## Table of Contents
1. [TypeScript Challenges](#typescript-challenges)
2. [JavaScript Challenges](#javascript-challenges)
3. [Java Challenges](#java-challenges)
4. [Algorithm Challenges](#algorithm-challenges)
5. [Data Structure Challenges](#data-structure-challenges)

---

## TypeScript Challenges

### 1. Gym Membership System with HashMap
**Problem:** Create a gym membership system where each member can track their workout sessions. Implement methods to add workouts and calculate average workout durations using a HashMap-like structure.

**Requirements:**
- `Member` class with name and workout tracking
- `Workout` class with type and duration
- `addWorkout()` method to add a workout session
- `getAverageWorkoutDurations()` method to calculate average duration per workout type

**Solution:**
```typescript
// Workout class to represent each workout session
class Workout {
    constructor(
        public type: string,  // e.g., "cardio", "strength", "yoga"
        public duration: number  // duration in minutes
    ) {}
}

// Member class with HashMap-like structure for workouts
class Member {
    private name: string;
    private workouts: Map<string, number[]>; // workout type -> array of durations
    
    constructor(name: string) {
        this.name = name;
        this.workouts = new Map<string, number[]>();
    }
    
    // Add a workout session
    addWorkout(workout: Workout): void {
        if (this.workouts.has(workout.type)) {
            // If workout type exists, add duration to existing array
            this.workouts.get(workout.type)!.push(workout.duration);
        } else {
            // If new workout type, create new array
            this.workouts.set(workout.type, [workout.duration]);
        }
        console.log(`Added ${workout.type} workout (${workout.duration} mins) for ${this.name}`);
    }
    
    // Get average duration for each workout type
    getAverageWorkoutDurations(): Map<string, number> {
        const averages = new Map<string, number>();
        
        this.workouts.forEach((durations, workoutType) => {
            const total = durations.reduce((sum, duration) => sum + duration, 0);
            const average = total / durations.length;
            averages.set(workoutType, Math.round(average * 10) / 10); // Round to 1 decimal
        });
        
        return averages;
    }
    
    // Display member's workout summary
    displaySummary(): void {
        console.log(`\n=== Workout Summary for ${this.name} ===`);
        const averages = this.getAverageWorkoutDurations();
        
        averages.forEach((avgDuration, workoutType) => {
            const sessions = this.workouts.get(workoutType)!.length;
            console.log(`${workoutType}: ${sessions} session(s), avg ${avgDuration} mins`);
        });
    }
}

// Usage Example
const member1 = new Member("John Doe");

// Add workout sessions
member1.addWorkout(new Workout("cardio", 30));
member1.addWorkout(new Workout("strength", 45));
member1.addWorkout(new Workout("cardio", 40));
member1.addWorkout(new Workout("yoga", 60));
member1.addWorkout(new Workout("strength", 50));
member1.addWorkout(new Workout("cardio", 35));

// Display summary
member1.displaySummary();

// Output:
// Added cardio workout (30 mins) for John Doe
// Added strength workout (45 mins) for John Doe
// Added cardio workout (40 mins) for John Doe
// Added yoga workout (60 mins) for John Doe
// Added strength workout (50 mins) for John Doe
// Added cardio workout (35 mins) for John Doe
//
// === Workout Summary for John Doe ===
// cardio: 3 session(s), avg 35 mins
// strength: 2 session(s), avg 47.5 mins
// yoga: 1 session(s), avg 60 mins
```

**Key Concepts:**
- **Map (HashMap):** TypeScript's `Map` provides key-value storage with O(1) access
- **Generics:** `Map<string, number[]>` ensures type safety
- **Array Methods:** `reduce()` for calculating totals
- **Object-Oriented Design:** Encapsulation of workout tracking logic

---

### 2. Remove Duplicates from Array of Objects
**Problem:** Given an array of products, remove duplicate products based on the product ID while preserving the most recent entry.

**Solution:**
```typescript
interface Product {
    id: number;
    name: string;
    price: number;
    lastUpdated: Date;
}

// Method 1: Using Map (preserves last occurrence)
function removeDuplicatesWithMap(products: Product[]): Product[] {
    const productMap = new Map<number, Product>();
    
    // Map automatically overwrites duplicates with the latest entry
    products.forEach(product => {
        productMap.set(product.id, product);
    });
    
    // Convert Map values back to array
    return Array.from(productMap.values());
}

// Method 2: Using Set with custom logic (preserves first occurrence)
function removeDuplicatesWithSet(products: Product[]): Product[] {
    const seenIds = new Set<number>();
    const uniqueProducts: Product[] = [];
    
    products.forEach(product => {
        if (!seenIds.has(product.id)) {
            seenIds.add(product.id);
            uniqueProducts.push(product);
        }
    });
    
    return uniqueProducts;
}

// Method 3: Using reduce (most flexible)
function removeDuplicatesWithReduce(products: Product[]): Product[] {
    const productMap = products.reduce((map, product) => {
        map.set(product.id, product);
        return map;
    }, new Map<number, Product>());
    
    return Array.from(productMap.values());
}

// Usage Example
const products: Product[] = [
    { id: 1, name: "Laptop", price: 1000, lastUpdated: new Date('2024-01-01') },
    { id: 2, name: "Mouse", price: 25, lastUpdated: new Date('2024-01-02') },
    { id: 1, name: "Laptop Pro", price: 1200, lastUpdated: new Date('2024-01-03') }, // Duplicate
    { id: 3, name: "Keyboard", price: 75, lastUpdated: new Date('2024-01-04') },
    { id: 2, name: "Wireless Mouse", price: 30, lastUpdated: new Date('2024-01-05') } // Duplicate
];

console.log("Original Products:", products.length); // 5

const uniqueProductsMap = removeDuplicatesWithMap(products);
console.log("After removing duplicates (Map):", uniqueProductsMap.length); // 3
console.log(uniqueProductsMap);

// Output (preserves last occurrence):
// [
//   { id: 1, name: "Laptop Pro", price: 1200, ... },
//   { id: 3, name: "Keyboard", price: 75, ... },
//   { id: 2, name: "Wireless Mouse", price: 30, ... }
// ]
```

**Performance Comparison:**

| Method | Time Complexity | Space Complexity | Preserves |
|--------|----------------|------------------|-----------|
| Map | O(n) | O(n) | Last occurrence |
| Set | O(n) | O(n) | First occurrence |
| reduce | O(n) | O(n) | Last occurrence |

---

## JavaScript Challenges

### 3. Two Sum Problem
**Problem:** Given an array of integers `nums` and an integer `target`, return indices of the two numbers that add up to `target`.

**Solution:**
```javascript
// Method 1: Brute Force - O(n²)
function twoSumBruteForce(nums, target) {
    for (let i = 0; i < nums.length; i++) {
        for (let j = i + 1; j < nums.length; j++) {
            if (nums[i] + nums[j] === target) {
                return [i, j];
            }
        }
    }
    return [];
}

// Method 2: HashMap - O(n) (Optimal)
function twoSum(nums, target) {
    const map = new Map(); // value -> index
    
    for (let i = 0; i < nums.length; i++) {
        const complement = target - nums[i];
        
        if (map.has(complement)) {
            return [map.get(complement), i];
        }
        
        map.set(nums[i], i);
    }
    
    return [];
}

// Test cases
console.log(twoSum([2, 7, 11, 15], 9));     // [0, 1] (2 + 7 = 9)
console.log(twoSum([3, 2, 4], 6));          // [1, 2] (2 + 4 = 6)
console.log(twoSum([3, 3], 6));             // [0, 1] (3 + 3 = 6)
```

**How it works:**
1. Iterate through array once
2. For each number, calculate what value we need (complement = target - current)
3. Check if complement exists in HashMap
4. If yes, return indices; if no, store current number and index in HashMap

---

### 4. Three Sum Problem
**Problem:** Given an array `nums`, find all unique triplets that sum to zero.

**Solution:**
```javascript
function threeSum(nums) {
    const result = [];
    nums.sort((a, b) => a - b); // Sort array first
    
    for (let i = 0; i < nums.length - 2; i++) {
        // Skip duplicates for first element
        if (i > 0 && nums[i] === nums[i - 1]) continue;
        
        let left = i + 1;
        let right = nums.length - 1;
        
        while (left < right) {
            const sum = nums[i] + nums[left] + nums[right];
            
            if (sum === 0) {
                result.push([nums[i], nums[left], nums[right]]);
                
                // Skip duplicates for second element
                while (left < right && nums[left] === nums[left + 1]) left++;
                // Skip duplicates for third element
                while (left < right && nums[right] === nums[right - 1]) right--;
                
                left++;
                right--;
            } else if (sum < 0) {
                left++; // Need larger sum
            } else {
                right--; // Need smaller sum
            }
        }
    }
    
    return result;
}

// Test cases
console.log(threeSum([-1, 0, 1, 2, -1, -4]));
// Output: [[-1, -1, 2], [-1, 0, 1]]

console.log(threeSum([0, 0, 0, 0]));
// Output: [[0, 0, 0]]
```

**Time Complexity:** O(n²) - One loop + two pointers
**Space Complexity:** O(1) - Not counting output array

---

## Algorithm Challenges

### 5. Dutch Flag Algorithm (Sort 0s, 1s, and 2s)
**Problem:** Given an array containing only 0s, 1s, and 2s, sort it in-place so that:
- All 0s go to the left
- All 1s stay in the center
- All 2s go to the right

**Solution:**
```javascript
function dutchFlagSort(arr) {
    let low = 0;      // Pointer for 0s
    let mid = 0;      // Pointer for 1s (current element)
    let high = arr.length - 1;  // Pointer for 2s
    
    while (mid <= high) {
        if (arr[mid] === 0) {
            // Swap with low pointer and move both forward
            [arr[low], arr[mid]] = [arr[mid], arr[low]];
            low++;
            mid++;
        } else if (arr[mid] === 1) {
            // 1 is in correct position, just move forward
            mid++;
        } else {
            // arr[mid] === 2
            // Swap with high pointer and move high backward
            [arr[mid], arr[high]] = [arr[high], arr[mid]];
            high--;
            // Don't increment mid because we need to check swapped element
        }
    }
    
    return arr;
}

// Test cases
console.log(dutchFlagSort([2, 0, 2, 1, 1, 0]));
// Output: [0, 0, 1, 1, 2, 2]

console.log(dutchFlagSort([2, 0, 1]));
// Output: [0, 1, 2]

console.log(dutchFlagSort([0, 0, 0, 1, 1, 2, 2, 2]));
// Output: [0, 0, 0, 1, 1, 2, 2, 2]

console.log(dutchFlagSort([2, 2, 2, 0, 0, 1]));
// Output: [0, 0, 1, 2, 2, 2]
```

**How it works (Three Pointers):**

```
Initial: [2, 0, 2, 1, 1, 0]
         low/mid           high

Step 1: arr[mid]=2, swap with high
        [0, 0, 2, 1, 1, 2]
         low/mid       high

Step 2: arr[mid]=0, swap with low
        [0, 0, 2, 1, 1, 2]
           low mid    high

Step 3: arr[mid]=0, swap with low
        [0, 0, 2, 1, 1, 2]
             low mid  high

Continue until mid > high...
Final:  [0, 0, 1, 1, 2, 2]
```

**Visual Representation:**
```
Partitions:
[0 0 ... 0] [1 1 ... 1] [2 2 ... 2]
 ↑           ↑           ↑
 low         mid         high
```

**Time Complexity:** O(n) - Single pass
**Space Complexity:** O(1) - In-place sorting

---

## Java Challenges

### 6. Remove Duplicates from ArrayList
**Problem:** Remove duplicate integers from an ArrayList while preserving order.

**Solution:**
```java
import java.util.*;

public class RemoveDuplicates {
    
    // Method 1: Using LinkedHashSet (preserves order)
    public static List<Integer> removeDuplicatesWithSet(List<Integer> list) {
        // LinkedHashSet maintains insertion order
        Set<Integer> set = new LinkedHashSet<>(list);
        return new ArrayList<>(set);
    }
    
    // Method 2: Using Stream (Java 8+)
    public static List<Integer> removeDuplicatesWithStream(List<Integer> list) {
        return list.stream()
            .distinct()
            .collect(Collectors.toList());
    }
    
    // Method 3: Manual approach with Set
    public static List<Integer> removeDuplicatesManual(List<Integer> list) {
        Set<Integer> seen = new HashSet<>();
        List<Integer> result = new ArrayList<>();
        
        for (Integer num : list) {
            if (!seen.contains(num)) {
                seen.add(num);
                result.add(num);
            }
        }
        
        return result;
    }
    
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 2, 4, 1, 5, 3, 6);
        
        System.out.println("Original: " + numbers);
        // Output: [1, 2, 3, 2, 4, 1, 5, 3, 6]
        
        System.out.println("Using Set: " + removeDuplicatesWithSet(numbers));
        // Output: [1, 2, 3, 4, 5, 6]
        
        System.out.println("Using Stream: " + removeDuplicatesWithStream(numbers));
        // Output: [1, 2, 3, 4, 5, 6]
    }
}
```

---

### 7. Remove Duplicates from Array of Objects (Java)
**Problem:** Remove duplicate products based on product ID.

**Solution:**
```java
import java.util.*;
import java.util.stream.Collectors;

class Product {
    private int id;
    private String name;
    private double price;
    
    public Product(int id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }
    
    public int getId() { return id; }
    public String getName() { return name; }
    public double getPrice() { return price; }
    
    @Override
    public String toString() {
        return String.format("Product{id=%d, name='%s', price=%.2f}", id, name, price);
    }
}

public class ProductDeduplication {
    
    // Method 1: Using Map (preserves last occurrence)
    public static List<Product> removeDuplicatesWithMap(List<Product> products) {
        Map<Integer, Product> productMap = new LinkedHashMap<>();
        
        for (Product product : products) {
            productMap.put(product.getId(), product);
        }
        
        return new ArrayList<>(productMap.values());
    }
    
    // Method 2: Using Stream (Java 8+)
    public static List<Product> removeDuplicatesWithStream(List<Product> products) {
        return products.stream()
            .collect(Collectors.toMap(
                Product::getId,
                product -> product,
                (existing, replacement) -> replacement, // Keep last occurrence
                LinkedHashMap::new
            ))
            .values()
            .stream()
            .collect(Collectors.toList());
    }
    
    public static void main(String[] args) {
        List<Product> products = Arrays.asList(
            new Product(1, "Laptop", 1000.0),
            new Product(2, "Mouse", 25.0),
            new Product(1, "Laptop Pro", 1200.0),  // Duplicate ID
            new Product(3, "Keyboard", 75.0),
            new Product(2, "Wireless Mouse", 30.0) // Duplicate ID
        );
        
        System.out.println("Original Products: " + products.size());
        products.forEach(System.out::println);
        
        System.out.println("\nAfter removing duplicates: ");
        List<Product> unique = removeDuplicatesWithMap(products);
        unique.forEach(System.out::println);
        
        // Output:
        // Product{id=1, name='Laptop Pro', price=1200.00}
        // Product{id=2, name='Wireless Mouse', price=30.00}
        // Product{id=3, name='Keyboard', price=75.00}
    }
}
```

---

## Data Structure Challenges

### 8. Implement LRU Cache
**Problem:** Design a Least Recently Used (LRU) cache with O(1) operations.

**Solution (JavaScript):**
```javascript
class LRUCache {
    constructor(capacity) {
        this.capacity = capacity;
        this.cache = new Map(); // Map maintains insertion order
    }
    
    get(key) {
        if (!this.cache.has(key)) {
            return -1;
        }
        
        // Move accessed item to end (most recently used)
        const value = this.cache.get(key);
        this.cache.delete(key);
        this.cache.set(key, value);
        
        return value;
    }
    
    put(key, value) {
        // If key exists, delete it first
        if (this.cache.has(key)) {
            this.cache.delete(key);
        }
        
        // Add to end (most recently used)
        this.cache.set(key, value);
        
        // If capacity exceeded, remove least recently used (first item)
        if (this.cache.size > this.capacity) {
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);
        }
    }
}

// Usage
const lru = new LRUCache(2);

lru.put(1, 1);  // cache: {1=1}
lru.put(2, 2);  // cache: {1=1, 2=2}
console.log(lru.get(1));  // 1, cache: {2=2, 1=1}
lru.put(3, 3);  // cache: {1=1, 3=3}, evicts 2
console.log(lru.get(2));  // -1 (not found)
```

---

### 9. Find First Non-Repeating Character
**Problem:** Find the first character in a string that doesn't repeat.

**Solution:**
```javascript
function firstNonRepeating(str) {
    const charCount = new Map();
    
    // Count occurrences
    for (const char of str) {
        charCount.set(char, (charCount.get(char) || 0) + 1);
    }
    
    // Find first with count 1
    for (const char of str) {
        if (charCount.get(char) === 1) {
            return char;
        }
    }
    
    return null;
}

console.log(firstNonRepeating("leetcode"));  // "l"
console.log(firstNonRepeating("loveleetcode"));  // "v"
console.log(firstNonRepeating("aabb"));  // null
```

---

## Summary

| Challenge | Data Structure | Time Complexity | Key Technique |
|-----------|---------------|----------------|---------------|
| **Gym Membership** | Map | O(n) | HashMap for grouping |
| **Remove Duplicates** | Map/Set | O(n) | Hash-based deduplication |
| **Two Sum** | Map | O(n) | Complement lookup |
| **Three Sum** | Two Pointers | O(n²) | Sorted array + pointers |
| **Dutch Flag** | Three Pointers | O(n) | In-place partitioning |
| **LRU Cache** | Map (ordered) | O(1) | Ordered hash map |
| **First Non-Repeating** | Map | O(n) | Character frequency |

---

## Tips for Coding Interviews

1. **Clarify Requirements:** Ask about edge cases, input constraints
2. **Think Out Loud:** Explain your approach before coding
3. **Start Simple:** Brute force first, then optimize
4. **Test:** Walk through test cases manually
5. **Time/Space Complexity:** Always analyze both

**Common Patterns:**
- **Two Pointers:** Sorted arrays, string problems
- **HashMap:** Frequency counting, fast lookups
- **Sliding Window:** Subarray/substring problems
- **DFS/BFS:** Tree/graph traversal
- **Dynamic Programming:** Optimization problems with overlapping subproblems
