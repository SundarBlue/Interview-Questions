# Angular & RxJS Interview Questions
## Hard Level Questions (Sections 15-20)

---

## Additional Advanced Questions

### Q: Have you created a custom RxJS operator?
**Answer:** Yes, custom operators allow you to create reusable logic for Observable streams.

**Example 1: Retry with exponential backoff**
```typescript
import { Observable, throwError, timer } from 'rxjs';
import { mergeMap, finalize } from 'rxjs/operators';

export function retryWithBackoff(maxRetries: number = 3, delayMs: number = 1000) {
  return <T>(source: Observable<T>) => source.pipe(
    retryWhen(errors => errors.pipe(
      mergeMap((error, index) => {
        const retryAttempt = index + 1;
        
        if (retryAttempt > maxRetries) {
          return throwError(() => error);
        }
        
        const delay = delayMs * Math.pow(2, index);
        console.log(`Retry attempt ${retryAttempt} after ${delay}ms`);
        
        return timer(delay);
      })
    ))
  );
}

// Usage
this.http.get('/api/data').pipe(
  retryWithBackoff(3, 1000)
).subscribe(data => console.log(data));
```

**Example 2: Log operator**
```typescript
import { tap } from 'rxjs/operators';

export function log<T>(message: string) {
  return tap<T>({
    next: value => console.log(`${message}:`, value),
    error: error => console.error(`${message} ERROR:`, error),
    complete: () => console.log(`${message} COMPLETE`)
  });
}

// Usage
this.http.get('/api/users').pipe(
  log('Fetching users')
).subscribe();
```

**Example 3: Cache operator**
```typescript
import { Observable, of } from 'rxjs';
import { tap, shareReplay } from 'rxjs/operators';

const cache = new Map<string, Observable<any>>();

export function cache<T>(key: string) {
  return (source: Observable<T>) => {
    if (cache.has(key)) {
      return cache.get(key) as Observable<T>;
    }
    
    const cached$ = source.pipe(
      shareReplay(1),
      finalize(() => cache.delete(key))
    );
    
    cache.set(key, cached$);
    return cached$;
  };
}

// Usage
this.http.get('/api/config').pipe(
  cache('app-config')
).subscribe();
```

### Q: Implemented custom decorators?
**Answer:** Yes, custom decorators add metadata or modify class behavior.

**Example 1: @Debounce decorator**
```typescript
export function Debounce(delay: number = 300) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const original = descriptor.value;
    let timeout: any;
    
    descriptor.value = function (...args: any[]) {
      clearTimeout(timeout);
      timeout = setTimeout(() => {
        original.apply(this, args);
      }, delay);
    };
    
    return descriptor;
  };
}

// Usage
@Component({...})
export class SearchComponent {
  @Debounce(500)
  onSearch(query: string) {
    console.log('Searching:', query);
    this.searchService.search(query).subscribe();
  }
}
```

**Example 2: @Memoize decorator (cache method results)**
```typescript
export function Memoize() {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const original = descriptor.value;
    const cache = new Map<string, any>();
    
    descriptor.value = function (...args: any[]) {
      const key = JSON.stringify(args);
      
      if (cache.has(key)) {
        return cache.get(key);
      }
      
      const result = original.apply(this, args);
      cache.set(key, result);
      return result;
    };
    
    return descriptor;
  };
}

// Usage
@Component({...})
export class Component {
  @Memoize()
  calculateExpensiveValue(a: number, b: number): number {
    console.log('Calculating...'); // Only logs once per unique input
    return a * b * Math.random();
  }
}
```

**Example 3: @Loading decorator (show loading state)**
```typescript
export function Loading(loadingPropertyName: string = 'loading') {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const original = descriptor.value;
    
    descriptor.value = async function (...args: any[]) {
      this[loadingPropertyName] = true;
      
      try {
        const result = await original.apply(this, args);
        return result;
      } finally {
        this[loadingPropertyName] = false;
      }
    };
    
    return descriptor;
  };
}

// Usage
@Component({...})
export class Component {
  isLoading = false;
  
  @Loading('isLoading')
  async fetchData() {
    // isLoading automatically set to true
    const data = await this.http.get('/api/data').toPromise();
    // isLoading automatically set to false
    return data;
  }
}
```

