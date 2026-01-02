# Angular & RxJS Interview Questions
## Medium Level Questions (Sections 8-14)

---

## Additional Medium-Level Questions

### Q: Difference between pure and impure pipes?

| Feature | Pure Pipe | Impure Pipe |
|---------|-----------|-------------|
| **Change Detection** | Only when input value/reference changes | On every change detection cycle |
| **Performance** | Fast (cached) | Slower (recalculates often) |
| **Default** | Yes (`pure: true`) | No (`pure: false`) |
| **Use Case** | Simple transformations | Async data, complex logic |

**Example:**
```typescript
// Pure pipe (default)
@Pipe({ name: 'capitalize', pure: true })
export class CapitalizePipe implements PipeTransform {
  transform(value: string): string {
    return value.toUpperCase();
  }
}

// Impure pipe (runs on every change detection)
@Pipe({ name: 'asyncFilter', pure: false })
export class AsyncFilterPipe implements PipeTransform {
  transform(items: any[], searchTerm: string): any[] {
    return items.filter(item => item.name.includes(searchTerm));
  }
}
```

**When to use impure pipes:**
- Working with mutable data (arrays that change)
- Async operations
- When you need immediate updates

**⚠️ Warning:** Impure pipes hurt performance - use sparingly!

### Q: Is calling a method in template interpolation good? How to optimize?
**Answer:** **No**, calling methods in templates is bad practice because:
- Method is called on **every change detection cycle**
- Can cause performance issues
- Hard to optimize

**❌ Bad:**
```typescript
@Component({
  template: `<p>Total: {{ calculateTotal() }}</p>`
})
export class BadComponent {
  calculateTotal() {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
}
```

**✅ Good - Use computed property:**
```typescript
@Component({
  template: `<p>Total: {{ total }}</p>`
})
export class GoodComponent {
  total = 0;
  
  ngOnChanges() {
    this.total = this.items.reduce((sum, item) => sum + item.price, 0);
  }
}
```

**✅ Better - Use pure pipe:**
```typescript
@Pipe({ name: 'sum', pure: true })
export class SumPipe implements PipeTransform {
  transform(items: any[]): number {
    return items.reduce((sum, item) => sum + item.price, 0);
  }
}

// Template: <p>Total: {{ items | sum }}</p>
```

**✅ Best - Use memoization (Angular 16+):**
```typescript
import { computed, signal } from '@angular/core';

@Component({...})
export class OptimizedComponent {
  items = signal<Item[]>([]);
  
  // Computed value (cached, only recalculates when items change)
  total = computed(() => 
    this.items().reduce((sum, item) => sum + item.price, 0)
  );
}
```

### Q: Define Services?
**Answer:** Services are TypeScript classes that provide specific functionality across components. They follow the **Singleton pattern** (one instance shared across app).

**Common use cases:**
- Data fetching (HTTP calls)
- Business logic
- State management
- Utility functions

**Example:**
```typescript
@Injectable({
  providedIn: 'root' // Singleton service
})
export class UserService {
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users');
  }
  
  getUserById(id: number): Observable<User> {
    return this.http.get<User>(`/api/users/${id}`);
  }
}
```

**Benefits:**
- **Reusability**: Share logic across components
- **Separation of Concerns**: Components focus on UI
- **Testability**: Easy to mock services
- **Dependency Injection**: Angular manages lifecycle

### Q: What is RxJS and its relation to Angular?
**Answer:** **RxJS (Reactive Extensions for JavaScript)** is a library for reactive programming using Observables.

**Angular uses RxJS for:**
- **HTTP calls**: `HttpClient` returns Observables
- **Forms**: `valueChanges` is an Observable
- **Router**: Route events are Observables
- **Event handling**: Custom event streams

**Core concepts:**
```typescript
// Observable: Stream of data over time
const data$ = this.http.get('/api/data');

// Operators: Transform data
data$.pipe(
  map(data => data.items),
  filter(items => items.length > 0),
  catchError(err => of([]))
).subscribe(items => console.log(items));
```

**Why RxJS in Angular:**
- Handles async operations elegantly
- Composable (chain operations)
- Cancellable (unsubscribe)
- Rich operator library

### Q: Main RxJS parts?

**1. Observable** - Stream that emits values over time
```typescript
const observable$ = new Observable(observer => {
  observer.next(1);
  observer.next(2);
  observer.complete();
});
```

**2. Subject** - Observable + Observer (can emit and subscribe)
```typescript
const subject = new Subject<number>();
subject.subscribe(v => console.log('A:', v));
subject.next(1); // A: 1
```

**3. BehaviorSubject** - Stores current value, emits to new subscribers
```typescript
const behavior = new BehaviorSubject<number>(0);
behavior.subscribe(v => console.log('A:', v)); // A: 0 (immediate)
behavior.next(1); // A: 1
behavior.subscribe(v => console.log('B:', v)); // B: 1 (gets last value)
```

**4. ReplaySubject** - Buffers N values, replays to new subscribers
```typescript
const replay = new ReplaySubject<number>(2); // Buffer 2 values
replay.next(1);
replay.next(2);
replay.next(3);
replay.subscribe(v => console.log(v)); // 2, 3 (last 2 values)
```

### Q: Hot vs cold observables? Multiple subscriptions to cold?
**Answer:**

**Cold Observable** (Unicast):
- Starts emitting only when subscribed
- Each subscriber gets its own execution
- Example: HTTP calls, `of()`, `from()`

```typescript
const cold$ = this.http.get('/api/data');
cold$.subscribe(data => console.log('A:', data)); // Makes HTTP call
cold$.subscribe(data => console.log('B:', data)); // Makes ANOTHER HTTP call
```

