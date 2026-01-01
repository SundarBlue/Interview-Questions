# CSS & Performance Optimization Interview Questions

## 1. CSS Performance Optimization
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

---

## 2. Three Box Resize Problem
**Problem:** You have 3 boxes in a container. When one box is resized, the other 2 boxes should automatically adjust their heights to maintain the total container height.

### Solution 1: Using Flexbox (CSS)

**HTML:**
```html
<div class="container">
  <div class="box box1" data-box="1">
    <div class="resize-handle"></div>
    <div class="content">Box 1</div>
  </div>
  <div class="box box2" data-box="2">
    <div class="resize-handle"></div>
    <div class="content">Box 2</div>
  </div>
  <div class="box box3" data-box="3">
    <div class="content">Box 3</div>
  </div>
</div>
```

**CSS:**
```css
.container {
  display: flex;
  flex-direction: column;
  height: 600px;
  width: 400px;
  border: 2px solid #333;
  overflow: hidden;
}

.box {
  flex: 1; /* All boxes share space equally */
  min-height: 50px;
  position: relative;
  border: 1px solid #ccc;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: flex 0.3s ease;
}

.box1 { background-color: #ffebee; }
.box2 { background-color: #e3f2fd; }
.box3 { background-color: #e8f5e9; }

.resize-handle {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 10px;
  background-color: #666;
  cursor: ns-resize;
  z-index: 10;
}

.resize-handle:hover {
  background-color: #333;
}

/* Dynamic flex values controlled by JavaScript */
.box[style*="flex"] {
  transition: none; /* Disable transition during resize */
}
```

**JavaScript (Interactive Resize):**
```javascript
class BoxResizer {
  constructor() {
    this.container = document.querySelector('.container');
    this.boxes = Array.from(document.querySelectorAll('.box'));
    this.isResizing = false;
    this.currentBox = null;
    this.startY = 0;
    this.startHeights = [];
    
    this.init();
  }
  
  init() {
    const handles = document.querySelectorAll('.resize-handle');
    handles.forEach((handle, index) => {
      handle.addEventListener('mousedown', (e) => this.startResize(e, index));
    });
    
    document.addEventListener('mousemove', (e) => this.resize(e));
    document.addEventListener('mouseup', () => this.stopResize());
  }
  
  startResize(e, boxIndex) {
    e.preventDefault();
    this.isResizing = true;
    this.currentBox = boxIndex;
    this.startY = e.clientY;
    
    // Store current heights
    this.startHeights = this.boxes.map(box => box.offsetHeight);
    this.containerHeight = this.container.offsetHeight;
    
    // Disable transitions during resize
    this.boxes.forEach(box => box.style.transition = 'none');
  }
  
  resize(e) {
    if (!this.isResizing) return;
    
    const deltaY = e.clientY - this.startY;
    const currentBoxIndex = this.currentBox;
    const nextBoxIndex = currentBoxIndex + 1;
    
    // Calculate new heights
    let newCurrentHeight = this.startHeights[currentBoxIndex] + deltaY;
    let newNextHeight = this.startHeights[nextBoxIndex] - deltaY;
    
    // Apply minimum height constraint
    const minHeight = 50;
    newCurrentHeight = Math.max(minHeight, newCurrentHeight);
    newNextHeight = Math.max(minHeight, newNextHeight);
    
    // Calculate flex ratios based on heights
    const totalHeight = this.containerHeight;
    const currentFlex = newCurrentHeight / totalHeight;
    const nextFlex = newNextHeight / totalHeight;
    
    // Calculate the third box flex
    const otherBoxIndex = 3 - currentBoxIndex - nextBoxIndex - 2; // Get the index of third box
    const thirdBoxHeight = this.startHeights.filter((_, i) => 
      i !== currentBoxIndex && i !== nextBoxIndex
    )[0];
    const thirdFlex = thirdBoxHeight / totalHeight;
    
    // Apply new flex values
    this.boxes[currentBoxIndex].style.flex = currentFlex;
    this.boxes[nextBoxIndex].style.flex = nextFlex;
    
    // Update content
    this.boxes[currentBoxIndex].querySelector('.content').textContent = 
      `Box ${currentBoxIndex + 1} (${Math.round(currentFlex * 100)}%)`;
    this.boxes[nextBoxIndex].querySelector('.content').textContent = 
      `Box ${nextBoxIndex + 1} (${Math.round(nextFlex * 100)}%)`;
  }
  
  stopResize() {
    if (!this.isResizing) return;
    this.isResizing = false;
    
    // Re-enable transitions
    setTimeout(() => {
      this.boxes.forEach(box => box.style.transition = '');
    }, 10);
  }
}

// Initialize
document.addEventListener('DOMContentLoaded', () => {
  new BoxResizer();
});
```

