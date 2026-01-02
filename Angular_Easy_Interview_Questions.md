# Angular & RxJS Interview Questions
## Easy Level Questions (Sections 2-6 + Additional Questions)

**Note:** Section 1 (Change Detection Strategies) and Section 7 (Nested API Calls) are in the Scenario-Based file.

---

## 2. Dependency Injection (DI) - What & Why?

### Simple, non-technical explanation

Dependency Injection (DI) means "you ask for what you need, and the framework gives it to you." You do not build or find the tools yourself — Angular provides them.

**Think of it like ordering food:**
- **Without DI:** You go buy ingredients and cook the meal yourself.
- **With DI:** You tell the waiter what you want and the kitchen brings it ready-to-eat.

**Visual Flow:**
```
WITHOUT DI:
Component → "I need UserService" → Goes and creates new UserService() itself
                                 → Must know HOW to create it
                                 → Must create everything UserService needs too

WITH DI (Angular provides):
Component → "I need UserService" → Angular hears this
                                 ↓
                         Angular checks if UserService exists
                                 ↓
                         Angular creates it (if needed)
                                 ↓
                         Angular gives it to Component ✅
                                 ↓
                    Component uses it (no setup needed!)
```

**Why this helps (simple bullets):**
- Makes code easier to test (replace real parts with simple fakes)
- Lets you change implementations in one place (no mass edits)
- Prevents repeated setup logic across the app
- Angular manages complex dependency chains for you

---

### Very short examples (what changes)

**Without DI:**
```typescript
class UserComponent {
  userService = new UserService(); // ❌ You create it
}
```

**With DI:**
```typescript
class UserComponent {
  constructor(private userService: UserService) {} // ✅ Angular gives it
}
```

**Result:** Both get the same functionality, but with DI the component doesn't care how the service is made — it just uses it.

---

### What DI solves (short scenarios)
- **Testing:** swap real services with mock ones for fast tests
- **Flexibility:** switch implementations (e.g., mock → real API) centrally
- **Maintainability:** fewer places to update when internal details change

**Terminology:**
- **Dependency Injection (DI):** framework provides objects your code needs
- **Inversion of Control (IoC):** your code delegates creation to the framework

---

### Quick comparison table

| Aspect | Without DI | With DI |
|--------|-----------|---------|
| **Who creates service?** | You do (`new UserService()`) | Angular does |
| **Testing** | ❌ Hard (real API calls) | ✅ Easy (mock services) |
| **Changing service** | ❌ Edit every file | ✅ Edit one config |
| **Maintenance** | ❌ Hard | ✅ Easy |

---

*Now for programmers: detailed examples and advanced patterns below.*


**With DI (✅ Change in ONE place):**

**Option 1: NgModule-based (Traditional)**
```typescript
// app.module.ts - Change ONCE here
@NgModule({
  providers: [
    { provide: UserService, useClass: UserApiService } // ✅ Changed in one place!
  ]
})
export class AppModule {}

// Components don't need ANY changes!
export class UserComponent {
  constructor(private userService: UserService) {} // ✅ Still works!
}

export class AdminComponent {
  constructor(private userService: UserService) {} // ✅ Still works!
}

export class DashboardComponent {
  constructor(private userService: UserService) {} // ✅ Still works!
}
```

**Option 2: Standalone Components (Modern Angular 14+)**
```typescript
// main.ts - Change ONCE here (Application Bootstrap)
bootstrapApplication(AppComponent, {
  providers: [
    { provide: UserService, useClass: UserApiService } // ✅ Changed in one place!
  ]
});

// OR use providedIn in the service itself (Best practice!)
@Injectable({
  providedIn: 'root' // ✅ Available everywhere automatically
})
export class UserApiService {}

// Standalone Components - No changes needed!
@Component({
  selector: 'app-user',
  standalone: true,
  template: `...`
})
export class UserComponent {
  constructor(private userService: UserService) {} // ✅ Still works!
}

@Component({
  selector: 'app-admin',
  standalone: true,
  template: `...`
})
export class AdminComponent {
  constructor(private userService: UserService) {} // ✅ Still works!
}

@Component({
  selector: 'app-dashboard',
  standalone: true,
  template: `...`
})
export class DashboardComponent {
  constructor(private userService: UserService) {} // ✅ Still works!
}
```

**Option 3: Component-Level Providers (Standalone)**
```typescript
// Provide service to this component AND ALL its children (including lazy-loaded!)
@Component({
  selector: 'app-user',
  standalone: true,
  providers: [
    { provide: UserService, useClass: UserApiService } // ✅ Scoped to this component + children
  ],
  template: `<router-outlet></router-outlet>` // Children will use UserApiService too!
})
export class UserComponent {
  constructor(private userService: UserService) {} // Gets UserApiService
}
```

### ⚠️ Important: Component Providers Affect Lazy-Loaded Children!