### Q: How have you used TypeScript generics for complex interfaces?
**Answer:** Generics create reusable, type-safe code for complex data structures.

**Example 1: Generic API Response**
```typescript
interface ApiResponse<T> {
  data: T;
  message: string;
  status: number;
  timestamp: Date;
}

// Generic service
@Injectable({ providedIn: 'root' })
export class ApiService {
  constructor(private http: HttpClient) {}
  
  get<T>(url: string): Observable<T> {
    return this.http.get<ApiResponse<T>>(url).pipe(
      map(response => response.data)
    );
  }
}

// Usage with type safety
interface User {
  id: number;
  name: string;
}

// TypeScript knows this returns Observable<User[]>
const users$ = this.apiService.get<User[]>('/api/users');
```

**Example 2: Generic State Management**
```typescript
interface State<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

class StateManager<T> {
  private state$ = new BehaviorSubject<State<T>>({
    data: null,
    loading: false,
    error: null
  });
  
  getData(): Observable<T | null> {
    return this.state$.pipe(map(s => s.data));
  }
  
  setLoading(loading: boolean) {
    this.state$.next({ ...this.state$.value, loading });
  }
  
  setData(data: T) {
    this.state$.next({ data, loading: false, error: null });
  }
  
  setError(error: string) {
    this.state$.next({ ...this.state$.value, loading: false, error });
  }
}

// Usage
const userState = new StateManager<User>();
const productState = new StateManager<Product[]>();
```

**Example 3: Generic Form Builder**
```typescript
type FormControls<T> = {
  [K in keyof T]: FormControl<T[K]>;
};

function createTypedFormGroup<T>(
  initialValue: T
): FormGroup<FormControls<T>> {
  const controls = {} as FormControls<T>;
  
  for (const key in initialValue) {
    controls[key] = new FormControl(initialValue[key]);
  }
  
  return new FormGroup(controls);
}

// Usage with full type safety
interface UserForm {
  name: string;
  email: string;
  age: number;
}

const form = createTypedFormGroup<UserForm>({
  name: '',
  email: '',
  age: 0
});

// TypeScript knows these types!
form.controls.name.setValue('John'); // ✅ string
form.controls.age.setValue(25);      // ✅ number
form.controls.age.setValue('25');    // ❌ Error: Type 'string' not assignable to 'number'
```

### Q: Unconventional challenges in Angular apps (scalability with Nx, performance)?
**Answer:**

**1. Scalability with Nx Monorepo:**
```bash
# Create Nx workspace
npx create-nx-workspace@latest myorg --preset=angular-monorepo

# Generate apps
nx g @nx/angular:app customer-portal
nx g @nx/angular:app admin-dashboard

# Generate shared libraries
nx g @nx/angular:lib shared/ui
nx g @nx/angular:lib shared/data-access
nx g @nx/angular:lib shared/utils

# Run specific app
nx serve customer-portal
```

**Library structure:**
```
libs/
  ├── shared/
  │   ├── ui/              (Reusable components)
  │   ├── data-access/     (Services, state)
  │   └── utils/           (Helper functions)
  ├── customer/
  │   ├── feature-cart/
  │   └── feature-checkout/
  └── admin/
      └── feature-users/
```

**2. Performance challenges:**

**Problem: Large component tree**
```typescript
// Solution: Use OnPush everywhere
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class OptimizedComponent {
  // Use signals for reactive state
  users = signal<User[]>([]);
  filteredUsers = computed(() => 
    this.users().filter(u => u.active)
  );
}
```

**Problem: Heavy initial bundle**
```typescript
// Solution: Aggressive lazy loading
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes')
  },
  {
    path: 'reports',
    loadComponent: () => import('./reports/reports.component')
  }
];

// Preload strategy for critical routes
export class CustomPreloadStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    return route.data?.['preload'] ? load() : of(null);
  }
}
```