### Solution 2: Using CSS Grid

**CSS:**
```css
.container {
  display: grid;
  grid-template-rows: 1fr 1fr 1fr; /* Equal distribution */
  height: 600px;
  width: 400px;
  gap: 2px;
}

.box {
  min-height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Dynamically adjust grid-template-rows via JS */
.container[data-rows] {
  grid-template-rows: attr(data-rows);
}
```

**JavaScript for Grid:**
```javascript
function adjustGrid(box1Height, box2Height, box3Height) {
  const container = document.querySelector('.container');
  const total = box1Height + box2Height + box3Height;
  
  container.style.gridTemplateRows = `${box1Height}fr ${box2Height}fr ${box3Height}fr`;
}

// Example: Make box1 twice as large
adjustGrid(2, 1, 1);
```

### Solution 3: Pure JavaScript Calculation

```javascript
class ThreeBoxResizer {
  constructor(containerSelector) {
    this.container = document.querySelector(containerSelector);
    this.boxes = this.container.querySelectorAll('.box');
    this.containerHeight = this.container.offsetHeight;
  }
  
  // Resize one box, others split the remaining space
  resizeBox(boxIndex, newHeight) {
    const minHeight = 50;
    const totalHeight = this.containerHeight;
    
    // Ensure minimum height
    newHeight = Math.max(minHeight, Math.min(newHeight, totalHeight - 2 * minHeight));
    
    // Calculate remaining height for other two boxes
    const remainingHeight = totalHeight - newHeight;
    const otherBoxHeight = remainingHeight / 2;
    
    // Apply heights
    this.boxes.forEach((box, index) => {
      if (index === boxIndex) {
        box.style.height = `${newHeight}px`;
        box.style.flex = 'none';
      } else {
        box.style.height = `${otherBoxHeight}px`;
        box.style.flex = 'none';
      }
    });
  }
  
  // Resize with custom distribution
  resizeCustom(box1Percent, box2Percent, box3Percent) {
    // Normalize percentages to ensure they sum to 100
    const total = box1Percent + box2Percent + box3Percent;
    const heights = [
      (box1Percent / total) * this.containerHeight,
      (box2Percent / total) * this.containerHeight,
      (box3Percent / total) * this.containerHeight
    ];
    
    this.boxes.forEach((box, index) => {
      box.style.height = `${heights[index]}px`;
      box.style.flex = 'none';
    });
  }
}

// Usage
const resizer = new ThreeBoxResizer('.container');

// Make box 1 take 300px, others split remaining
resizer.resizeBox(0, 300);

// Custom distribution: 50%, 30%, 20%
resizer.resizeCustom(50, 30, 20);
```

### Real-World Example - Dashboard Layout

