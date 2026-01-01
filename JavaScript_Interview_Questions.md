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

---

## 10. Node.js: Reading Large Files with Streams
**Concept:** When dealing with large files, loading the entire file into memory can crash your application. Streams allow you to process data in chunks, making it memory-efficient.

### Why Streams?
- **Memory Efficient**: Process data chunk by chunk instead of loading entire file
- **Time Efficient**: Start processing immediately as data arrives
- **Scalable**: Can handle files larger than available RAM

### Types of Streams:
1. **Readable** - Read data (fs.createReadStream)
2. **Writable** - Write data (fs.createWriteStream)
3. **Duplex** - Both read and write
4. **Transform** - Modify data while reading/writing

**Example - Reading Large File Line by Line:**
```javascript
const fs = require('fs');
const readline = require('readline');

// Create read stream
const fileStream = fs.createReadStream('large-file.txt', {
  encoding: 'utf8',
  highWaterMark: 64 * 1024 // 64KB chunks (default 16KB)
});

// Create readline interface
const rl = readline.createInterface({
  input: fileStream,
  crlfDelay: Infinity // Treat \r\n as single line break
});

// Read line by line
let lineNumber = 0;

rl.on('line', (line) => {
  lineNumber++;
  console.log(`Line ${lineNumber}: ${line}`);
  
  // Process each line here
  // Example: Filter, transform, or save to database
});

rl.on('close', () => {
  console.log(`Finished reading ${lineNumber} lines`);
});

rl.on('error', (err) => {
  console.error('Error reading file:', err);
});
```

**Example - With Error Handling and Pause/Resume:**
```javascript
const fs = require('fs');
const readline = require('readline');

function readLargeFile(filePath) {
  return new Promise((resolve, reject) => {
    const fileStream = fs.createReadStream(filePath, {
      encoding: 'utf8'
    });
    
    const rl = readline.createInterface({
      input: fileStream,
      crlfDelay: Infinity
    });
    
    let lineCount = 0;
    let processedCount = 0;
    
    rl.on('line', async (line) => {
      lineCount++;
      
      // Pause stream while processing (for async operations)
      rl.pause();
      
      try {
        // Simulate async processing (e.g., database insert)
        await processLine(line, lineCount);
        processedCount++;
        
        // Resume stream
        rl.resume();
      } catch (err) {
        console.error(`Error processing line ${lineCount}:`, err);
        rl.resume();
      }
    });
    
    rl.on('close', () => {
      console.log(`Total lines: ${lineCount}`);
      console.log(`Successfully processed: ${processedCount}`);
      resolve({ lineCount, processedCount });
    });
    
    rl.on('error', (err) => {
      reject(err);
    });
    
    fileStream.on('error', (err) => {
      reject(err);
    });
  });
}

async function processLine(line, lineNumber) {
  // Your processing logic here
  console.log(`Processing line ${lineNumber}: ${line}`);
  
  // Simulate async work
  return new Promise(resolve => setTimeout(resolve, 10));
}

// Usage
readLargeFile('large-file.txt')
  .then(result => console.log('Success:', result))
  .catch(err => console.error('Failed:', err));
```

**Example - Transform Stream (Uppercase conversion):**
```javascript
const fs = require('fs');
const { Transform } = require('stream');

// Custom transform stream
const upperCaseTransform = new Transform({
  transform(chunk, encoding, callback) {
    // Transform chunk to uppercase
    this.push(chunk.toString().toUpperCase());
    callback();
  }
});

// Read → Transform → Write
fs.createReadStream('input.txt')
  .pipe(upperCaseTransform)
  .pipe(fs.createWriteStream('output.txt'))
  .on('finish', () => console.log('Transformation complete'))
  .on('error', (err) => console.error('Error:', err));
```

**Example - CSV Processing:**
```javascript
const fs = require('fs');
const readline = require('readline');

async function processCSV(filePath) {
  const fileStream = fs.createReadStream(filePath);
  const rl = readline.createInterface({
    input: fileStream,
    crlfDelay: Infinity
  });
  
  let isFirstLine = true;
  let headers = [];
  const data = [];
  
  for await (const line of rl) {
    if (isFirstLine) {
      headers = line.split(',');
      isFirstLine = false;
      continue;
    }
    
    const values = line.split(',');
    const row = {};
    
    headers.forEach((header, index) => {
      row[header.trim()] = values[index]?.trim();
    });
    
    data.push(row);
    
    // Process row immediately instead of storing all in memory
    console.log('Row:', row);
  }
  
  return data;
}

// Usage
processCSV('large-data.csv')
  .then(data => console.log(`Processed ${data.length} rows`))
  .catch(err => console.error('Error:', err));
```

**Stream Benefits:**
- ✅ **Memory Efficient**: Only keeps small chunks in memory
- ✅ **Fast**: Can process files larger than RAM
- ✅ **Real-time**: Start processing immediately
- ✅ **Composable**: Can pipe streams together