**Problem: Complex state management**
```typescript
// Solution: Signals + RxJS interop (Angular 16+)
@Injectable({ providedIn: 'root' })
export class CartStore {
  // State as signals
  private items = signal<CartItem[]>([]);
  private loading = signal(false);
  
  // Computed values
  readonly total = computed(() => 
    this.items().reduce((sum, item) => sum + item.price, 0)
  );
  
  // Expose as readonly
  readonly items$ = toObservable(this.items);
  
  // Actions
  addItem(item: CartItem) {
    this.items.update(items => [...items, item]);
  }
}
```

### Q: Manage API type consistency (GraphQL, tRPC)?
**Answer:**

**Option 1: GraphQL with Code Generation**
```typescript
// Install
npm install @apollo/client graphql

// schema.graphql
type User {
  id: ID!
  name: String!
  email: String!
}

// Generate types
npx graphql-codegen

// Generated types
interface User {
  id: string;
  name: string;
  email: string;
}

// Type-safe queries
@Injectable({ providedIn: 'root' })
export class UserService {
  constructor(private apollo: Apollo) {}
  
  getUsers(): Observable<User[]> {
    return this.apollo.query<{ users: User[] }>({
      query: gql`
        query GetUsers {
          users {
            id
            name
            email
          }
        }
      `
    }).pipe(map(result => result.data.users));
  }
}
```

**Option 2: tRPC (End-to-end type safety)**
```typescript
// Backend (Express + tRPC)
import { initTRPC } from '@trpc/server';

const t = initTRPC.create();

const appRouter = t.router({
  getUsers: t.procedure.query(() => {
    return db.users.findMany();
  }),
  createUser: t.procedure
    .input(z.object({ name: z.string(), email: z.string() }))
    .mutation(({ input }) => {
      return db.users.create(input);
    })
});

export type AppRouter = typeof appRouter;

// Frontend (Angular)
import { inject } from '@angular/core';
import { createTRPCProxyClient, httpBatchLink } from '@trpc/client';
import type { AppRouter } from './server';

const trpc = createTRPCProxyClient<AppRouter>({
  links: [httpBatchLink({ url: 'http://localhost:3000/trpc' })]
});

@Injectable({ providedIn: 'root' })
export class TrpcService {
  // Fully type-safe!
  async getUsers() {
    return await trpc.getUsers.query();
    // TypeScript knows the return type!
  }
  
  async createUser(name: string, email: string) {
    return await trpc.createUser.mutate({ name, email });
  }
}
```

**Option 3: OpenAPI/Swagger Code Generation**
```bash
# Generate Angular service from OpenAPI spec
npx @openapitools/openapi-generator-cli generate \
  -i swagger.json \
  -g typescript-angular \
  -o src/app/api

# Auto-generated type-safe services
import { UsersService, User } from './api';

@Component({...})
export class Component {
  constructor(private usersApi: UsersService) {}
  
  ngOnInit() {
    // Fully typed!
    this.usersApi.getUsers().subscribe(
      (users: User[]) => console.log(users)
    );
  }
}
```

---

## 15. AOT (Ahead-of-Time) vs JIT (Just-in-Time) Compilation
**Concept:** How Angular compiles your TypeScript and templates into JavaScript.

### JIT (Just-in-Time) Compilation
- Compiles in the **browser at runtime**
- Used during **development** (`ng serve`)
- **Slower** initial load (compilation happens in browser)
- **Larger** bundle size (includes Angular compiler)
- **Better** for debugging (source maps available)

### AOT (Ahead-of-Time) Compilation
- Compiles **during build time** (before deployment)
- Used in **production** (`ng build --prod`)
- **Faster** initial load (pre-compiled)
- **Smaller** bundle size (no compiler shipped)
- **Better security** (templates pre-compiled, less injection risks)
- **Early error detection** (template errors found at build time)

### Comparison:

| Feature | JIT | AOT |
|---------|-----|-----|
| **Compilation** | In browser (runtime) | During build (before deployment) |
| **Speed** | Slower initial load | Faster initial load |
| **Bundle Size** | Larger (includes compiler ~1MB) | Smaller (no compiler) |
| **Error Detection** | At runtime | At build time |
| **Use Case** | Development | Production |
| **Command** | `ng serve` | `ng build --prod` |

**Example:**

