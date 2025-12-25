# JavaScript Interview Questions

## 1. var, let, const
**Concept:** These are keywords used to declare variables.
- **`var`**: Function-scoped or globally scoped. Can be re-declared and updated. Hoisted with `undefined`.
- **`let`**: Block-scoped. Can be updated but not re-declared in the same scope. Hoisted but in the "Temporal Dead Zone".
- **`const`**: Block-scoped. Cannot be updated or re-declared. Must be initialized during declaration. However, object properties declared with `const` can be mutated.

**Example:**
```javascript
// var
if (true) {
  var x = 10;
}
console.log(x); // 10 (accessible outside block)

// let & const
if (true) {
  let y = 20;
  const z = 30;
}
// console.log(y); // ReferenceError: y is not defined
```

**Related Topics:** Hoisting, Scope, Temporal Dead Zone (TDZ).

### Scope (Global, Function, Block)
**Concept:** Scope determines the accessibility of variables.
- **Global Scope:** Accessible everywhere.
- **Function Scope:** Accessible only within the function (`var`).
- **Block Scope:** Accessible only within the block `{}` (`let`, `const`).

**Example:**
```javascript
var globalVar = "I am global";

function scopeExample() {
  var functionVar = "I am function scoped";
  if (true) {
    let blockVar = "I am block scoped";
    console.log(blockVar); // Accessible
  }
  // console.log(blockVar); // Error: blockVar is not defined
}
```

## 2. Hoisting & Temporal Dead Zone (TDZ)
**Concept:**
- **Hoisting**: JavaScript's behavior of moving declarations to the top of the current scope (script or function). Only declarations are hoisted, not initializations.
- **TDZ**: The period between the start of the block and the point where the variable is declared. Accessing a `let` or `const` variable in the TDZ throws a `ReferenceError`.

**Example:**
```javascript
console.log(a); // undefined (Hoisted)
var a = 5;

console.log(b); // ReferenceError: Cannot access 'b' before initialization (TDZ)
let b = 10;
```

## 3. Closures
**Concept:** A closure is a function bundled together with references to its surrounding state (the lexical environment). It allows an inner function to access the scope of an outer function even after the outer function has finished executing.

**Example:**
```javascript
function outer() {
  let count = 0;
  return function inner() {
    count++;
    return count;
  };
}

const counter = outer();
console.log(counter()); // 1
console.log(counter()); // 2
```
**Use Cases:** Data privacy (emulating private methods), Currying, Memoization.

### Currying
**Concept:** Transforming a function that takes multiple arguments into a sequence of functions that each take a single argument.

**Example:**
```javascript
function multiply(a) {
  return function(b) {
    return a * b;
  };
}
const multiplyByTwo = multiply(2);
console.log(multiplyByTwo(5)); // 10
```

### Memoization
**Concept:** Optimization technique to cache results of expensive function calls.

**Example:**
```javascript
function memoizedAdd() {
  let cache = {};
  return function(n) {
    if (n in cache) {
      console.log('Fetching from cache');
      return cache[n];
    } else {
      console.log('Calculating result');
      let result = n + 10;
      cache[n] = result;
      return result;
    }
  };
}
const newAdd = memoizedAdd();
console.log(newAdd(9)); // Calculating result, 19
console.log(newAdd(9)); // Fetching from cache, 19
```

## 4. Arrow vs Normal Functions & `this` Behavior
**Concept:**
- **Normal Function**: `this` depends on *how* the function is called (dynamic scoping).
- **Arrow Function**: `this` is lexically inherited from the surrounding scope at the time of definition. It does not have its own `this`.

**Example:**
```javascript
const obj = {
  name: 'Alice',
  normal: function() {
    console.log(this.name); // 'Alice'
  },
  arrow: () => {
    console.log(this.name); // undefined (inherits from global/window)
  }
};
```
**Note:** Arrow functions cannot be used as constructors and do not have the `arguments` object.

**Example (Constructor Limitation):**
```javascript
const Person = (name) => {
  this.name = name;
};
// const p = new Person('Alice'); // TypeError: Person is not a constructor
```