**Angular's Hierarchical Dependency Injection:**
- Providers at component level create a **new injector scope**
- **ALL child components** (including lazy-loaded routes) inherit from this scope
- Lazy-loaded children **WILL** get the overridden service

**Hierarchical DI Example (use DI diagram below)**

Use the diagram in **"How Angular's Injector Hierarchy Works"** (below) as the primary illustration for how component-level providers affect lazy-loaded children.

**Short explanation:**
- Providing `{ provide: UserService, useClass: UserApiService }` on a parent component creates a new injector scope.
- All child components, including lazy-loaded children rendered inside the parent's `<router-outlet>`, inherit this override and will receive `UserApiService` instead of the root `UserService`.

(Kept the concise NgModule and Component-level code examples above — the diagram below is the canonical example for hierarchy.)

### How Angular's Injector Hierarchy Works

```
Application Root (providedIn: 'root')
  └── UserService (default)
       │
       ├── AppComponent
       │    └── HomeComponent → uses UserService (default)
       │
       └── AdminComponent (providers: [{ provide: UserService, useClass: UserApiService }])
            └── Creates NEW injector scope ⚠️
                 │
                 ├── AdminComponent → uses UserApiService ✅
                 │
                 ├── AdminUsersComponent (lazy-loaded child) → uses UserApiService ✅
                 │
                 └── AdminSettingsComponent (lazy-loaded child) → uses UserApiService ✅

Result: Lazy-loaded children GET the overridden service!
```

---

### NgModule-based Hierarchical DI (lazy-loaded module override)

```
Application Root (providedIn: 'root')
  └── UserService (default)
       │
       ├── AppComponent
       │    └── HomeComponent → uses UserService (default)
       │
       └── AdminModule (lazy-loaded, providers: [{ provide: UserService, useClass: UserApiService }])
            └── Creates NEW injector scope ⚠️
                 │
                 ├── AdminComponent → uses UserApiService ✅
                 │
                 ├── AdminUsersComponent (inside AdminModule) → uses UserApiService ✅
                 │
                 └── AdminSettingsComponent (inside AdminModule) → uses UserApiService ✅

Result: Lazy-loaded module and its components GET the overridden service!
```

**Eager vs Lazy (short note):**
- Lazy-loaded module creates its own injector, so module providers override root only for that module.
- Eagerly-loaded module merges providers into the root injector (overrides become global).

**Best Practice:** Use `providedIn: 'root'` for app-wide singletons and module-level providers for feature-scoped overrides.
### Is This a Problem? It Depends!

| Scenario | Problem? | Solution |
|----------|----------|----------|
| **Want children to use override** | ✅ No problem - works as expected! | Keep provider in parent component |
| **Don't want children affected** | ❌ Yes, this is a problem | Move provider to `main.ts` or use `providedIn: 'root'` |
| **Want different service per component** | ⚠️ Each component needs own provider | Add providers to each component individually |

### Example: When This is GOOD

**Use Case:** Admin section uses different API service

```typescript
// Admin components use admin-specific API
@Component({
  selector: 'app-admin',
  standalone: true,
  providers: [
    { provide: UserService, useClass: AdminApiService } // ✅ All admin children use this
  ],
  template: `<router-outlet></router-outlet>`
})
export class AdminComponent {}

// Regular app uses default service
@Component({
  selector: 'app-public',
  standalone: true,
  template: `<router-outlet></router-outlet>`
})
export class PublicComponent {
  private userService = inject(UserService); // Uses default UserService
}
```

### Example: When This is BAD

**Problem:** Accidentally override service for all children

```typescript
// ❌ BAD: Override affects all children unintentionally
@Component({
  selector: 'app-dashboard',
  standalone: true,
  providers: [
    { provide: UserService, useClass: MockUserService } // ⚠️ All children get mock!
  ],
  template: `<router-outlet></router-outlet>`
})
export class DashboardComponent {}

// Child gets mock service (maybe not what you want!)
@Component({...})
export class ReportsComponent {
  private userService = inject(UserService); // ⚠️ Gets MockUserService (from parent)
}

// ✅ SOLUTION: Use providedIn: 'root' for global services
@Injectable({
  providedIn: 'root' // Available everywhere, no hierarchy issues
})
export class UserService {}
```

### Key Takeaways:
1. ✅ **Component providers affect ALL children** (including lazy-loaded routes)
2. ✅ **Using `{ provide: UserService, useClass: UserApiService }` is fine** - it's standard Angular DI
3. ⚠️ **Be careful** - lazy-loaded children WILL inherit the override
4. ✅ **For most cases:** Use `providedIn: 'root'` to avoid hierarchy confusion

### Comparison: NgModule vs Standalone