```typescript
// Component with template
@Component({
  selector: 'app-user',
  template: `
    <h1>{{ userName }}</h1>
    <button (click)="save()">Save</button>
  `
})
export class UserComponent {
  userName = 'John';
  save() { }
}
```

**JIT Process:**
```
1. Browser downloads Angular app
2. Browser downloads Angular compiler (~1MB)
3. Compiler compiles templates in browser
4. App renders (Slower)
```

**AOT Process:**
```
1. ng build compiles templates on your machine
2. Browser downloads pre-compiled code (no compiler)
3. App renders immediately (Faster)
```

**How to enable AOT:**
```bash
# Development (JIT by default)
ng serve

# Production (AOT by default)
ng build --prod

# Force AOT in development
ng serve --aot
```

**Benefits of AOT:**
- ⚡ **Faster rendering** - No compilation in browser
- 📦 **Smaller bundle** - Compiler not included (~40% smaller)
- 🔒 **More secure** - Templates pre-compiled, reduces injection attacks
- 🐛 **Early detection** - Template errors caught during build
- 🎯 **Better tree-shaking** - Unused code removed more effectively

---

## 16. Signals (Angular 16+)
**Concept:** A new reactive primitive in Angular for managing state and change detection more efficiently. Signals are an alternative to RxJS Observables for simpler state management.

### What is a Signal?
A signal is a wrapper around a value that notifies consumers when the value changes.

**Example - Basic Signal:**
```typescript
import { Component, signal, computed, effect } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <h2>Count: {{ count() }}</h2>
    <h3>Double: {{ doubleCount() }}</h3>
    <button (click)="increment()">Increment</button>
    <button (click)="reset()">Reset</button>
  `
})
export class CounterComponent {
  // Create a signal
  count = signal(0);
  
  // Computed signal (derived value)
  doubleCount = computed(() => this.count() * 2);
  
  // Effect (side effect when signal changes)
  constructor() {
    effect(() => {
      console.log('Count changed to:', this.count());
    });
  }
  
  increment() {
    this.count.set(this.count() + 1); // Set new value
    // OR
    // this.count.update(value => value + 1); // Update based on current
  }
  
  reset() {
    this.count.set(0);
  }
}
```

### Signal Methods:

| Method | Purpose | Example |
|--------|---------|---------|
| `signal(initialValue)` | Create a signal | `count = signal(0)` |
| `signal()` | Read value | `this.count()` |
| `set(newValue)` | Set new value | `this.count.set(10)` |
| `update(fn)` | Update based on current | `this.count.update(v => v + 1)` |
| `computed(() => {})` | Derived value | `double = computed(() => count() * 2)` |
| `effect(() => {})` | Side effect on change | `effect(() => console.log(count()))` |

**Example - Form with Signals:**
```typescript
@Component({
  selector: 'app-user-form',
  template: `
    <input [value]="username()" (input)="updateUsername($event)" />
    <p>Username: {{ username() }}</p>
    <p>Length: {{ usernameLength() }}</p>
    <p *ngIf="isValid()">✅ Valid</p>
    <p *ngIf="!isValid()">❌ Invalid</p>
  `
})
export class UserFormComponent {
  username = signal('');
  
  // Computed signals
  usernameLength = computed(() => this.username().length);
  isValid = computed(() => this.username().length >= 3);
  
  updateUsername(event: Event) {
    const input = event.target as HTMLInputElement;
    this.username.set(input.value);
  }
  
  constructor() {
    // Effect runs whenever username changes
    effect(() => {
      if (this.isValid()) {
        console.log('Valid username:', this.username());
      }
    });
  }
}
```

### Signals vs RxJS Observables:

| Feature | Signals | RxJS Observables |
|---------|---------|------------------|
| **Learning Curve** | Simple | Steeper |
| **Boilerplate** | Less | More (subscribe, unsubscribe) |
| **Memory Leaks** | No (auto cleanup) | Possible (if not unsubscribed) |
| **Operators** | Limited | Many (map, filter, switchMap, etc.) |
| **Use Case** | Simple state | Complex async operations |
| **Change Detection** | Automatic, efficient | Need async pipe or manual |

**When to use what:**
- **Signals**: Simple state management, component-level state, computed values
- **RxJS**: HTTP calls, complex async flows, event streams, multiple operators needed

**Example - Combining Signals with HTTP:**
```typescript
@Component({...})
export class UserListComponent {
  users = signal<User[]>([]);
  loading = signal(false);
  error = signal<string | null>(null);
  