## 5. Call, Apply, Bind
**Concept:** Methods to explicitly set the value of `this`.
- **`call(thisArg, arg1, arg2)`**: Invokes the function immediately with a specified `this` and arguments provided individually.
- **`apply(thisArg, [args])`**: Invokes the function immediately with a specified `this` and arguments provided as an array.
- **`bind(thisArg, arg1)`**: Returns a *new function* with `this` permanently bound. Does not invoke immediately.

**Example:**
```javascript
function greet(greeting) {
  console.log(`${greeting}, ${this.name}`);
}
const person = { name: 'John' };

greet.call(person, 'Hello');      // Hello, John
greet.apply(person, ['Hi']);      // Hi, John
const greetJohn = greet.bind(person, 'Welcome');
greetJohn();                      // Welcome, John
```

## 6. Shallow Copy vs Deep Copy
**Concept:**
- **Shallow Copy**: Copies the top-level properties. Nested objects are copied by reference (changes affect original).
- **Deep Copy**: Copies all levels of the object recursively. New memory is allocated for nested objects.

**Example:**
```javascript
const original = { a: 1, b: { c: 2 } };

// Shallow Copy
const shallow = { ...original };
shallow.b.c = 99;
console.log(original.b.c); // 99 (Affected)

// Deep Copy
const deep = JSON.parse(JSON.stringify(original));
// Or structuredClone(original) in modern browsers
deep.b.c = 100;
console.log(original.b.c); // 99 (Not Affected)
```

## 7. Event Loop & Asynchronous JavaScript
**Concept:** JavaScript is single-threaded. The Event Loop handles asynchronous operations.
1.  **Call Stack**: Executes synchronous code.
2.  **Web APIs**: Handles async tasks (setTimeout, fetch).
3.  **Callback Queue (Task Queue)**: Holds callbacks (e.g., setTimeout).
4.  **Microtask Queue**: Holds Promises/MutationObserver callbacks (Higher priority than Task Queue).

**Order:** Call Stack -> Microtasks -> Macrotasks (Callback Queue).

**Example:**
```javascript
console.log('Start');
setTimeout(() => console.log('Timeout'), 0);
Promise.resolve().then(() => console.log('Promise'));
console.log('End');

// Output: Start, End, Promise, Timeout
```

## 8. Event Delegation
**Concept:** Instead of attaching event listeners to specific nodes, you attach a single listener to a parent element. This works because of **Event Bubbling** (events propagate up the DOM).

**Example:**
```javascript
document.getElementById('parent-list').addEventListener('click', (e) => {
  if (e.target.tagName === 'LI') {
    console.log('List item clicked:', e.target.textContent);
  }
});
```
**Benefits:** Less memory usage, handles dynamically added elements automatically.

## 9. Coding Challenges

### Dutch National Flag Problem (Sort 0, 1, 2)
**Problem:** Sort an array of 0s, 1s, and 2s in linear time without extra space.
**Algorithm:** Use three pointers: `low` (0s), `mid` (1s), `high` (2s).
- If `arr[mid] == 0`: Swap `arr[low]` and `arr[mid]`, increment `low` and `mid`.
- If `arr[mid] == 1`: Increment `mid`.
- If `arr[mid] == 2`: Swap `arr[mid]` and `arr[high]`, decrement `high`.

**Example:**
```javascript
function sortColors(nums) {
  let low = 0, mid = 0, high = nums.length - 1;
  while (mid <= high) {
    if (nums[mid] === 0) {
      [nums[low], nums[mid]] = [nums[mid], nums[low]];
      low++; mid++;
    } else if (nums[mid] === 1) {
      mid++;
    } else {
      [nums[mid], nums[high]] = [nums[high], nums[mid]];
      high--;
    }
  }
  return nums;
}
```

### Resizing Boxes (Layout Logic)
**Problem:** 3 boxes. If one resizes, the others should adjust.
**Solution:**
- **CSS (Flexbox):** Use `flex-grow` and `flex-shrink`. If one box's `flex-basis` or width increases, others shrink proportionally.
- **JS:** Listen to resize events (or mouse drag), calculate the delta, and subtract that delta from the other boxes' widths.

**Example (CSS Concept):**
```css
.container { display: flex; }
.box { flex: 1; transition: all 0.3s; }
.box.expanded { flex: 2; } /* Expands, others shrink */
```