**Real-World Use Cases:**
- Processing log files
- ETL operations (Extract, Transform, Load)
- Video/Audio streaming
- Large CSV/JSON file processing
- File uploads/downloads

---

## 11. CSS Performance Optimization
**Concept:** Techniques to make CSS load and render faster, improving overall page performance.

### 1. Minimize and Compress CSS
```css
/* Before - Unminified (10KB) */
.header {
  background-color: #ffffff;
  padding: 20px;
  margin: 0;
}

/* After - Minified (3KB) */
.header{background-color:#fff;padding:20px;margin:0}
```

**Tools:** cssnano, clean-css, webpack css-loader

### 2. Critical CSS (Inline Above-the-Fold CSS)
```html
<!-- Inline critical CSS in <head> for faster first paint -->
<head>
  <style>
    /* Critical CSS - Only above-the-fold styles */
    body { margin: 0; font-family: Arial; }
    .header { background: #333; color: white; padding: 20px; }
    .hero { min-height: 100vh; }
  </style>
  
  <!-- Load rest of CSS asynchronously -->
  <link rel="preload" href="styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
  <noscript><link rel="stylesheet" href="styles.css"></noscript>
</head>
```

### 3. Reduce CSS Selector Complexity
```css
/* ❌ Bad - Too specific, slow */
body div.container ul li a.active span {
  color: red;
}

/* ✅ Good - Simple, fast */
.nav-link-active {
  color: red;
}
```

### 4. Avoid Expensive Properties
```css
/* ❌ Slow - Triggers layout recalculation */
.box {
  width: calc(100% - 20px);
  box-shadow: 0 0 50px rgba(0,0,0,0.5);
  filter: blur(5px);
}

/* ✅ Fast - GPU accelerated */
.box {
  transform: translateX(10px); /* Uses GPU */
  opacity: 0.9;                /* Uses GPU */
}
```

### 5. Use CSS Containment
```css
/* Tell browser this element is independent */
.card {
  contain: layout style paint;
  /* Or use: contain: content; for layout + style + paint */
}

.isolated-widget {
  contain: strict; /* Maximum isolation */
}
```

### 6. Optimize @import (Use <link> instead)
```html
<!-- ❌ Bad - Blocks rendering -->
<style>
  @import url('style1.css');
  @import url('style2.css');
</style>

<!-- ✅ Good - Parallel download -->
<link rel="stylesheet" href="style1.css">
<link rel="stylesheet" href="style2.css">
```

### 7. Remove Unused CSS
```bash
# Use PurgeCSS to remove unused styles
npm install -D @fullhuman/postcss-purgecss

# Result: 100KB → 10KB
```

### 8. Use CSS Variables Efficiently
```css
/* ✅ Good - Define once, reuse everywhere */
:root {
  --primary-color: #007bff;
  --spacing-unit: 8px;
}

.button {
  background: var(--primary-color);
  padding: calc(var(--spacing-unit) * 2);
}
```

### 9. Optimize Animations
```css
/* ❌ Bad - Triggers layout recalculation */
@keyframes slideIn {
  from { margin-left: -100px; }
  to { margin-left: 0; }
}

/* ✅ Good - GPU accelerated */
@keyframes slideIn {
  from { transform: translateX(-100px); }
  to { transform: translateX(0); }
}

.animated {
  will-change: transform; /* Hint to browser */
}
```

### 10. Use Modern CSS Features
```css
/* Use CSS Grid instead of float-based layouts */
.container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}

/* Use aspect-ratio instead of padding hack */
.video-container {
  aspect-ratio: 16 / 9; /* Modern */
  /* Old way: padding-top: 56.25%; */
}
```

### 11. Lazy Load Non-Critical CSS
```html
<!-- Load non-critical CSS after page load -->
<script>
  window.addEventListener('load', function() {
    const link = document.createElement('link');
    link.rel = 'stylesheet';
    link.href = 'non-critical.css';
    document.head.appendChild(link);
  });
</script>
```

### 12. Content Visibility (Modern)
```css
/* Skip rendering off-screen content */
.article-section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px; /* Estimated height */
}
```

### Performance Checklist:
- ✅ Minify CSS (reduce file size by 70%)
- ✅ Inline critical CSS (faster first paint)
- ✅ Remove unused CSS with PurgeCSS
- ✅ Use simple selectors (avoid deep nesting)
- ✅ Prefer transform/opacity for animations
- ✅ Use CSS containment for independent components
- ✅ Load non-critical CSS asynchronously
- ✅ Enable gzip/brotli compression on server
- ✅ Use CDN for CSS files
- ✅ Avoid @import, use <link> instead

**Measuring Performance:**
```javascript
// Measure CSS load time
performance.getEntriesByType('resource')
  .filter(r => r.name.endsWith('.css'))
  .forEach(css => {
    console.log(`${css.name}: ${css.duration}ms`);
  });
```