  constructor(private http: HttpClient) {}
  
  loadUsers() {
    this.loading.set(true);
    this.error.set(null);
    
    this.http.get<User[]>('/api/users').subscribe({
      next: (data) => {
        this.users.set(data);
        this.loading.set(false);
      },
      error: (err) => {
        this.error.set(err.message);
        this.loading.set(false);
      }
    });
  }
}
```

---

## 17. Angular Lifecycle Hooks
**Concept:** Lifecycle hooks are methods that Angular calls at specific moments in a component's lifecycle.

### Lifecycle Hook Order:

```
Constructor → ngOnChanges → ngOnInit → ngDoCheck → ngAfterContentInit 
→ ngAfterContentChecked → ngAfterViewInit → ngAfterViewChecked → ngOnDestroy
```

### All Lifecycle Hooks:

| Hook | Called When | Use Case |
|------|-------------|----------|
| `constructor()` | Instance created | Dependency injection only |
| `ngOnChanges()` | @Input properties change | React to input changes |
| `ngOnInit()` | After first ngOnChanges | Initialize component, API calls |
| `ngDoCheck()` | Every change detection | Custom change detection |
| `ngAfterContentInit()` | After <ng-content> initialized | Access projected content |
| `ngAfterContentChecked()` | After content checked | After content change detection |
| `ngAfterViewInit()` | After view initialized | Access @ViewChild elements |
| `ngAfterViewChecked()` | After view checked | After view change detection |
| `ngOnDestroy()` | Before component destroyed | Cleanup, unsubscribe |

**Complete Example:**
```typescript
@Component({
  selector: 'app-lifecycle',
  template: `
    <h2>{{ title }}</h2>
    <div #contentDiv>Content</div>
    <ng-content></ng-content>
  `
})
export class LifecycleComponent implements OnInit, OnChanges, OnDestroy, 
    AfterViewInit, AfterContentInit {
  
  @Input() title: string = '';
  @ViewChild('contentDiv') contentDiv!: ElementRef;
  
  private subscription!: Subscription;
  
  // 1. Constructor - Dependency Injection
  constructor(private apiService: ApiService) {
    console.log('1. Constructor called');
    // Only DI here, no DOM access
  }
  
  // 2. ngOnChanges - Called when @Input changes
  ngOnChanges(changes: SimpleChanges): void {
    console.log('2. ngOnChanges called', changes);
    if (changes['title']) {
      console.log('Title changed from', changes['title'].previousValue, 
        'to', changes['title'].currentValue);
    }
  }
  
  // 3. ngOnInit - Component initialization
  ngOnInit(): void {
    console.log('3. ngOnInit called');
    // Best place for:
    // - API calls
    // - Initialize properties
    // - Set up subscriptions
    this.subscription = this.apiService.getData().subscribe(data => {
      console.log('Data loaded:', data);
    });
  }
  
  // 4. ngDoCheck - Custom change detection
  ngDoCheck(): void {
    console.log('4. ngDoCheck called');
    // Use sparingly - called very frequently!
  }
  
  // 5. ngAfterContentInit - After <ng-content> projected
  ngAfterContentInit(): void {
    console.log('5. ngAfterContentInit called');
    // Access @ContentChild here
  }
  
  // 6. ngAfterContentChecked - After content change detection
  ngAfterContentChecked(): void {
    console.log('6. ngAfterContentChecked called');
  }
  
  // 7. ngAfterViewInit - After view initialized
  ngAfterViewInit(): void {
    console.log('7. ngAfterViewInit called');
    // Safe to access @ViewChild here
    console.log('ContentDiv:', this.contentDiv.nativeElement);
  }
  
  // 8. ngAfterViewChecked - After view change detection
  ngAfterViewChecked(): void {
    console.log('8. ngAfterViewChecked called');
  }
  
  // 9. ngOnDestroy - Cleanup
  ngOnDestroy(): void {
    console.log('9. ngOnDestroy called');
    // IMPORTANT: Unsubscribe to prevent memory leaks
    if (this.subscription) {
      this.subscription.unsubscribe();
    }
  }
}
```

**Common Patterns:**

```typescript
// Pattern 1: API calls in ngOnInit
ngOnInit() {
  this.userService.getUsers().subscribe(users => {
    this.users = users;
  });
}

