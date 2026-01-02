# Angular & RxJS Scenario-Based Interview Questions

This file contains the most scenario-focused Angular interview questions that present concrete problems and solutions.

**Contents:**
- Section 1: Change Detection - Rapid Button Click Scenario
- Section 7: Nested API Calls - API Inside API Problem
- Section 9: Handling Large Lists - 100k Items Performance
- Section 11: Multiple API Failures - 10 APIs Failing
- Section 13: Highlight Matching Words - Reactive Forms + Directive

---
## 1. Change Detection Strategies
**Concept:** How Angular checks for changes to update the DOM.

### Scenario: 10 Consecutive Rapid Button Clicks

**What happens when a button is clicked 10 times rapidly?**

| Click # | **Default (CheckAlways)** | **OnPush** |
|---------|---------------------------|------------|
| **Click 1** | ✅ CD runs on entire component tree from root → checks ALL components | ✅ CD runs on this component + ancestors → checks only this component and parent chain |
| **Click 2** | ✅ CD runs again on entire tree → checks ALL components | ✅ CD runs again on this component + ancestors → checks only this component and parent chain |
| **Click 3** | ✅ CD runs again on entire tree → checks ALL components | ✅ CD runs again on this component + ancestors → checks only this component and parent chain |
| **Click 4** | ✅ CD runs again on entire tree → checks ALL components | ✅ CD runs again on this component + ancestors → checks only this component and parent chain |
| **Click 5** | ✅ CD runs again on entire tree → checks ALL components | ✅ CD runs again on this component + ancestors → checks only this component and parent chain |
| **Click 6** | ✅ CD runs again on entire tree → checks ALL components | ✅ CD runs again on this component + ancestors → checks only this component and parent chain |
| **Click 7** | ✅ CD runs again on entire tree → checks ALL components | ✅ CD runs again on this component + ancestors → checks only this component and parent chain |
| **Click 8** | ✅ CD runs again on entire tree → checks ALL components | ✅ CD runs again on this component + ancestors → checks only this component and parent chain |
| **Click 9** | ✅ CD runs again on entire tree → checks ALL components | ✅ CD runs again on this component + ancestors → checks only this component and parent chain |
| **Click 10** | ✅ CD runs again on entire tree → checks ALL components | ✅ CD runs again on this component + ancestors → checks only this component and parent chain |
| **Total CD Cycles** | **10 full tree checks** | **10 component checks (narrower scope)** |
| **Performance Impact** | 🔴 **High** - Checks 1000s of components if tree is large | 🟡 **Medium** - Only checks this component + ancestors |

### Key Takeaway:
- **Both strategies run CD on every single click** because clicks are events originating from the component
- **Difference**: Default checks the **entire app**, OnPush checks **only the affected component path**
- **OnPush is faster**, but doesn't prevent multiple CD cycles from rapid clicks

### What does "Component + Ancestors" mean?

**Simple Explanation:** Think of your Angular app as a **family tree**. Ancestors are the parent, grandparent, great-grandparent components above you.

**Example Component Tree:**
```
AppComponent (Root - great-grandparent)
  ├── HeaderComponent (grandparent)
  ├── DashboardComponent (parent)
  │     ├── UserListComponent (YOU - where button was clicked) ⬅️ Click happens here
  │     ├── ProductListComponent (sibling)
  │     └── ChartComponent (sibling)
  └── FooterComponent (uncle)
```

**When you click a button in `UserListComponent`:**

| Strategy | What Gets Checked |
|----------|-------------------|
| **Default** | ✅ AppComponent<br>✅ HeaderComponent<br>✅ DashboardComponent<br>✅ **UserListComponent** (clicked)<br>✅ ProductListComponent<br>✅ ChartComponent<br>✅ FooterComponent<br><br>**Result: ALL 7 components checked** |
| **OnPush** | ✅ AppComponent (ancestor)<br>❌ HeaderComponent (not in path)<br>✅ DashboardComponent (ancestor - parent)<br>✅ **UserListComponent** (clicked)<br>❌ ProductListComponent (sibling - not checked)<br>❌ ChartComponent (sibling - not checked)<br>❌ FooterComponent (not in path)<br><br>**Result: Only 3 components checked (path from root to clicked component)** |

**In Simple Terms:**
- **Ancestors** = Your parents going up to the root (AppComponent)
- **OnPush checks only the path** from the clicked component up to the root
- **Siblings and unrelated components are skipped** ✅ This is why OnPush is faster!