| Feature | **NgModule** | **Standalone Components** |
|---------|--------------|---------------------------|
| **Global Providers** | `@NgModule({ providers: [...] })` | `bootstrapApplication(App, { providers: [...] })` |
| **Service Location** | app.module.ts | main.ts |
| **Component Providers** | `@Component({ providers: [...] })` | `@Component({ providers: [...] })` |
| **Best Practice** | `providedIn: 'root'` in service | `providedIn: 'root'` in service |
| **Boilerplate** | More (need NgModule) | Less (no NgModule needed) |
| **Future** | Legacy (still supported) | ✅ Recommended by Angular team |

### Real-World Example: Switching Services

**Scenario:** Switch from `UserService` to `UserApiService` in a standalone app

```typescript
// ========== STEP 1: Define Services ==========
@Injectable({
  providedIn: 'root'
})
export class UserService {
  getAll() {
    return of([{ id: 1, name: 'Mock User' }]);
  }
}

@Injectable()
export class UserApiService {
  constructor(private http: HttpClient) {}
  
  getAll() {
    return this.http.get('/api/users');
  }
}

// ========== STEP 2: Bootstrap with Provider Override ==========
// main.ts - Change service implementation globally
bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(),
    { provide: UserService, useClass: UserApiService } // ✅ One change here!
  ]
});

// ========== STEP 3: Components Stay Unchanged ==========
@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div *ngFor="let user of users">{{ user.name }}</div>
  `
})
export class UserListComponent {
  private userService = inject(UserService); // ✅ Gets UserApiService now!
  users = [];
  
  ngOnInit() {
    this.userService.getAll().subscribe(users => {
      this.users = users; // ✅ Real API data now!
    });
  }
}

@Component({
  selector: 'app-admin-users',
  standalone: true,
  template: `...`
})
export class AdminUsersComponent {
  private userService = inject(UserService); // ✅ Gets UserApiService automatically!
}

@Component({
  selector: 'app-dashboard',
  standalone: true,
  template: `...`
})
export class DashboardComponent {
  private userService = inject(UserService); // ✅ Gets UserApiService automatically!
}
```

### Best Practice: Use `providedIn: 'root'` (Works for Both!)

```typescript
// This is the SIMPLEST and BEST approach for most services
@Injectable({
  providedIn: 'root' // ✅ Available everywhere, tree-shakable
})
export class UserService {
  getAll() {
    return this.http.get('/api/users');
  }
}

// No need to add to providers array anywhere!
// Works in both NgModule and Standalone apps automatically!
```

#### Problem 3: Service Dependencies (Nightmare Without DI)

**Without DI (❌ Cascading dependencies):**
```typescript
export class UserService {
  private http: HttpClient;
  private logger: LoggerService;
  private config: ConfigService;
  
  constructor() {
    // You must create ALL dependencies manually!
    this.http = new HttpClient(); // ❌ But HttpClient needs more dependencies!
    this.logger = new LoggerService(); // ❌ LoggerService also needs dependencies!
    this.config = new ConfigService(); // ❌ ConfigService also needs dependencies!
    // This becomes a nightmare! 😱
  }
}

export class UserComponent {
  private userService: UserService;
  
  constructor() {
    // You must know HOW to create UserService and ALL its dependencies!
    this.userService = new UserService(); // ❌ What if UserService constructor changes?
  }
}
```

**With DI (✅ Angular handles everything):**
```typescript
export class UserService {
  // Angular provides all dependencies automatically!
  constructor(
    private http: HttpClient,
    private logger: LoggerService,
    private config: ConfigService
  ) {} // ✅ Angular resolves ALL dependencies!
}

export class UserComponent {
  // You just ask for UserService, Angular figures out the rest!
  constructor(private userService: UserService) {} // ✅ So simple!
}
```

### Comparison Table: Same Result, Different Approach

| Aspect | **Without DI (Tight Coupling)** | **With DI (Loose Coupling)** |
|--------|----------------------------------|------------------------------|
| **Final Result** | ✅ Gets users from API | ✅ Gets users from API |
| **Functionality** | ✅ Same | ✅ Same |
| **Code You Write** | `new UserService()` | `constructor(private userService: UserService)` |
| **Who Creates Service?** | YOU create it manually | Angular creates it for you |
| **Testing** | ❌ Cannot mock (real API calls) | ✅ Easy to mock |
| **Changing Implementation** | ❌ Change every file | ✅ Change one config |
| **Service Dependencies** | ❌ Must create manually | ✅ Angular handles it |
| **Singleton (One Instance)** | ❌ New instance per component | ✅ Angular provides same instance |
| **Maintenance** | ❌ Hard (scattered code) | ✅ Easy (centralized) |

### Visual Comparison

```
WITHOUT DI (You do everything):
Component → new UserService() → new HttpClient() → new HttpHandler() → ...
           ↑
    You must create the entire chain manually! 😱

WITH DI (Angular does everything):
Component → asks for UserService
           ↓
         Angular → "I'll give you UserService"
                → "UserService needs HttpClient? I'll provide it"
                → "HttpClient needs HttpHandler? I'll provide it"
                → Everything handled automatically! 😊