// Pattern 2: React to input changes
@Input() userId!: number;

ngOnChanges(changes: SimpleChanges) {
  if (changes['userId'] && !changes['userId'].firstChange) {
    this.loadUserData(this.userId);
  }
}

// Pattern 3: Access child component
@ViewChild(ChildComponent) child!: ChildComponent;

ngAfterViewInit() {
  this.child.someMethod(); // Now safe to access
}

// Pattern 4: Cleanup subscriptions
private destroy$ = new Subject<void>();

ngOnInit() {
  this.apiService.getData()
    .pipe(takeUntil(this.destroy$))
    .subscribe(data => {});
}

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}
```

---

## 18. Observable vs Promise
**Concept:** Both handle asynchronous operations, but they work differently.

### Key Differences:

| Feature | Observable | Promise |
|---------|-----------|---------|
| **Eager/Lazy** | Lazy (doesn't execute until subscribed) | Eager (executes immediately) |
| **Cancellable** | ✅ Yes (unsubscribe) | ❌ No |
| **Multiple Values** | ✅ Yes (stream of values) | ❌ No (single value) |
| **Operators** | ✅ Many (map, filter, switchMap, etc.) | ❌ Limited (then, catch) |
| **Error Handling** | Can retry, catch, and continue | Stops on error |
| **Return** | Requires subscription | Returns promise |

**Example - Observable:**
```typescript
import { Observable } from 'rxjs';

// Observable - Lazy (doesn't execute until subscribed)
const observable$ = new Observable(observer => {
  console.log('Observable started');
  observer.next(1);
  observer.next(2);
  observer.next(3);
  setTimeout(() => observer.next(4), 1000);
  setTimeout(() => observer.complete(), 2000);
});

// Nothing happens yet (Lazy)
console.log('Before subscribe');

// Subscribe to start execution
const subscription = observable$.subscribe({
  next: (value) => console.log('Value:', value),
  complete: () => console.log('Complete')
});

// Can cancel/unsubscribe
setTimeout(() => {
  subscription.unsubscribe();
  console.log('Unsubscribed');
}, 1500);

// Output:
// Before subscribe
// Observable started
// Value: 1
// Value: 2
// Value: 3
// Value: 4
// Unsubscribed (complete never called)
```

**Example - Promise:**
```typescript
// Promise - Eager (executes immediately)
const promise = new Promise((resolve, reject) => {
  console.log('Promise started immediately');
  setTimeout(() => resolve('Done'), 1000);
});

// Already executing (Eager)
console.log('Before then');

promise.then(value => console.log('Value:', value));

// Cannot cancel!

// Output:
// Promise started immediately
// Before then
// (after 1 second) Value: Done
```

**HTTP Call Example:**
```typescript
@Component({...})
export class DataComponent implements OnDestroy {
  // Observable (RxJS)
  loadDataWithObservable() {
    const subscription = this.http.get('/api/data').subscribe({
      next: (data) => console.log('Data:', data),
      error: (err) => console.error('Error:', err)
    });
    
    // Can cancel
    setTimeout(() => subscription.unsubscribe(), 500);
  }
  
  // Promise
  async loadDataWithPromise() {
    try {
      const data = await this.http.get('/api/data').toPromise();
      console.log('Data:', data);
    } catch (err) {
      console.error('Error:', err);
    }
    // Cannot cancel!
  }
  