```html
<!DOCTYPE html>
<html>
<head>
  <style>
    body { margin: 0; font-family: Arial; }
    
    .dashboard {
      display: flex;
      flex-direction: column;
      height: 100vh;
    }
    
    .panel {
      position: relative;
      overflow: auto;
      border: 1px solid #ddd;
    }
    
    .panel-header {
      background: #f5f5f5;
      padding: 10px;
      font-weight: bold;
      cursor: grab;
    }
    
    .panel-header:active {
      cursor: grabbing;
    }
    
    .panel-content {
      padding: 20px;
    }
    
    .divider {
      height: 5px;
      background: #ccc;
      cursor: ns-resize;
      position: relative;
      z-index: 10;
    }
    
    .divider:hover {
      background: #999;
    }
    
    #panel1 { flex: 2; background: #ffebee; }
    #panel2 { flex: 3; background: #e3f2fd; }
    #panel3 { flex: 1; background: #e8f5e9; }
  </style>
</head>
<body>
  <div class="dashboard">
    <div class="panel" id="panel1">
      <div class="panel-header">Panel 1 - Charts</div>
      <div class="panel-content">
        <h3>Sales Chart</h3>
        <p>Chart content here...</p>
      </div>
    </div>
    
    <div class="divider" data-resize="1"></div>
    
    <div class="panel" id="panel2">
      <div class="panel-header">Panel 2 - Data Table</div>
      <div class="panel-content">
        <h3>User Table</h3>
        <p>Table content here...</p>
      </div>
    </div>
    
    <div class="divider" data-resize="2"></div>
    
    <div class="panel" id="panel3">
      <div class="panel-header">Panel 3 - Logs</div>
      <div class="panel-content">
        <h3>System Logs</h3>
        <p>Log entries...</p>
      </div>
    </div>
  </div>
  
  <script>
    class PanelResizer {
      constructor() {
        this.dashboard = document.querySelector('.dashboard');
        this.panels = Array.from(document.querySelectorAll('.panel'));
        this.dividers = Array.from(document.querySelectorAll('.divider'));
        this.isResizing = false;
        this.currentDivider = null;
        
        this.init();
      }
      
      init() {
        this.dividers.forEach((divider, index) => {
          divider.addEventListener('mousedown', (e) => {
            e.preventDefault();
            this.startResize(index, e.clientY);
          });
        });
        
        document.addEventListener('mousemove', (e) => this.onResize(e));
        document.addEventListener('mouseup', () => this.stopResize());
      }
      
      startResize(dividerIndex, startY) {
        this.isResizing = true;
        this.currentDivider = dividerIndex;
        this.startY = startY;
        this.startFlexValues = this.panels.map(p => parseFloat(getComputedStyle(p).flex) || 1);
      }
      
      onResize(e) {
        if (!this.isResizing) return;
        
        const deltaY = e.clientY - this.startY;
        const panelIndex = this.currentDivider;
        const nextPanelIndex = panelIndex + 1;
        
        // Calculate new flex values
        const totalFlex = this.startFlexValues.reduce((a, b) => a + b, 0);
        const flexPerPixel = totalFlex / this.dashboard.offsetHeight;
        const deltaFlex = deltaY * flexPerPixel;
        
        let newFlex = [...this.startFlexValues];
        newFlex[panelIndex] += deltaFlex;
        newFlex[nextPanelIndex] -= deltaFlex;
        
        // Minimum flex value
        const minFlex = 0.5;
        if (newFlex[panelIndex] >= minFlex && newFlex[nextPanelIndex] >= minFlex) {
          this.panels[panelIndex].style.flex = newFlex[panelIndex];
          this.panels[nextPanelIndex].style.flex = newFlex[nextPanelIndex];
        }
      }
      
      stopResize() {
        this.isResizing = false;
      }
    }
    
    new PanelResizer();
  </script>
</body>
</html>
```

### Key Concepts:
- ✅ **Flexbox** - Use `flex` property for proportional sizing
- ✅ **Event Listeners** - Track mouse down, move, up for resizing
- ✅ **Constraints** - Apply minimum/maximum heights
- ✅ **Smooth UX** - Add transitions, visual feedback
- ✅ **Responsive** - Works with any container size

---

## Summary

| Topic | Key Takeaway |
|-------|-------------|
| **CSS Performance** | Minify, critical CSS, avoid expensive properties, use GPU acceleration |
| **Three Box Resize** | Use Flexbox with dynamic flex values, track mouse events for interactive resize |
| **Optimization** | Remove unused CSS, lazy load, use modern features (Grid, aspect-ratio) |