```

### Key Takeaway:
- **Same result** (both get users)
- **Different approach** (who creates the service)
- **DI is better** because it makes code testable, flexible, and maintainable

---

## 2.1. Constructor Injection vs inject() Function

### Traditional Constructor Injection (Old Way)
```typescript
import { Component } from '@angular/core';
import { UserService } from './user.service';
import { HttpClient } from '@angular/common/http';
import { Router } from '@angular/router';

@Component({
  selector: 'app-user',
  template: `<h1>Users</h1>`
})
export class UserComponent {
  // Dependencies injected via constructor
  constructor(
    private userService: UserService,
    private http: HttpClient,
    private router: Router
  ) {}
  
  loadUsers() {
    this.userService.getAll();
  }
}
```

### New inject() Function (Angular 14+)
```typescript
import { Component, inject } from '@angular/core';
import { UserService } from './user.service';
import { HttpClient } from '@angular/common/http';
import { Router } from '@angular/router';

@Component({
  selector: 'app-user',
  template: `<h1>Users</h1>`
})
export class UserComponent {
  // Dependencies injected using inject() function
  private userService = inject(UserService);
  private http = inject(HttpClient);
  private router = inject(Router);
  
  loadUsers() {
    this.userService.getAll();
  }
}
```

### Comparison: Constructor vs inject()

| Feature | **Constructor Injection** | **inject() Function** |
|---------|---------------------------|----------------------|
| **Syntax** | Verbose (parameters in constructor) | Cleaner (property initialization) |
| **Boilerplate** | More code | Less code |
| **Readability** | Constructor can get long with many dependencies | Properties listed clearly |
| **Conditional Injection** | ❌ Not possible | ✅ Possible (if/else logic) |
| **Outside Constructor** | ❌ Only in constructor | ✅ Can use in functions, variables |
| **TypeScript** | Strong typing automatic | Strong typing automatic |
| **Compatibility** | Works in all versions | Angular 14+ only |
| **Testing** | Easy to mock | Easy to mock |

### Which is Better? ✅ inject() is the Modern Choice

**Reasons:**

**1. Less Boilerplate**
```typescript
// Constructor - 10 lines
constructor(
  private serviceA: ServiceA,
  private serviceB: ServiceB,
  private serviceC: ServiceC,
  private serviceD: ServiceD
) {}

// inject() - 4 lines (cleaner!)
private serviceA = inject(ServiceA);
private serviceB = inject(ServiceB);
private serviceC = inject(ServiceC);
private serviceD = inject(ServiceD);
```

**2. Conditional Injection (inject() only!)**
```typescript
export class DashboardComponent {
  // Inject different service based on environment
  private apiService = inject(
    environment.production ? ProdApiService : DevApiService
  );
  
  // Optional injection
  private analyticsService = inject(AnalyticsService, { optional: true });
}
```

**3. Use Outside Constructor**
```typescript
export class ReportComponent {
  // Can use inject() in property initializers
  private logger = inject(LoggerService);
  
  // Can use in arrow functions
  private getUserName = () => {
    const authService = inject(AuthService); // ✅ Works!
    return authService.currentUser.name;
  };
}
```

**4. Functional Guards/Resolvers (Modern Angular)**
```typescript
// Old way - Class-based guard
@Injectable()
export class AuthGuard implements CanActivate {
  constructor(private authService: AuthService, private router: Router) {}
  
  canActivate() {
    return this.authService.isAuthenticated();
  }
}

// New way - Functional guard with inject()
export const authGuard: CanActivateFn = () => {
  const authService = inject(AuthService);
  const router = inject(Router);
  
  if (authService.isAuthenticated()) {
    return true;
  }
  
  return router.createUrlTree(['/login']);
};

// Usage in routes
const routes: Routes = [
  { path: 'dashboard', component: DashboardComponent, canActivate: [authGuard] }
];
```

**5. Standalone Components (Angular 15+)**
```typescript
@Component({
  selector: 'app-products',
  standalone: true,
  imports: [CommonModule],
  template: `...`
})
export class ProductsComponent {
  // inject() works perfectly with standalone components
  private productService = inject(ProductService);
  private cartService = inject(CartService);
  
  products = this.productService.getAll();
}
```

### Real-World Comparison

```typescript
// ❌ Old Way - Constructor (verbose, harder to read)
@Component({...})
export class CheckoutComponent implements OnInit {
  constructor(
    private cartService: CartService,
    private paymentService: PaymentService,
    private orderService: OrderService,
    private userService: UserService,
    private notificationService: NotificationService,
    private router: Router,
    private activatedRoute: ActivatedRoute,
    private fb: FormBuilder
  ) {}
  
  ngOnInit() {
    const userId = this.activatedRoute.snapshot.params['id'];
    this.loadUserData(userId);
  }
}