  // Multiple values with Observable
  loadStream() {
    this.websocket.getMessages() // Returns Observable
      .subscribe(message => {
        console.log('New message:', message);
        // Can receive many messages over time
      });
  }
}
```

**When to use what:**

| Use Case | Use |
|----------|-----|
| HTTP calls (one response) | Observable or Promise |
| Cancellable requests | Observable ✅ |
| WebSockets/Events (multiple values) | Observable ✅ |
| Simple async operation | Promise |
| Need operators (map, filter, retry) | Observable ✅ |
| Legacy code | Promise |
| Angular HTTP | Observable (default) |

---

## 19. Routing: `forRoot()` vs `forChild()`
**Concept:** Methods to configure routing modules in Angular.

### forRoot()
- Used in the **root module** (AppModule) **only once**
- Registers **global services** (Router, ActivatedRoute, etc.)
- Sets up the main router configuration

### forChild()
- Used in **feature modules** (lazy-loaded or eager-loaded)
- Adds **additional routes** without registering services again
- Avoids creating multiple instances of router services

**Example - App Module (Root):**
```typescript
// app-routing.module.ts
const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule)
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)], // Use forRoot() in AppModule
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

```typescript
// app.module.ts
@NgModule({
  declarations: [AppComponent, HomeComponent, AboutComponent],
  imports: [
    BrowserModule,
    AppRoutingModule // RouterModule.forRoot() is inside
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

**Example - Feature Module (Child):**
```typescript
// admin-routing.module.ts
const routes: Routes = [
  { path: '', component: AdminDashboardComponent },
  { path: 'users', component: AdminUsersComponent },
  { path: 'settings', component: AdminSettingsComponent }
];

@NgModule({
  imports: [RouterModule.forChild(routes)], // Use forChild() in feature modules
  exports: [RouterModule]
})
export class AdminRoutingModule { }
```

```typescript
// admin.module.ts
@NgModule({
  declarations: [AdminDashboardComponent, AdminUsersComponent, AdminSettingsComponent],
  imports: [
    CommonModule,
    AdminRoutingModule // RouterModule.forChild() is inside
  ]
})
export class AdminModule { }
```

### Comparison:

| Feature | forRoot() | forChild() |
|---------|-----------|------------|
| **Usage** | AppModule (once) | Feature modules (multiple) |
| **Services** | Registers router services | Uses existing services |
| **Instances** | Creates Router instance | Reuses Router instance |
| **Purpose** | Main app routing | Additional routes |

**What happens if you use `forRoot()` in a feature module?**
❌ **Problem:** Multiple instances of Router services created → Memory leaks and unexpected behavior

**Structure:**
```
App
├── AppModule (forRoot)
│   ├── Home
│   └── About
├── AdminModule (forChild) - Lazy loaded
│   ├── Dashboard
│   ├── Users
│   └── Settings
└── UserModule (forChild) - Lazy loaded
    ├── Profile
    └── Orders
```

---

## 20. Custom Directives
**Concept:** Create reusable behaviors that can be applied to DOM elements using custom directives.

### Types of Directives:
1. **Component Directives** - Has template
2. **Structural Directives** - Change DOM structure (*ngIf, *ngFor)
3. **Attribute Directives** - Change appearance/behavior

### Example 1 - Highlight Directive (Attribute Directive)
```typescript
import { Directive, ElementRef, Renderer2, HostListener, Input } from '@angular/core';

@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  @Input() appHighlight: string = 'yellow'; // Custom color
  @Input() defaultColor: string = 'transparent';
  
  constructor(private el: ElementRef, private renderer: Renderer2) {}
  
  ngOnInit() {
    this.setBackground(this.defaultColor);
  }
  
  @HostListener('mouseenter') onMouseEnter() {
    this.setBackground(this.appHighlight);
  }
  
  @HostListener('mouseleave') onMouseLeave() {
    this.setBackground(this.defaultColor);
  }
  
  private setBackground(color: string) {
    this.renderer.setStyle(this.el.nativeElement, 'backgroundColor', color);
  }
}
```

**Usage:**
```html
<p appHighlight>Hover me (default yellow)</p>
<p [appHighlight]="'lightblue'">Hover me (custom blue)</p>
<p appHighlight="pink" defaultColor="lightgray">Hover me (pink)</p>
```

### Example 2 - Form Validation Directive (Username Alphabet Only)
```typescript
import { Directive } from '@angular/core';
import { NG_VALIDATORS, Validator, AbstractControl, ValidationErrors } from '@angular/forms';

