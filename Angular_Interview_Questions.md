# Angular & RxJS Interview Questions

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

## 10. Lazy Loading & Bundle Splitting
**Concept:** Splitting the application into smaller bundles (chunks) that are loaded on demand (lazy loaded) rather than all at once. This improves initial load time.

**Example (Routes):**
```typescript
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule)
  }
];
```

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

## 12. Unknown in Angular (Strict Typing)
**Concept:** Using `unknown` instead of `any` enforces type checking. In Angular `HttpClient`, you can type the response as `unknown` and then validate it (e.g., using Zod or type guards) before using it, ensuring runtime safety.

**Example:**
```typescript
this.http.get<unknown>('/api/data').subscribe(data => {
  if (isValidData(data)) {
    // Safe to use
  }
});
```

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