// ✅ New Way - inject() (clean, easy to read)
@Component({...})
export class CheckoutComponent implements OnInit {
  private cartService = inject(CartService);
  private paymentService = inject(PaymentService);
  private orderService = inject(OrderService);
  private userService = inject(UserService);
  private notificationService = inject(NotificationService);
  private router = inject(Router);
  private activatedRoute = inject(ActivatedRoute);
  private fb = inject(FormBuilder);
  
  ngOnInit() {
    const userId = this.activatedRoute.snapshot.params['id'];
    this.loadUserData(userId);
  }
}
```

### Summary Table: When to Use What?

| Scenario | Recommendation |
|----------|----------------|
| **New Angular projects (v14+)** | ✅ Use `inject()` |
| **Standalone components** | ✅ Use `inject()` |
| **Functional guards/resolvers** | ✅ Use `inject()` (only option) |
| **Legacy projects (pre-v14)** | Use constructor injection |
| **Need conditional injection** | ✅ Use `inject()` |
| **Team prefers traditional style** | Use constructor injection |

**Official Recommendation:** Angular team encourages using `inject()` for new code as it's more flexible and aligns with modern Angular patterns (standalone, functional APIs).

---

## 2.2. Dependency Injection Decorators (@Self, @SkipSelf, @Optional)

### What are DI Decorators?

These decorators control **WHERE** Angular looks for a service when you inject it. They tell Angular's injector system to search in specific places or behave differently if the service is not found.

**Think of it like asking for help in an office building:**
- **@Self()** = "Only ask people in MY office (this room only)"
- **@SkipSelf()** = "Don't ask me, ask people on other floors (skip my level)"
- **@Optional()** = "If nobody has it, that's okay, I won't complain (no error)"

---

### Visual: Angular's Injector Hierarchy (Who Has the Service?)

```
Root Injector (providedIn: 'root')
  └── LoggerService ✅ (available here)
       │
       ├── ParentComponent
       │    └── LoggerService ✅ (also provided here - NEW instance)
       │         │
       │         └── ChildComponent 🎯 (asks for LoggerService)
       │              └── Which LoggerService will I get?
       │                   - @Self() → Look ONLY in ChildComponent ❌ (not found → ERROR!)
       │                   - @SkipSelf() → Skip ChildComponent, look in ParentComponent ✅ (found!)
       │                   - No decorator → Look in ChildComponent first, then up ✅ (finds Parent's)
       │                   - @Optional() → If not found anywhere, return null (no error)
```

---

### Decorator 1: @Self() - "Only My Level"

**What it does:** Searches ONLY in the current component's injector. Does NOT look up the hierarchy.

**Visual Flow:**
```
Root (has LoggerService)
  ↓
ParentComponent (has LoggerService)
  ↓
ChildComponent → constructor(@Self() logger: LoggerService)
                 ↓
            Looks ONLY here ❌ (not found)
                 ↓
            Throws ERROR! ❌
```

**Example:**
```typescript
@Component({
  selector: 'app-child',
  providers: [LoggerService] // ✅ Provided at component level
})
class ChildComponent {
  constructor(@Self() private logger: LoggerService) {
    // ✅ Works! Found in ChildComponent's own injector
  }
}

@Component({
  selector: 'app-child2'
  // ❌ No providers here
})
class Child2Component {
  constructor(@Self() private logger: LoggerService) {
    // ❌ ERROR! Not found in Child2Component's injector
    // Won't look in parent or root
  }
}
```

**Use case:** When you want to ensure a component uses its OWN instance, not a shared one.

---

### Decorator 2: @SkipSelf() - "Skip My Level, Ask Above"

**What it does:** Skips the current component's injector and searches in parent injectors only.

**Visual Flow:**
```
Root (has LoggerService) ✅
  ↓
ParentComponent (has LoggerService) ✅
  ↓
ChildComponent → constructor(@SkipSelf() logger: LoggerService)
                 ↓
            SKIPS ChildComponent's injector (even if it has one)
                 ↓
            Looks in ParentComponent ✅ (found!)
```

**Example:**
```typescript
@Component({
  selector: 'app-parent',
  providers: [LoggerService] // ✅ Provided here
})
class ParentComponent {}

@Component({
  selector: 'app-child',
  providers: [LoggerService] // Has its own instance
})
class ChildComponent {
  constructor(@SkipSelf() private logger: LoggerService) {
    // ✅ Skips its own LoggerService
    // ✅ Gets ParentComponent's LoggerService instead
  }
}
```

**Use case:** When you want the parent's service, not your own.

---

### Decorator 3: @Optional() - "It's Okay if Not Found"

**What it does:** If the service is not found anywhere, returns `null` instead of throwing an error.

**Visual Flow:**
```
Root (NO LoggerService) ❌
  ↓
ParentComponent (NO LoggerService) ❌
  ↓
ChildComponent → constructor(@Optional() logger: LoggerService)
                 ↓
            Searches everywhere... not found
                 ↓
            Returns null (no error) ✅
```

**Example:**
```typescript
@Component({
  selector: 'app-child'
  // No providers anywhere for AnalyticsService
})
class ChildComponent {
  constructor(@Optional() private analytics: AnalyticsService | null) {
    if (this.analytics) {
      this.analytics.trackEvent('page_view'); // ✅ Use if available
    } else {
      console.log('Analytics not available'); // ✅ Graceful fallback
    }
  }
}
```

**Use case:** Optional features that may or may not be configured.

---

### Combining Decorators

You can combine `@SkipSelf()` and `@Optional()` together:

```typescript
@Component({...})
class ChildComponent {
  constructor(
    @Optional() @SkipSelf() private parentLogger: LoggerService | null
  ) {
    // Skip my own injector, look in parent
    // If parent doesn't have it, return null (no error)
  }
}
```

**Visual:**
```
Root (NO LoggerService) ❌
  ↓
ParentComponent (NO LoggerService) ❌
  ↓
ChildComponent → constructor(@Optional() @SkipSelf() logger)
                 ↓
            Skips ChildComponent
                 ↓
            Looks in ParentComponent ❌ (not found)
                 ↓
            Returns null ✅ (no error because of @Optional)
```

---

### Comparison Table

| Decorator | Search Scope | If Not Found | Use Case |
|-----------|-------------|--------------|----------|
| **None (default)** | Current → Parent → Root | ❌ Error | Normal dependency |
| **@Self()** | Current component ONLY | ❌ Error | Force own instance |
| **@SkipSelf()** | Parent → Root (skip current) | ❌ Error | Use parent's instance |
| **@Optional()** | Current → Parent → Root | ✅ Returns `null` | Optional feature |
| **@Optional() + @SkipSelf()** | Parent → Root (skip current) | ✅ Returns `null` | Optional parent service |

---

### Real-World Scenario

**Scenario:** Form controls need to access their parent form

```
AppComponent
  └── ParentFormComponent (provides FormService)
       └── ChildInputComponent → needs PARENT's FormService, not its own
```

**Code:**
```typescript
@Component({
  selector: 'app-parent-form',
  providers: [FormService] // Parent provides FormService
})
class ParentFormComponent {}

@Component({
  selector: 'app-child-input'
})
class ChildInputComponent {
  constructor(@SkipSelf() private formService: FormService) {
    // ✅ Gets parent's FormService
    // Even if ChildInput had its own provider, it would skip it
  }
}
```

---

### Using inject() function (modern way)

**Constructor way:**
```typescript
constructor(
  @Self() private serviceA: MyService,
  @Optional() @SkipSelf() private parentService: MyService | null
) {}
```

**inject() way:**
```typescript
private serviceA = inject(MyService, { self: true });
private parentService = inject(MyService, { optional: true, skipSelf: true });
```

Both work the same! Use `inject()` for cleaner, modern Angular code.

## 3. RxJS Operators: `takeUntil` vs `takeUntilDestroyed`
**Concept:** Managing subscription leaks.
- **`takeUntil(notifier$)`**: Emits values until the `notifier$` Observable emits. Commonly used with a `destroy$` Subject in `ngOnDestroy`.
- **`takeUntilDestroyed`** (Angular 16+): An operator that automatically completes the observable when the current context (component/directive) is destroyed. **⚠️ Requires injection context or passing `DestroyRef` - will throw runtime error if not available!**

**Example:**
```typescript
// Old Pattern - takeUntil (always works)
private destroy$ = new Subject<void>();
data$.pipe(takeUntil(this.destroy$)).subscribe();
ngOnDestroy() { this.destroy$.next(); }

// New Pattern (Angular 16+) - takeUntilDestroyed
data$.pipe(takeUntilDestroyed()).subscribe(); // ✅ Works in injection context
```

### ⚠️ Important: takeUntilDestroyed() Requires Injection Context!

**takeUntilDestroyed() works ONLY in these places:**
1. Inside constructor
2. In class property initializers
3. When you pass `DestroyRef` explicitly

**❌ Runtime Error Example:**
```typescript
@Component({...})
class MyComponent {
  ngOnInit() {
    // ❌ ERROR! Not in injection context
    this.data$.pipe(takeUntilDestroyed()).subscribe();
    // Error: NG0203 - inject() must be called from an injection context
  }
}
```

**✅ Correct Usage:**

**Option 1: Use in constructor or property initializer (injection context)**
```typescript
@Component({...})
class MyComponent {
  private data$ = this.http.get('/api/data');
  
  // ✅ Works - property initializer is injection context
  private subscription = this.data$.pipe(takeUntilDestroyed()).subscribe();
  
  constructor(private http: HttpClient) {
    // ✅ Works - constructor is injection context
    this.data$.pipe(takeUntilDestroyed()).subscribe();
  }
}
```

**Option 2: Pass DestroyRef explicitly**
```typescript
@Component({...})
class MyComponent {
  private destroyRef = inject(DestroyRef); // Get DestroyRef in injection context
  
  ngOnInit() {
    // ✅ Works - passing DestroyRef explicitly
    this.data$.pipe(takeUntilDestroyed(this.destroyRef)).subscribe();
  }
  
  loadData() {
    // ✅ Works - using stored DestroyRef
    this.http.get('/api/data')
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe();
  }
}
```

**Option 3: Use takeUntil (old pattern - works everywhere)**
```typescript
@Component({...})
class MyComponent {
  private destroy$ = new Subject<void>();
  
  ngOnInit() {
    // ✅ Works - no injection context needed
    this.data$.pipe(takeUntil(this.destroy$)).subscribe();
  }
  
  loadData() {
    // ✅ Works anywhere
    this.http.get('/api/data')
      .pipe(takeUntil(this.destroy$))
      .subscribe();
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

### Comparison: takeUntil vs takeUntilDestroyed

| Feature | **takeUntil** | **takeUntilDestroyed** |
|---------|---------------|------------------------|
| **Angular Version** | All versions | Angular 16+ |
| **Boilerplate** | More (need Subject + ngOnDestroy) | Less (automatic) |
| **Injection Context** | ❌ Not required | ⚠️ **Required** (or pass DestroyRef) |
| **Works in ngOnInit** | ✅ Yes | ❌ No (unless you pass DestroyRef) |
| **Works in methods** | ✅ Yes | ❌ No (unless you pass DestroyRef) |
| **Runtime Error Risk** | ❌ No | ⚠️ **Yes** (if no injection context) |
| **Use Case** | Any lifecycle hook or method | Constructor or property initializers |

### Summary

- **takeUntil:** Works everywhere, more boilerplate, no context issues
- **takeUntilDestroyed:** Less boilerplate BUT needs injection context or explicit DestroyRef
- **Best Practice:** Use `takeUntilDestroyed(this.destroyRef)` pattern to avoid runtime errors

## 4. `shareReplay`
**Concept:** A multicasting operator. It shares the underlying subscription with multiple subscribers and replays the last `N` emissions to new subscribers.
**Use Case:** Caching HTTP requests to avoid multiple network calls for the same data.

**Example:**
```typescript
this.users$ = this.http.get('/api/users').pipe(
  shareReplay(1) // Cache the last 1 value
);
```

## 5. Error Handling (Runtime - Angular)
**Concept:**
- **`ErrorHandler`**: Global error handler class in Angular. You can implement `ErrorHandler` to intercept all runtime errors (e.g., for logging to Sentry).
- **`HttpInterceptor`**: Best place to handle API errors globally (e.g., showing toast notifications on 404 or 500).

**Example (Global Handler):**
```typescript
@Injectable()
export class GlobalErrorHandler implements ErrorHandler {
  handleError(error: any) {
    console.error('An error occurred:', error);
    // Log to external service
  }
}

## 6. Reactive Forms: Dynamic Styling based on Value
**Concept:** You can subscribe to `valueChanges` of a form control to dynamically update styles or classes.

**Example:**
```typescript
// component.ts
this.myForm.get('status')?.valueChanges.subscribe(value => {
  this.statusColor = value === 'Active' ? 'green' : 'red';
});

// template.html
<p [style.color]="statusColor">Status: {{ myForm.get('status')?.value }}</p>
```

---

## Additional Basic Angular Questions

### Q: What is the use of Angular?
**Answer:** Angular is a TypeScript-based open-source web application framework for building dynamic single-page applications (SPAs). It provides:
- Component-based architecture
- Two-way data binding
- Dependency injection
- Built-in routing
- Form validation
- HTTP client for API communication
- CLI for scaffolding and building

### Q: Differentiate between AngularJS vs Angular?

| Feature | AngularJS (1.x) | Angular (2+) |
|---------|----------------|--------------|
| **Language** | JavaScript | TypeScript |
| **Architecture** | MVC (Controllers) | Component-based |
| **Mobile Support** | No | Yes (responsive) |
| **Performance** | Slower (digest cycle) | Faster (change detection) |
| **CLI** | No official CLI | Angular CLI |
| **Dependency Injection** | Limited | Hierarchical DI |

### Q: What are directives in Angular and what types do we have?
**Answer:** Directives are classes that add behavior to elements in Angular applications.

**Three types:**
1. **Component Directives** - Directives with templates (`@Component`)
2. **Structural Directives** - Change DOM structure (`*ngIf`, `*ngFor`, `*ngSwitch`)
3. **Attribute Directives** - Change appearance/behavior (`ngClass`, `ngStyle`, custom directives)

### Q: Explain the importance of NPM and Node_Modules folder?
**Answer:**
- **NPM (Node Package Manager)**: Package manager for JavaScript that manages project dependencies
- **node_modules folder**: Contains all installed packages/libraries
  - Generated by `npm install`
  - Should NOT be committed to version control (add to `.gitignore`)
  - Can be recreated using `package.json`

### Q: Explain the importance of Package.json file in Angular?
**Answer:** `package.json` is the project manifest file that contains:
- Project metadata (name, version, description)
- **Dependencies**: Production packages (`"dependencies"`)
- **DevDependencies**: Development-only packages (`"devDependencies"`)
- **Scripts**: Command shortcuts (`"start"`, `"build"`, `"test"`)
- Engine requirements (Node/NPM versions)

```json
{
  "name": "my-app",
  "scripts": {
    "start": "ng serve",
    "build": "ng build"
  },
  "dependencies": {
    "@angular/core": "^17.0.0"
  }
}
```

### Q: What is TypeScript and why do we need it?
**Answer:** TypeScript is a superset of JavaScript that adds static typing.

**Benefits:**
- **Type Safety**: Catch errors at compile-time
- **Better IDE Support**: Autocomplete, refactoring
- **Modern Features**: Interfaces, generics, decorators
- **Code Documentation**: Types serve as documentation
- **Easier Refactoring**: Compiler catches breaking changes

```typescript
// JavaScript (no type safety)
function add(a, b) { return a + b; }

// TypeScript (type safe)
function add(a: number, b: number): number { return a + b; }
```

### Q: Explain importance of Angular CLI?
**Answer:** Angular CLI (Command Line Interface) is a tool for:
- **Scaffolding**: Generate projects, components, services (`ng generate`)
- **Development Server**: `ng serve` with live reload
- **Building**: `ng build` for production optimization
- **Testing**: `ng test` for unit tests
- **Best Practices**: Enforces Angular coding standards

```bash
ng new my-app              # Create project
ng generate component user # Generate component
ng serve                   # Run dev server
ng build --configuration production  # Production build
```

### Q: What is a decorator in Angular?
**Answer:** Decorators are TypeScript functions that add metadata to classes, methods, properties, or parameters. They start with `@` symbol.

**Common Angular decorators:**
```typescript
@Component({...})      // Mark class as component
@Injectable()          // Mark class as injectable service
@Input()              // Mark property as input
@Output()             // Mark property as output
@ViewChild()          // Query for view element
```

### Q: What are Annotations or MetaData?
**Answer:** Metadata (via decorators) provides configuration information about classes:

```typescript
@Component({
  selector: 'app-user',      // Metadata: How to use component
  templateUrl: './user.html', // Metadata: Template location
  styleUrls: ['./user.css']   // Metadata: Styles location
})
export class UserComponent {}
```

Angular uses this metadata to understand how to process the class.

### Q: What is a template?
**Answer:** A template is the HTML view associated with a component. It defines what gets rendered on screen.

**Two ways to define:**
```typescript
// Inline template
@Component({
  template: `<h1>{{ title }}</h1>`
})

// External template file
@Component({
  templateUrl: './user.component.html'
})
```

Templates support:
- Data binding (`{{ }}`, `[]`, `()`, `[()]`)
- Directives (`*ngIf`, `*ngFor`)
- Pipes (`| date`, `| uppercase`)

### Q: Is a component a directive? Explain your thoughts.
**Answer:** **Yes**, a component IS a directive. 

- Component = Directive + Template
- Component is a special type of directive with a template
- All components are directives, but not all directives are components

```typescript
// Component (directive with template)
@Component({
  selector: 'app-user',
  template: '<h1>User</h1>'
})
export class UserComponent {}

// Attribute Directive (no template)
@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {}
```

### Q: Explain the four types of Data bindings in Angular?

| Type | Syntax | Direction | Example |
|------|--------|-----------|---------|
| **Interpolation** | `{{ }}` | Component → View | `<h1>{{ title }}</h1>` |
| **Property Binding** | `[property]` | Component → View | `<img [src]="imageUrl">` |
| **Event Binding** | `(event)` | View → Component | `<button (click)="save()">` |
| **Two-way Binding** | `[(ngModel)]` | Both directions | `<input [(ngModel)]="name">` |

### Q: What is SPA in Angular?
**Answer:** **SPA (Single Page Application)** is a web application that loads a single HTML page and dynamically updates content without full page reloads.

**Benefits:**
- Faster navigation (no full page reload)
- Better user experience
- Reduced server load
- Mobile app-like feel

**How Angular implements SPA:**
- Angular Router manages navigation
- Components are loaded/unloaded dynamically
- Only data is fetched from server (via APIs)

### Q: How to implement routing in Angular?
**Answer:**

**1. Define routes:**
```typescript
// app.routes.ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'users', component: UserListComponent },
  { path: 'users/:id', component: UserDetailComponent },
  { path: '**', component: NotFoundComponent }
];
```

**2. Provide router:**
```typescript
// app.config.ts
import { provideRouter } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes)]
};
```

**3. Add router outlet and links:**
```html
<nav>
  <a routerLink="/">Home</a>
  <a routerLink="/users">Users</a>
</nav>

<router-outlet></router-outlet>
```

**Note:** Section 7 (Nested API Calls) is available in the Scenario-Based file.

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