@Directive({
  selector: '[appAlphabetOnly]',
  providers: [{
    provide: NG_VALIDATORS,
    useExisting: AlphabetOnlyDirective,
    multi: true
  }]
})
export class AlphabetOnlyDirective implements Validator {
  validate(control: AbstractControl): ValidationErrors | null {
    const value = control.value;
    
    if (!value) {
      return null; // Don't validate empty values
    }
    
    // Check if value contains only alphabets
    const alphabetPattern = /^[a-zA-Z]+$/;
    
    if (!alphabetPattern.test(value)) {
      return { 'alphabetOnly': { value: control.value } };
    }
    
    return null;
  }
}
```

**Usage in Template Form:**
```html
<form #userForm="ngForm">
  <input 
    name="username" 
    ngModel 
    appAlphabetOnly 
    #username="ngModel" 
    placeholder="Username (alphabets only)"
  />
  <div *ngIf="username.invalid && username.touched">
    <p *ngIf="username.errors?.['alphabetOnly']">
      Username must contain only alphabets!
    </p>
  </div>
</form>
```

**Usage in Reactive Form:**
```typescript
@Component({
  template: `
    <form [formGroup]="userForm">
      <input formControlName="username" appAlphabetOnly />
      <div *ngIf="username.invalid && username.touched">
        <p *ngIf="username.errors?.['alphabetOnly']">
          Username must contain only alphabets!
        </p>
      </div>
    </form>
  `
})
export class UserFormComponent {
  userForm = this.fb.group({
    username: ['', [Validators.required]]
  });
  
  get username() {
    return this.userForm.get('username')!;
  }
  
  constructor(private fb: FormBuilder) {}
}
```

### Example 3 - Tooltip Directive
```typescript
@Directive({
  selector: '[appTooltip]'
})
export class TooltipDirective {
  @Input() appTooltip: string = '';
  private tooltipElement: HTMLElement | null = null;
  
  constructor(private el: ElementRef, private renderer: Renderer2) {}
  
  @HostListener('mouseenter') onMouseEnter() {
    this.showTooltip();
  }
  
  @HostListener('mouseleave') onMouseLeave() {
    this.hideTooltip();
  }
  
  private showTooltip() {
    this.tooltipElement = this.renderer.createElement('span');
    this.renderer.appendChild(
      this.tooltipElement,
      this.renderer.createText(this.appTooltip)
    );
    
    this.renderer.appendChild(document.body, this.tooltipElement);
    this.renderer.addClass(this.tooltipElement, 'tooltip');
    
    const hostPos = this.el.nativeElement.getBoundingClientRect();
    const tooltipPos = this.tooltipElement.getBoundingClientRect();
    
    this.renderer.setStyle(this.tooltipElement, 'top', 
      `${hostPos.bottom + 5}px`);
    this.renderer.setStyle(this.tooltipElement, 'left', 
      `${hostPos.left + (hostPos.width - tooltipPos.width) / 2}px`);
  }
  
  private hideTooltip() {
    if (this.tooltipElement) {
      this.renderer.removeChild(document.body, this.tooltipElement);
      this.tooltipElement = null;
    }
  }
}
```

**Usage:**
```html
<button appTooltip="Click to save changes">Save</button>
```

```css
/* styles.css */
.tooltip {
  position: absolute;
  background: #333;
  color: white;
  padding: 5px 10px;
  border-radius: 4px;
  font-size: 12px;
  z-index: 1000;
}
```

### Example 4 - Structural Directive (*appUnless)
```typescript
@Directive({
  selector: '[appUnless]'
})
export class UnlessDirective {
  private hasView = false;
  
  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef
  ) {}
  
  @Input() set appUnless(condition: boolean) {
    if (!condition && !this.hasView) {
      this.viewContainer.createEmbeddedView(this.templateRef);
      this.hasView = true;
    } else if (condition && this.hasView) {
      this.viewContainer.clear();
      this.hasView = false;
    }
  }
}
```

**Usage:**
```html
<!-- Shows content when condition is FALSE (opposite of *ngIf) -->
<p *appUnless="isLoggedIn">Please log in to continue</p>
```
```