**Hot Observable** (Multicast):
- Already emitting, subscribers join midstream
- Shared execution for all subscribers
- Example: DOM events, Subjects

```typescript
const hot$ = fromEvent(document, 'click');
hot$.subscribe(e => console.log('A:', e)); // Shares clicks
hot$.subscribe(e => console.log('B:', e)); // Same clicks
```

**Converting cold to hot:**
```typescript
const cold$ = this.http.get('/api/data');

// Using share() - becomes hot
const hot$ = cold$.pipe(share());
hot$.subscribe(data => console.log('A:', data)); // HTTP call
hot$.subscribe(data => console.log('B:', data)); // NO new HTTP call

// Using shareReplay() - becomes hot + caches
const cached$ = cold$.pipe(shareReplay(1));
```

### Q: Do we need to unsubscribe from every observable?
**Answer:** **Not always!** Here's when:

**✅ Must unsubscribe (manual subscriptions):**
- `interval()`, `fromEvent()`
- Custom Observables
- Long-lived subscriptions

**❌ No need to unsubscribe (auto-completed):**
- HTTP calls (`HttpClient`)
- `of()`, `from()`
- Finite Observables

**Best practices:**
```typescript
// 1. Use async pipe (auto unsubscribes)
@Component({
  template: `<div>{{ users$ | async }}</div>`
})
export class Component {
  users$ = this.userService.getUsers();
}

// 2. Use takeUntilDestroyed (Angular 16+)
export class Component {
  private destroyRef = inject(DestroyRef);
  
  ngOnInit() {
    this.service.data$
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe(data => this.data = data);
  }
}

// 3. Manual cleanup (old way)
export class Component implements OnDestroy {
  private destroy$ = new Subject<void>();
  
  ngOnInit() {
    this.service.data$
      .pipe(takeUntil(this.destroy$))
      .subscribe(data => this.data = data);
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

---

## 8. Subject vs BehaviorSubject
**Concept:**
- **`Subject`**: Does not have an initial value. Subscribers only receive values emitted *after* they subscribe.
- **`BehaviorSubject`**: Requires an initial value. New subscribers immediately receive the *last emitted value* (or initial value).

**Example:**
```typescript
// Subject
const sub = new Subject<number>();
sub.subscribe(v => console.log('Sub A:', v));
sub.next(1); // Sub A: 1
sub.subscribe(v => console.log('Sub B:', v)); // Sub B gets nothing yet

// BehaviorSubject
const bSub = new BehaviorSubject<number>(0);
bSub.subscribe(v => console.log('BSub A:', v)); // BSub A: 0
bSub.next(1); // BSub A: 1
bSub.subscribe(v => console.log('BSub B:', v)); // BSub B: 1 (Immediate)
```

**Note:** Section 9 (Handling Large Lists) is available in the Scenario-Based file.

---

## 10. Lazy Loading & Bundle Splitting-----|
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

**Note:** Section 11 (Handling Multiple API Failures) is available in the Scenario-Based file.

---

## 12. Unknown in Angular (Strict Typing)

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

**Note:** Section 13 (Highlight Matching Words) is available in the Scenario-Based file.

---

## 14. Microtask vs Macrotask
**Concept:** JavaScript's event loop processes tasks in different queues with different priorities.

### Macrotasks (Task Queue)
- **Lower priority** - Executed after microtasks
- **Examples:** setTimeout, setInterval, setImmediate, I/O operations, UI rendering

### Microtasks (Job Queue)
- **Higher priority** - Executed before macrotasks
- **Examples:** Promises (.then, .catch, .finally), async/await, queueMicrotask(), MutationObserver

### Execution Order:
```
1. Execute synchronous code (Call Stack)
2. Execute ALL Microtasks (Promise callbacks)
3. Execute ONE Macrotask (setTimeout callback)
4. Execute ALL Microtasks again
5. Render UI (if needed)
6. Repeat from step 3
```

**Example:**
```typescript
console.log('1. Synchronous start');

setTimeout(() => {
  console.log('2. Macrotask - setTimeout');
}, 0);

Promise.resolve().then(() => {
  console.log('3. Microtask - Promise');
});

queueMicrotask(() => {
  console.log('4. Microtask - queueMicrotask');
});

console.log('5. Synchronous end');

// Output:
// 1. Synchronous start
// 5. Synchronous end
// 3. Microtask - Promise
// 4. Microtask - queueMicrotask
// 2. Macrotask - setTimeout
```

**Complex Example:**
```typescript
console.log('Start');

setTimeout(() => {
  console.log('Timeout 1');
  Promise.resolve().then(() => console.log('Promise inside Timeout 1'));
}, 0);

Promise.resolve()
  .then(() => {
    console.log('Promise 1');
    setTimeout(() => console.log('Timeout inside Promise 1'), 0);
  })
  .then(() => console.log('Promise 2'));

setTimeout(() => console.log('Timeout 2'), 0);

console.log('End');

// Output:
// Start
// End
// Promise 1
// Promise 2
// Timeout 1
// Promise inside Timeout 1
// Timeout inside Promise 1
// Timeout 2
```

**Real-World Analogy:**
Think of a **restaurant kitchen**:
- **Synchronous Code** = Chef cooking current order (blocking)
- **Microtasks** = Quick garnishes that must be done immediately before serving
- **Macrotasks** = New orders waiting in queue

**Angular Context:**
```typescript
@Component({...})
export class MyComponent {
  ngOnInit() {
    console.log('1. Synchronous');
    
    // Macrotask
    setTimeout(() => {
      console.log('3. Timeout (Macrotask)');
    }, 0);
    
    // Microtask (Promise from HTTP call)
    this.http.get('/api/data').subscribe(data => {
      console.log('2. HTTP Response (Microtask)');
    });
  }
}
```

---