**To optimize rapid clicks, use these techniques:**

```typescript
// Option 1: Debounce using RxJS
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<button (click)="onButtonClick()">Click Me</button>`
})
export class MyComponent {
  private clickSubject = new Subject<void>();
  
  constructor() {
    this.clickSubject.pipe(
      debounceTime(300), // Wait 300ms after last click
      takeUntilDestroyed()
    ).subscribe(() => {
      this.handleClick();
    });
  }
  
  onButtonClick() {
    this.clickSubject.next();
  }
  
  handleClick() {
    console.log('Processing click...');
  }
}

// Option 2: Disable button during processing
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<button [disabled]="isProcessing" (click)="onClick()">Save</button>`
})
export class SaveComponent {
  isProcessing = false;
  
  onClick() {
    if (this.isProcessing) return;
    
    this.isProcessing = true;
    this.save().subscribe(() => {
      this.isProcessing = false;
    });
  }
}

// Option 3: Detach change detector for manual control
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<button (click)="onClick()">Click</button>`
})
export class ManualCDComponent {
  constructor(private cdr: ChangeDetectorRef) {
    this.cdr.detach(); // Detach automatic CD
  }
  
  onClick() {
    this.processClick();
    this.cdr.detectChanges(); // Manually trigger CD when needed
  }
}
```

**Example:**
```typescript
@Component({
  selector: 'app-item',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `...`
})
export class ItemComponent {}
```
### ChangeDetectorRef: `markForCheck` vs `detectChanges`
**Concept:** When using `OnPush` strategy or detaching from the change detection tree, you need to manually trigger updates using `ChangeDetectorRef`.

- **`markForCheck()`**: Marks the component and all its ancestors as "dirty". Angular will check them during the *next* change detection cycle. It does not run change detection immediately.
- **`detectChanges()`**: Triggers change detection *immediately* for this component and its children.

**Example:**
```typescript
@Component({
  selector: 'app-timer',
  template: `Count: {{ count }}`,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class TimerComponent implements OnInit {
  count = 0;

  constructor(private cdr: ChangeDetectorRef) {}

  ngOnInit() {
    // setInterval runs outside Angular's zone awareness for OnPush in some cases, 
    // or if data comes from a source that doesn't trigger CD automatically.
    setInterval(() => {
      this.count++;
      // Tells Angular: "Check me in the next cycle"
      this.cdr.markForCheck(); 
    }, 1000);
  }

  forceUpdate() {
    this.count = 100;
    // Forces an immediate check of this component and its children
    this.cdr.detectChanges();
  }
}
```


---

## 7. Nested API Calls (API Inside API - What Will You Do?)
**Problem:** When one API call depends on the result of another (nested dependencies), using nested `.subscribe()` creates "callback hell" and makes code hard to maintain.

**❌ Bad Practice (Nested Subscriptions - Callback Hell):**
```typescript
// DON'T DO THIS!
getUserOrders(userId: number) {
  this.userService.getUser(userId).subscribe(user => {
    this.orderService.getOrders(user.id).subscribe(orders => {
      this.productService.getProducts(orders[0].productId).subscribe(product => {
        this.reviewService.getReviews(product.id).subscribe(reviews => {
          // 4 levels deep! Hard to read and maintain
          console.log('Reviews:', reviews);
        });
      });
    });
  });
}
```

**✅ Good Practice (Flattening Operators):**

Use RxJS **flattening operators** to chain API calls cleanly:
- **`switchMap`**: Cancels previous inner observable (best for user-triggered actions like search, autocomplete)
- **`mergeMap`**: Runs all in parallel (good for independent operations)
- **`concatMap`**: Runs in sequence (good for order-sensitive operations)
- **`exhaustMap`**: Ignores new requests while previous is running (good for preventing duplicate submissions)

### Example 1: Sequential API Calls with `switchMap`

```typescript
@Component({...})
export class UserOrdersComponent {
  
  getUserOrderDetails(userId: number) {
    this.userService.getUser(userId).pipe(
      // Step 1: Get user
      switchMap(user => 
        this.orderService.getOrders(user.id).pipe(
          // Pass user data along with orders
          map(orders => ({ user, orders }))
        )
      ),
      // Step 2: Get product details for first order
      switchMap(({ user, orders }) => 
        this.productService.getProduct(orders[0].productId).pipe(
          map(product => ({ user, orders, product }))
        )
      ),
      // Step 3: Get reviews for the product
      switchMap(({ user, orders, product }) => 
        this.reviewService.getReviews(product.id).pipe(
          map(reviews => ({ user, orders, product, reviews }))
        )
      )
    ).subscribe(result => {
      // Clean, flat code - all data available here
      console.log('User:', result.user);
      console.log('Orders:', result.orders);
      console.log('Product:', result.product);
      console.log('Reviews:', result.reviews);
    });
  }
}
```

### Example 2: User ID from Route → Fetch User → Fetch Posts

```typescript
@Component({
  template: `
    <div *ngFor="let post of posts">{{ post.title }}</div>
  `
})
export class UserPostsComponent implements OnInit {
  posts: Post[] = [];
  
  constructor(
    private route: ActivatedRoute,
    private userService: UserService,
    private postService: PostService
  ) {}
  
  ngOnInit() {
    this.route.params.pipe(
      // Step 1: Get userId from route params
      switchMap(params => 
        this.userService.getUser(params['id'])
      ),
      // Step 2: Use user.id to fetch posts
      switchMap(user => 
        this.postService.getUserPosts(user.id)
      )
    ).subscribe(posts => {
      this.posts = posts;
    });
  }
}
```

### Example 3: Order → Customer → Address (Real-world scenario)

```typescript
@Component({...})
export class OrderDetailsComponent {
  
  loadOrderDetails(orderId: number) {
    this.orderService.getOrder(orderId).pipe(
      // Get order details
      switchMap(order => 
        this.customerService.getCustomer(order.customerId).pipe(
          map(customer => ({ order, customer }))
        )
      ),
      // Get customer's shipping address
      switchMap(({ order, customer }) => 
        this.addressService.getAddress(customer.addressId).pipe(
          map(address => ({ order, customer, address }))
        )
      )
    ).subscribe(({ order, customer, address }) => {
      console.log('Order:', order);
      console.log('Customer:', customer.name);
      console.log('Ship to:', address.street);
    });
  }
}
```

### When to Use Which Operator?

| Operator | Behavior | Use Case | Example |
|----------|----------|----------|---------|
| **`switchMap`** | Cancels previous inner subscription when new value arrives | User typing in search box, route changes | Autocomplete search |
| **`mergeMap`** | Keeps all inner subscriptions active (parallel) | Independent operations that can run simultaneously | Saving multiple form sections |
| **`concatMap`** | Waits for previous inner subscription to complete (sequential) | Order matters (e.g., upload file, then save metadata) | File upload → Database save |
| **`exhaustMap`** | Ignores new values while inner subscription is active | Prevent duplicate requests | Login button (prevent double-click) |

### Example 4: Search Autocomplete (why `switchMap` is perfect)

```typescript
@Component({
  template: `
    <input [formControl]="searchControl" placeholder="Search users...">
    <div *ngFor="let user of searchResults">{{ user.name }}</div>
  `
})
export class SearchComponent {
  searchControl = new FormControl('');
  searchResults: User[] = [];
  
  ngOnInit() {
    this.searchControl.valueChanges.pipe(
      debounceTime(300),  // Wait for user to stop typing
      distinctUntilChanged(),  // Only if value actually changed
      switchMap(query => 
        this.userService.searchUsers(query)
      )
      // switchMap cancels previous search if user types again
    ).subscribe(users => {
      this.searchResults = users;
    });
  }
}
```

**Why `switchMap` here?**
- User types "Jo" → API call starts
- User types "John" → Previous "Jo" call is CANCELLED
- Only "John" results are shown (no race conditions)

### Example 5: Combining Multiple Dependent Calls with Error Handling

```typescript
getUserDashboard(userId: number) {
  this.userService.getUser(userId).pipe(
    switchMap(user => 
      forkJoin({
        orders: this.orderService.getOrders(user.id),
        favorites: this.favoriteService.getFavorites(user.id),
        recommendations: this.recommendationService.get(user.id)
      }).pipe(
        map(data => ({ user, ...data }))
      )
    ),
    catchError(error => {
      console.error('Failed to load dashboard:', error);
      return of(null);
    })
  ).subscribe(dashboard => {
    if (dashboard) {
      console.log('Dashboard loaded:', dashboard);
    }
  });
}
```

### Summary: Nested API Calls Best Practices

| ❌ Don't Do This | ✅ Do This Instead |
|-----------------|-------------------|
| Nested `.subscribe()` | Use `switchMap`, `mergeMap`, `concatMap` |
| `subscribe()` inside `subscribe()` | Chain operators with `pipe()` |
| Callback hell | Flat, readable operator chain |
| No error handling | Add `catchError` in the chain |
| Manual unsubscription tracking | Use `takeUntilDestroyed()` or `async` pipe |

**Key Takeaway:** Always use flattening operators (`switchMap`, `mergeMap`, `concatMap`, `exhaustMap`) for dependent API calls. Never nest `.subscribe()` calls!


---

## 9. Handling Large Lists - Multiple Strategies

**Problem:** Rendering 100,000 items (1 lakh) directly in the DOM will freeze or crash the browser.

### Strategy Comparison

| Strategy | Best For | Pros | Cons |
|----------|----------|------|------|
| **Virtual Scrolling** | Continuous scrolling, all data in memory | Fast, smooth scrolling | Requires all data upfront, memory intensive |
| **Pagination** | Tables, search results | Simple, low memory | Poor UX (click to load more) |
| **Infinite Scroll** | Social feeds, news | Great UX, loads on demand | Complex state management |
| **Server-side Filtering** | Very large datasets (millions) | Minimal data transfer | Backend dependency |
| **Windowing (manual)** | Custom scroll behavior | Full control | Complex implementation |

---

### Option 1: Virtual Scrolling (CDK - Recommended for large in-memory lists)

**When to use:** You have all data in memory and want smooth scrolling.

**How it works:** Only renders visible items in viewport, reuses DOM nodes as you scroll.

```typescript
import { CdkVirtualScrollViewport, ScrollingModule } from '@angular/cdk/scrolling';

@Component({
  selector: 'app-list',
  standalone: true,
  imports: [ScrollingModule, CommonModule],
  template: `
    <cdk-virtual-scroll-viewport itemSize="50" class="viewport">
      <div *cdkVirtualFor="let item of items" class="item">
        {{ item.name }}
      </div>
    </cdk-virtual-scroll-viewport>
  `,
  styles: [`
    .viewport {
      height: 500px;
      width: 100%;
      border: 1px solid #ccc;
    }
    .item {
      height: 50px;
      padding: 10px;
    }
  `]
})
export class ListComponent {
  items = Array.from({ length: 100000 }, (_, i) => ({
    id: i,
    name: `Item ${i}`
  }));
}
```

**Performance:**
- ✅ Renders only ~20 items at a time (what fits in viewport)
- ✅ Handles 100k+ items smoothly
- ⚠️ All data must be in memory

---

### Option 2: Pagination (Best for tables/search results)

**When to use:** Traditional table views, search results, admin panels.

**How it works:** Load data in chunks (pages), user clicks to navigate.

```typescript
@Component({
  selector: 'app-paginated-list',
  template: `
    <div *ngFor="let item of paginatedItems">{{ item.name }}</div>
    
    <div class="pagination">
      <button (click)="previousPage()" [disabled]="currentPage === 1">Previous</button>
      <span>Page {{ currentPage }} of {{ totalPages }}</span>
      <button (click)="nextPage()" [disabled]="currentPage === totalPages">Next</button>
    </div>
  `
})
export class PaginatedListComponent {
  allItems: any[] = []; // 100k items
  paginatedItems: any[] = [];
  currentPage = 1;
  pageSize = 50;
  
  get totalPages() {
    return Math.ceil(this.allItems.length / this.pageSize);
  }
  
  ngOnInit() {
    this.loadPage(1);
  }
  
  loadPage(page: number) {
    const startIndex = (page - 1) * this.pageSize;
    const endIndex = startIndex + this.pageSize;
    this.paginatedItems = this.allItems.slice(startIndex, endIndex);
    this.currentPage = page;
  }
  
  nextPage() {
    if (this.currentPage < this.totalPages) {
      this.loadPage(this.currentPage + 1);
    }
  }
  
  previousPage() {
    if (this.currentPage > 1) {
      this.loadPage(this.currentPage - 1);
    }
  }
}
```

**Performance:**
- ✅ Only renders 50 items per page
- ✅ Low memory footprint
- ❌ Poor UX (manual navigation)

---

### Option 3: Infinite Scroll (Best for social feeds)

**When to use:** Social media feeds, news articles, product listings.

**How it works:** Load more data automatically when user scrolls near bottom.

```typescript
import { fromEvent } from 'rxjs';
import { debounceTime, filter } from 'rxjs/operators';

@Component({
  selector: 'app-infinite-scroll',
  template: `
    <div class="list" #scrollContainer>
      <div *ngFor="let item of displayedItems" class="item">
        {{ item.name }}
      </div>
      <div *ngIf="loading" class="loader">Loading...</div>
    </div>
  `,
  styles: [`
    .list {
      height: 500px;
      overflow-y: auto;
    }
  `]
})
export class InfiniteScrollComponent implements OnInit {
  @ViewChild('scrollContainer') scrollContainer!: ElementRef;
  
  allItems: any[] = Array.from({ length: 100000 }, (_, i) => ({ id: i, name: `Item ${i}` }));
  displayedItems: any[] = [];
  currentIndex = 0;
  pageSize = 50;
  loading = false;
  
  ngOnInit() {
    this.loadMore();
  }
  
  ngAfterViewInit() {
    // Listen to scroll events
    fromEvent(this.scrollContainer.nativeElement, 'scroll')
      .pipe(
        debounceTime(200),
        filter(() => this.isNearBottom())
      )
      .subscribe(() => {
        if (!this.loading && this.currentIndex < this.allItems.length) {
          this.loadMore();
        }
      });
  }
  
  isNearBottom(): boolean {
    const element = this.scrollContainer.nativeElement;
    const threshold = 100; // Load when 100px from bottom
    return element.scrollHeight - element.scrollTop - element.clientHeight < threshold;
  }
  
  loadMore() {
    this.loading = true;
    
    // Simulate async loading
    setTimeout(() => {
      const nextItems = this.allItems.slice(
        this.currentIndex,
        this.currentIndex + this.pageSize
      );
      this.displayedItems.push(...nextItems);
      this.currentIndex += this.pageSize;
      this.loading = false;
    }, 500);
  }
}
```

**Performance:**
- ✅ Loads data progressively
- ✅ Great UX (automatic loading)
- ⚠️ Memory grows as user scrolls (all loaded items stay in DOM)

---

### Option 4: Server-Side Filtering/Pagination (Best for millions of records)

**When to use:** Dataset is too large to load in memory (millions of rows).

**How it works:** Backend handles filtering, sorting, pagination. Frontend only receives current page.

```typescript
@Component({
  selector: 'app-server-pagination',
  template: `
    <input [(ngModel)]="searchQuery" (input)="search()" placeholder="Search...">
    
    <table>
      <tr *ngFor="let item of items">
        <td>{{ item.name }}</td>
      </tr>
    </table>
    
    <button (click)="loadPage(currentPage - 1)" [disabled]="currentPage === 1">Previous</button>
    <span>{{ currentPage }} / {{ totalPages }}</span>
    <button (click)="loadPage(currentPage + 1)" [disabled]="currentPage === totalPages">Next</button>
  `
})
export class ServerPaginationComponent {
  items: any[] = [];
  currentPage = 1;
  totalPages = 0;
  pageSize = 50;
  searchQuery = '';
  
  constructor(private http: HttpClient) {}
  
  ngOnInit() {
    this.loadPage(1);
  }
  
  loadPage(page: number) {
    const params = {
      page: page.toString(),
      pageSize: this.pageSize.toString(),
      search: this.searchQuery
    };
    
    this.http.get<any>('/api/items', { params }).subscribe(response => {
      this.items = response.items;
      this.currentPage = response.currentPage;
      this.totalPages = response.totalPages;
    });
  }
  
  search() {
    this.loadPage(1); // Reset to first page on search
  }
}
```

**Backend API Response:**
```json
{
  "items": [...], // Only 50 items
  "currentPage": 1,
  "totalPages": 2000,
  "totalItems": 100000
}
```

**Performance:**
- ✅ Minimal frontend memory usage
- ✅ Handles millions of records
- ✅ Fast search/filtering on backend
- ⚠️ Requires backend support

---

### Which Strategy to Choose?

| Scenario | Best Strategy |
|----------|---------------|
| **100k items, all in memory, need smooth scroll** | Virtual Scrolling (CDK) ✅ |
| **Admin table, need sorting/filtering** | Pagination ✅ |
| **Social feed, news list** | Infinite Scroll ✅ |
| **Millions of records, can't load all** | Server-Side Pagination ✅ |
| **Real-time data updates** | Virtual Scrolling + WebSocket |
| **Mobile app with limited memory** | Server-Side Pagination ✅ |

---

### Performance Tips

1. **TrackBy function** (for all strategies):
```typescript
<div *ngFor="let item of items; trackBy: trackById">
  {{ item.name }}
</div>

trackById(index: number, item: any) {
  return item.id; // Angular reuses DOM nodes with same ID
}
```

2. **OnPush change detection:**
```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
```

3. **Debounce search input:**
```typescript
this.searchControl.valueChanges
  .pipe(debounceTime(300))
  .subscribe(query => this.search(query));
```

### Summary

- **Virtual Scrolling:** Best all-around for large in-memory lists ✅
- **Pagination:** Simple but poor UX
- **Infinite Scroll:** Great UX but memory grows
- **Server-Side:** Only option for truly massive datasets (millions)


---

## 11. Handling Multiple API Failures (If 10 APIs Fail, How Do You Handle It?)
**Concept:**
When calling multiple APIs simultaneously (e.g., 10 different endpoints), you need a strategy to:
1. **Prevent one failure from blocking all others** (graceful degradation)
2. **Retry failed requests** (with exponential backoff)
3. **Provide fallback data** or error messages
4. **Track which APIs succeeded/failed**

### Strategy 1: Resilient `forkJoin` with `catchError`
**Problem with default `forkJoin`:** If ONE API fails, the entire stream fails.

**Solution:** Wrap each API call with `catchError` to return a fallback value.

```typescript
import { forkJoin, of } from 'rxjs';
import { catchError, map } from 'rxjs/operators';

@Component({...})
export class DashboardComponent implements OnInit {
  
  ngOnInit() {
    // Calling 10 different APIs
    forkJoin({
      users: this.api.getUsers().pipe(
        catchError(err => {
          console.error('Users API failed:', err);
          return of([]); // Return empty array as fallback
        })
      ),
      products: this.api.getProducts().pipe(
        catchError(err => of([]))
      ),
      orders: this.api.getOrders().pipe(
        catchError(err => of([]))
      ),
      analytics: this.api.getAnalytics().pipe(
        catchError(err => of({ views: 0, clicks: 0 }))
      ),
      notifications: this.api.getNotifications().pipe(
        catchError(err => of([]))
      ),
      settings: this.api.getSettings().pipe(
        catchError(err => of(null))
      ),
      profile: this.api.getProfile().pipe(
        catchError(err => of(null))
      ),
      messages: this.api.getMessages().pipe(
        catchError(err => of([]))
      ),
      tasks: this.api.getTasks().pipe(
        catchError(err => of([]))
      ),
      reports: this.api.getReports().pipe(
        catchError(err => of([]))
      )
    }).subscribe(results => {
      // All APIs completed (some may have fallback values)
      console.log('Dashboard data loaded:', results);
      
      // Check which APIs failed
      if (!results.users.length) {
        this.showError('Failed to load users');
      }
      if (!results.settings) {
        this.showError('Failed to load settings');
      }
      
      // Continue with available data
      this.renderDashboard(results);
    });
  }
}
```

### Strategy 2: Retry Failed Requests with `retry` and `retryWhen`

```typescript
import { retry, retryWhen, delay, take } from 'rxjs/operators';

// Simple retry (3 attempts)
this.api.getUsers().pipe(
  retry(3), // Retry up to 3 times on failure
  catchError(err => of([]))
).subscribe(users => {
  console.log('Users:', users);
});

// Exponential backoff retry
this.api.getProducts().pipe(
  retryWhen(errors => 
    errors.pipe(
      delay(1000), // Wait 1 second between retries
      take(3)      // Retry max 3 times
    )
  ),
  catchError(err => of([]))
).subscribe(products => {
  console.log('Products:', products);
});
```

### Strategy 3: Track Success/Failure Status

```typescript
interface ApiResult<T> {
  data: T | null;
  success: boolean;
  error?: any;
}

function safeApiCall<T>(observable: Observable<T>): Observable<ApiResult<T>> {
  return observable.pipe(
    map(data => ({ data, success: true })),
    catchError(error => of({ data: null, success: false, error }))
  );
}

// Usage
forkJoin({
  users: safeApiCall(this.api.getUsers()),
  products: safeApiCall(this.api.getProducts()),
  orders: safeApiCall(this.api.getOrders()),
  // ... 7 more APIs
}).subscribe(results => {
  const failedApis = Object.entries(results)
    .filter(([key, result]) => !result.success)
    .map(([key]) => key);
  
  console.log(`${failedApis.length} APIs failed:`, failedApis);
  
  // Show user-friendly message
  if (failedApis.length > 5) {
    this.showError('Multiple services are unavailable. Please try again later.');
  }
  
  // Use successful data
  if (results.users.success) {
    this.displayUsers(results.users.data);
  }
});
```

### Strategy 4: Show Loading State Per API

```typescript
@Component({
  template: `
    <div *ngIf="loading.users">Loading users...</div>
    <div *ngIf="errors.users" class="error">Failed to load users</div>
    <div *ngIf="data.users">{{ data.users.length }} users loaded</div>
  `
})
export class DashboardComponent {
  loading = { users: true, products: true, orders: true };
  errors = { users: false, products: false, orders: false };
  data: any = {};
  
  ngOnInit() {
    // Load each API independently
    this.loadUsers();
    this.loadProducts();
    this.loadOrders();
    // ... load all 10 APIs
  }
  
  loadUsers() {
    this.api.getUsers().pipe(
      retry(2),
      catchError(err => {
        this.errors.users = true;
        return of([]);
      })
    ).subscribe(users => {
      this.loading.users = false;
      this.data.users = users;
    });
  }
}
```

### Summary Table:

| Approach | Pros | Cons | Use Case |
|----------|------|------|----------|
| **Resilient forkJoin** | All APIs run in parallel, one failure doesn't block others | More boilerplate code | Dashboard with multiple independent data sources |
| **Retry with backoff** | Handles transient failures (network hiccups) | Delays response time | APIs with intermittent connectivity issues |
| **Status tracking** | Detailed insight into which APIs failed | More complex state management | Critical monitoring dashboards |
| **Independent loading** | Fine-grained control, better UX | More code, harder to coordinate | User-facing dashboards with partial data |

**Real-World Example:**
```typescript
// E-commerce dashboard loading 10 different data points
forkJoin({
  recentOrders: this.orderService.getRecent().pipe(catchError(() => of([]))),
  topProducts: this.productService.getTopSellers().pipe(retry(2), catchError(() => of([]))),
  revenue: this.analyticsService.getRevenue().pipe(catchError(() => of(0))),
  inventory: this.inventoryService.getStock().pipe(catchError(() => of({}))),
  customers: this.customerService.getActive().pipe(catchError(() => of([]))),
  reviews: this.reviewService.getRecent().pipe(catchError(() => of([]))),
  refunds: this.refundService.getPending().pipe(catchError(() => of([]))),
  shipping: this.shippingService.getTracking().pipe(catchError(() => of([]))),
  notifications: this.notificationService.getUnread().pipe(catchError(() => of([]))),
  settings: this.settingsService.get().pipe(catchError(() => of(null)))
}).subscribe(dashboard => {
  // Even if 5 APIs fail, we still render the dashboard with available data
  this.renderDashboard(dashboard);
});


---

## 13. Scenario: Highlight Matching Words (Reactive Forms + Directive)
**Concept Overview:**
- **FormControl**: Captures user input reactively.
- **Component Logic**: Watches the FormControl value. Passes the typed word to the template.
- **Custom Directive**: Highlights matching words inside a paragraph dynamically.

**Example:**

**1. Component (Logic):**
```typescript
@Component({
  selector: 'app-highlighter',
  template: `
    <input [formControl]="searchControl" placeholder="Type to highlight...">
    <p [appHighlight]="searchControl.value">
      Angular is a platform for building mobile and desktop web applications.
    </p>
  `
})
export class HighlighterComponent {
  searchControl = new FormControl('');
}
```

**2. Custom Directive (Implementation):**
```typescript
@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective implements OnChanges {
  @Input('appHighlight') searchedWord: string | null = '';

  constructor(private el: ElementRef, private renderer: Renderer2) {}

  ngOnChanges() {
    // Store original text if not already stored (simplified for demo)
    // In real app, better to keep original text in a separate property
    
    if (!this.searchedWord) {
       // Logic to reset text would go here
       return;
    }
    
    const text = this.el.nativeElement.innerText; 
    const regex = new RegExp(`(${this.searchedWord})`, 'gi');
    const newText = text.replace(regex, '<span style="background-color: yellow;">$1</span>');
    
    this.renderer.setProperty(this.el.nativeElement, 'innerHTML', newText);
  }
}
```

---


---


