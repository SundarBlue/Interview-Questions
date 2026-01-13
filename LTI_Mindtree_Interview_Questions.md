# LTI Mindtree - BNY Mellon Front-End Developer Interview Questions
## Position: Senior Angular UI Developer (6+ Years Experience)

### Interview Focus Areas
- **Angular Development** (Primary Focus)
- **TypeScript** (Advanced)
- **JavaScript** (ES6+)
- **HTML5 & CSS3**
- **Leadership & Mentorship**
- **Financial Services Domain Knowledge**
- **Security & Performance Optimization**

---

## Angular Interview Questions (Advanced Level)

### 1. **Explain Angular Change Detection Strategy and when to use OnPush strategy?**

**Answer:**
Angular has two change detection strategies:

1. **Default Strategy:** Checks the entire component tree whenever any event occurs (clicks, HTTP requests, timers, etc.)
2. **OnPush Strategy:** Only checks the component when:
   - Input reference changes
   - Event originates from the component or its children
   - Manual change detection is triggered
   - Async pipe receives a new value

```typescript
@Component({
  selector: 'app-user-profile',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `...`
})
export class UserProfileComponent {
  @Input() user: User; // Only checked when reference changes
}
```

**When to use OnPush:**
- Performance-critical components
- Components with immutable data patterns
- Large lists or tables
- Leaf components in the component tree

### 2. **What is the difference between Subject, BehaviorSubject, ReplaySubject, and AsyncSubject in RxJS?**

**Answer:**

- **Subject:** No initial value, doesn't replay values to late subscribers
```typescript
const subject = new Subject<number>();
subject.next(1);
subject.subscribe(v => console.log(v)); // Won't receive 1
subject.next(2); // Receives 2
```

- **BehaviorSubject:** Requires initial value, replays the last value to new subscribers
```typescript
const behaviorSubject = new BehaviorSubject<number>(0);
behaviorSubject.next(1);
behaviorSubject.subscribe(v => console.log(v)); // Receives 1
```

- **ReplaySubject:** Replays specified number of previous values to new subscribers
```typescript
const replaySubject = new ReplaySubject<number>(2);
replaySubject.next(1);
replaySubject.next(2);
replaySubject.next(3);
replaySubject.subscribe(v => console.log(v)); // Receives 2, 3
```

- **AsyncSubject:** Only emits the last value when complete() is called
```typescript
const asyncSubject = new AsyncSubject<number>();
asyncSubject.next(1);
asyncSubject.next(2);
asyncSubject.subscribe(v => console.log(v)); // No value yet
asyncSubject.complete(); // Now receives 2
```

### 3. **Explain Angular Dependency Injection and different provider scopes?**

**Answer:**

Angular DI system provides dependencies where needed. Provider scopes determine service instance lifetime:

1. **Root Level (providedIn: 'root'):**
```typescript
@Injectable({ providedIn: 'root' })
export class UserService { }
// Single instance across entire app, tree-shakeable
```

2. **Module Level:**
```typescript
@NgModule({
  providers: [UserService]
})
// New instance per lazy-loaded module
```

3. **Component Level:**
```typescript
@Component({
  providers: [UserService]
})
// New instance per component instance
```

4. **Platform Level:** Shared across multiple applications

**Best Practice:** Use providedIn: 'root' for singleton services (most common case).

### 4. **What are Angular Guards and explain different types?**

**Answer:**

Guards control navigation and access to routes:

1. **CanActivate:** Determines if a route can be activated
```typescript
@Injectable({ providedIn: 'root' })
export class AuthGuard implements CanActivate {
  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<boolean> {
    return this.authService.isAuthenticated();
  }
}
```

2. **CanActivateChild:** Guards child routes
3. **CanDeactivate:** Prevents leaving a route (unsaved changes)
```typescript
export class UnsavedChangesGuard implements CanDeactivate<ComponentWithUnsavedChanges> {
  canDeactivate(component: ComponentWithUnsavedChanges): Observable<boolean> {
    return component.hasUnsavedChanges() 
      ? this.dialogService.confirm('Discard changes?')
      : of(true);
  }
}
```

4. **CanLoad:** Prevents lazy loading of modules
5. **Resolve:** Pre-fetches data before route activation

### 5. **Explain Angular Interceptors with real-world use cases?**

**Answer:**

Interceptors intercept HTTP requests/responses:

```typescript
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Clone and modify request
    const authReq = req.clone({
      headers: req.headers.set('Authorization', `Bearer ${this.getToken()}`)
    });
    
    return next.handle(authReq).pipe(
      catchError((error: HttpErrorResponse) => {
        if (error.status === 401) {
          // Handle unauthorized
          this.router.navigate(['/login']);
        }
        return throwError(() => error);
      })
    );
  }
}
```

**Real-world use cases:**
- Adding authentication tokens
- Error handling and logging
- Caching responses
- Request/response transformation
- Loading indicators
- Retry logic for failed requests

### 6. **What is lazy loading in Angular and how do you implement it?**

**Answer:**

Lazy loading loads feature modules on-demand rather than at startup, reducing initial bundle size.

```typescript
// app-routing.module.ts
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule),
    canLoad: [AuthGuard]
  },
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/dashboard.module').then(m => m.DashboardModule)
  }
];
```

**Benefits:**
- Faster initial load time
- Smaller initial bundle size
- Better performance for large applications
- Load modules based on user permissions

**Preloading Strategies:**
```typescript
// Preload all lazy modules after initial load
RouterModule.forRoot(routes, { preloadingStrategy: PreloadAllModules })
```

### 7. **Explain Angular Standalone Components and their benefits?**

**Answer:**

Standalone components (Angular 14+) don't require NgModule:

```typescript
@Component({
  selector: 'app-user-card',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule, MatCardModule],
  template: `...`
})
export class UserCardComponent { }
```

**Benefits:**
- Simplified Angular architecture
- Reduced boilerplate code
- Better tree-shaking
- Easier code reuse
- Clearer dependencies
- Simpler migration path

**Bootstrap standalone app:**
```typescript
// main.ts
bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes),
    provideHttpClient()
  ]
});
```

### 8. **What are Angular Signals and how do they improve reactivity?**

**Answer:**

Signals (Angular 16+) provide fine-grained reactivity:

```typescript
export class CounterComponent {
  count = signal(0);
  doubleCount = computed(() => this.count() * 2);
  
  increment() {
    this.count.update(value => value + 1);
  }
  
  constructor() {
    effect(() => {
      console.log(`Count changed to: ${this.count()}`);
    });
  }
}
```

**Benefits:**
- Better performance than Zone.js
- Fine-grained change detection
- Simpler mental model
- Better TypeScript inference
- Interoperable with RxJS

**Comparison with RxJS:**
- Signals: Synchronous, always have a value, simpler API
- RxJS: Asynchronous, powerful operators, complex scenarios

### 9. **Explain Angular State Management options and when to use each?**

**Answer:**

**1. Component State (Local):**
```typescript
export class UserComponent {
  users: User[] = [];
  loading = false;
}
```
Use for: Simple, component-specific state

**2. Service with BehaviorSubject:**
```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  private usersSubject = new BehaviorSubject<User[]>([]);
  users$ = this.usersSubject.asObservable();
  
  updateUsers(users: User[]) {
    this.usersSubject.next(users);
  }
}
```
Use for: Shared state between components

**3. NgRx (Redux Pattern):**
```typescript
// Actions
export const loadUsers = createAction('[User] Load Users');
export const loadUsersSuccess = createAction('[User] Load Users Success', props<{ users: User[] }>());

// Reducer
export const userReducer = createReducer(
  initialState,
  on(loadUsersSuccess, (state, { users }) => ({ ...state, users }))
);

// Selector
export const selectUsers = createSelector(
  selectUserState,
  (state) => state.users
);
```
Use for: Complex state, multiple data sources, time-travel debugging

**4. NgRx Component Store:**
Use for: Component-level complex state

**5. Akita / NGXS:**
Alternatives to NgRx with simpler APIs

### 10. **How do you optimize Angular application performance?**

**Answer:**

**1. OnPush Change Detection:**
```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
```

**2. TrackBy in ngFor:**
```typescript
<div *ngFor="let item of items; trackBy: trackById">{{ item.name }}</div>

trackById(index: number, item: Item): number {
  return item.id;
}
```

**3. Lazy Loading:**
Load modules on-demand

**4. Preloading Strategy:**
```typescript
RouterModule.forRoot(routes, { 
  preloadingStrategy: PreloadAllModules 
})
```

**5. Pure Pipes:**
```typescript
@Pipe({ name: 'filter', pure: true })
```

**6. Virtual Scrolling:**
```typescript
<cdk-virtual-scroll-viewport itemSize="50">
  <div *cdkVirtualFor="let item of items">{{ item }}</div>
</cdk-virtual-scroll-viewport>
```

**7. AOT Compilation:**
```bash
ng build --prod --aot
```

**8. Webpack Bundle Analyzer:**
```bash
npm install --save-dev webpack-bundle-analyzer
ng build --stats-json
```

**9. Unsubscribe from Observables:**
```typescript
private destroy$ = new Subject<void>();

ngOnInit() {
  this.service.getData()
    .pipe(takeUntil(this.destroy$))
    .subscribe();
}

ngOnDestroy() {
  this.destroy$.next();
  this.destroy$.complete();
}
```

**10. Use Web Workers for heavy computations**

### 11. **Explain Angular Forms: Template-driven vs Reactive Forms?**

**Answer:**

**Template-Driven Forms:**
```typescript
<form #userForm="ngForm" (ngSubmit)="onSubmit(userForm)">
  <input name="username" ngModel required />
  <input name="email" ngModel email />
</form>
```
**Pros:** Simple, less code
**Cons:** Hard to test, less control

**Reactive Forms:**
```typescript
export class UserComponent {
  userForm = this.fb.group({
    username: ['', [Validators.required, Validators.minLength(3)]],
    email: ['', [Validators.required, Validators.email]],
    address: this.fb.group({
      street: [''],
      city: ['']
    })
  });
  
  constructor(private fb: FormBuilder) {}
  
  onSubmit() {
    if (this.userForm.valid) {
      console.log(this.userForm.value);
    }
  }
}
```

**Reactive Forms Advantages:**
- Easier testing
- Type safety with Typed Forms (Angular 14+)
- Better control and validation
- Synchronous access to data model
- Easier to create dynamic forms

**Custom Validators:**
```typescript
export function passwordMatch(): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const password = control.get('password');
    const confirm = control.get('confirmPassword');
    return password && confirm && password.value !== confirm.value
      ? { passwordMismatch: true }
      : null;
  };
}
```

### 12. **What are Angular Pipes and how do you create custom pipes?**

**Answer:**

Pipes transform data in templates:

**Built-in pipes:**
```typescript
{{ date | date:'dd/MM/yyyy' }}
{{ price | currency:'USD' }}
{{ text | uppercase }}
{{ data | json }}
{{ value | async }}
```

**Custom Pipe:**
```typescript
@Pipe({ name: 'truncate', pure: true })
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit: number = 50, trail: string = '...'): string {
    return value.length > limit 
      ? value.substring(0, limit) + trail 
      : value;
  }
}

// Usage
{{ longText | truncate:100:'...' }}
```

**Pure vs Impure Pipes:**
- **Pure (default):** Only executes when input reference changes
- **Impure:** Executes on every change detection cycle

```typescript
@Pipe({ name: 'filter', pure: false })
export class FilterPipe implements PipeTransform {
  transform(items: any[], searchText: string): any[] {
    return items.filter(item => item.name.includes(searchText));
  }
}
```

**Async Pipe:**
Automatically subscribes and unsubscribes from Observables:
```typescript
users$ = this.userService.getUsers();

// Template
<div *ngFor="let user of users$ | async">{{ user.name }}</div>
```

### 13. **Explain Angular Content Projection and ViewChild vs ContentChild?**

**Answer:**

**Content Projection (ng-content):**
```typescript
// card.component.ts
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <div class="header">
        <ng-content select="[header]"></ng-content>
      </div>
      <div class="body">
        <ng-content></ng-content>
      </div>
      <div class="footer">
        <ng-content select="[footer]"></ng-content>
      </div>
    </div>
  `
})

// Usage
<app-card>
  <h2 header>Title</h2>
  <p>Content goes here</p>
  <button footer>Action</button>
</app-card>
```

**ViewChild:** Access child components in the view template
```typescript
@Component({
  template: '<app-child #child></app-child>'
})
export class ParentComponent {
  @ViewChild('child') childComponent!: ChildComponent;
  
  ngAfterViewInit() {
    this.childComponent.doSomething();
  }
}
```

**ContentChild:** Access projected content
```typescript
@Component({
  selector: 'app-card',
  template: '<ng-content></ng-content>'
})
export class CardComponent {
  @ContentChild(HeaderComponent) header!: HeaderComponent;
  
  ngAfterContentInit() {
    this.header.highlight();
  }
}
```

**Key Differences:**
- ViewChild: Direct children in component template
- ContentChild: Projected content from parent

### 14. **How do you handle errors globally in Angular?**

**Answer:**

**1. Global Error Handler:**
```typescript
@Injectable()
export class GlobalErrorHandler implements ErrorHandler {
  constructor(
    private injector: Injector,
    private notificationService: NotificationService
  ) {}
  
  handleError(error: Error | HttpErrorResponse) {
    const router = this.injector.get(Router);
    
    if (error instanceof HttpErrorResponse) {
      // HTTP errors
      if (!navigator.onLine) {
        this.notificationService.error('No internet connection');
      } else {
        this.notificationService.error(`Server error: ${error.status}`);
      }
    } else {
      // Client-side errors
      console.error('Client error:', error);
      this.notificationService.error('An unexpected error occurred');
    }
    
    // Log to monitoring service
    this.logError(error);
  }
  
  private logError(error: any) {
    // Send to logging service (e.g., Sentry, LogRocket)
  }
}

// app.module.ts
providers: [
  { provide: ErrorHandler, useClass: GlobalErrorHandler }
]
```

**2. HTTP Interceptor for HTTP Errors:**
```typescript
@Injectable()
export class ErrorInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      retry(2), // Retry failed requests
      catchError((error: HttpErrorResponse) => {
        let errorMessage = '';
        
        if (error.error instanceof ErrorEvent) {
          // Client-side error
          errorMessage = `Client Error: ${error.error.message}`;
        } else {
          // Server-side error
          errorMessage = `Server Error Code: ${error.status}\nMessage: ${error.message}`;
        }
        
        return throwError(() => new Error(errorMessage));
      })
    );
  }
}
```

### 15. **Explain Angular Module Federation and Micro-frontends?**

**Answer:**

Module Federation allows loading separately compiled applications at runtime:

**webpack.config.js:**
```javascript
const ModuleFederationPlugin = require("webpack/lib/container/ModuleFederationPlugin");

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: "app1",
      filename: "remoteEntry.js",
      exposes: {
        './Component': './src/app/my-component/my-component.component.ts',
      },
      shared: {
        "@angular/core": { singleton: true, strictVersion: true },
        "@angular/common": { singleton: true, strictVersion: true },
      }
    })
  ]
};
```

**Consuming Application:**
```javascript
new ModuleFederationPlugin({
  name: "shell",
  remotes: {
    app1: "app1@http://localhost:3001/remoteEntry.js",
  },
  shared: {
    "@angular/core": { singleton: true, strictVersion: true },
  }
})
```

**Benefits:**
- Independent deployment
- Team autonomy
- Technology diversity
- Incremental migration
- Reduced bundle size

**Challenges:**
- Shared dependencies management
- Versioning complexity
- Testing across microfrontends
- Communication between apps

---

## TypeScript Interview Questions (Advanced Level)

### 1. **Explain TypeScript Generics with real-world examples?**

**Answer:**

Generics create reusable components that work with multiple types:

**Basic Generic Function:**
```typescript
function identity<T>(arg: T): T {
  return arg;
}

const num = identity<number>(42);
const str = identity<string>("hello");
```

**Generic Interface:**
```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

interface User {
  id: number;
  name: string;
}

const userResponse: ApiResponse<User> = {
  data: { id: 1, name: "John" },
  status: 200,
  message: "Success"
};
```

**Generic Class:**
```typescript
class DataStore<T> {
  private data: T[] = [];
  
  add(item: T): void {
    this.data.push(item);
  }
  
  get(index: number): T {
    return this.data[index];
  }
  
  getAll(): T[] {
    return this.data;
  }
}

const userStore = new DataStore<User>();
userStore.add({ id: 1, name: "John" });
```

**Generic Constraints:**
```typescript
interface HasId {
  id: number;
}

function findById<T extends HasId>(items: T[], id: number): T | undefined {
  return items.find(item => item.id === id);
}
```

### 2. **What are TypeScript Utility Types?**

**Answer:**

**Partial<T>:** Makes all properties optional
```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

function updateUser(user: User, updates: Partial<User>): User {
  return { ...user, ...updates };
}

updateUser(user, { name: "New Name" }); // Valid
```

**Required<T>:** Makes all properties required
```typescript
type RequiredUser = Required<Partial<User>>;
```

**Readonly<T>:** Makes all properties readonly
```typescript
const config: Readonly<Config> = {
  apiUrl: "https://api.example.com"
};
// config.apiUrl = "new"; // Error
```

**Pick<T, K>:** Select specific properties
```typescript
type UserPreview = Pick<User, 'id' | 'name'>;
// { id: number; name: string; }
```

**Omit<T, K>:** Exclude specific properties
```typescript
type UserWithoutEmail = Omit<User, 'email'>;
// { id: number; name: string; }
```

**Record<K, T>:** Create object type with specific keys
```typescript
type UserRoles = Record<string, string[]>;
const roles: UserRoles = {
  admin: ['read', 'write', 'delete'],
  user: ['read']
};
```

**ReturnType<T>:** Extract return type of function
```typescript
function getUser() {
  return { id: 1, name: "John" };
}

type User = ReturnType<typeof getUser>;
// { id: number; name: string; }
```

### 3. **Explain TypeScript Decorators and their use in Angular?**

**Answer:**

Decorators are special declarations that can modify classes, methods, properties, or parameters.

**Class Decorator:**
```typescript
function Component(config: any) {
  return function(target: any) {
    // Modify the class
    target.prototype.selector = config.selector;
    target.prototype.template = config.template;
  };
}

@Component({
  selector: 'app-user',
  template: '<div>User</div>'
})
class UserComponent {}
```

**Method Decorator:**
```typescript
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Result:`, result);
    return result;
  };
  
  return descriptor;
}

class Calculator {
  @Log
  add(a: number, b: number): number {
    return a + b;
  }
}
```

**Property Decorator:**
```typescript
function Required(target: any, propertyKey: string) {
  let value: any;
  
  const getter = () => value;
  const setter = (newVal: any) => {
    if (!newVal) {
      throw new Error(`${propertyKey} is required`);
    }
    value = newVal;
  };
  
  Object.defineProperty(target, propertyKey, {
    get: getter,
    set: setter
  });
}
```

**Angular Decorators:**
- `@Component`: Define component metadata
- `@Injectable`: Mark class for dependency injection
- `@Input`: Pass data to child component
- `@Output`: Emit events to parent
- `@ViewChild`: Access child element/component
- `@HostListener`: Listen to host events

### 4. **What is the difference between Interface and Type in TypeScript?**

**Answer:**

**Interfaces:**
```typescript
interface User {
  id: number;
  name: string;
}

// Can extend
interface Admin extends User {
  permissions: string[];
}

// Declaration merging
interface User {
  email: string; // Merged with above
}
```

**Types:**
```typescript
type User = {
  id: number;
  name: string;
};

// Can use union types
type ID = string | number;

// Can use intersection
type Admin = User & {
  permissions: string[];
};

// Can use mapped types
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

// Can use conditional types
type IsString<T> = T extends string ? true : false;
```

**Key Differences:**
- Interfaces can be extended and merged, types cannot be merged
- Types can use unions, tuples, and advanced types
- Interfaces are better for object shapes
- Types are more flexible for complex type operations

**Best Practice:**
- Use interfaces for object shapes and class contracts
- Use types for unions, intersections, and utility types

### 5. **Explain Type Guards and Narrowing in TypeScript?**

**Answer:**

Type guards help TypeScript narrow down types:

**typeof Type Guard:**
```typescript
function printValue(value: string | number) {
  if (typeof value === "string") {
    console.log(value.toUpperCase()); // TypeScript knows it's string
  } else {
    console.log(value.toFixed(2)); // TypeScript knows it's number
  }
}
```

**instanceof Type Guard:**
```typescript
class Dog {
  bark() { console.log("Woof!"); }
}

class Cat {
  meow() { console.log("Meow!"); }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark(); // TypeScript knows it's Dog
  } else {
    animal.meow(); // TypeScript knows it's Cat
  }
}
```

**Custom Type Guard:**
```typescript
interface Fish {
  swim: () => void;
}

interface Bird {
  fly: () => void;
}

function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}

function move(pet: Fish | Bird) {
  if (isFish(pet)) {
    pet.swim(); // TypeScript knows it's Fish
  } else {
    pet.fly(); // TypeScript knows it's Bird
  }
}
```

**in Operator:**
```typescript
type Admin = { role: "admin"; permissions: string[] };
type User = { role: "user"; email: string };

function handleUser(user: Admin | User) {
  if ("permissions" in user) {
    console.log(user.permissions); // Admin
  } else {
    console.log(user.email); // User
  }
}
```

**Discriminated Unions:**
```typescript
interface SuccessResponse {
  status: "success";
  data: any;
}

interface ErrorResponse {
  status: "error";
  message: string;
}

type ApiResponse = SuccessResponse | ErrorResponse;

function handleResponse(response: ApiResponse) {
  switch (response.status) {
    case "success":
      console.log(response.data);
      break;
    case "error":
      console.log(response.message);
      break;
  }
}
```

### 6. **Explain Advanced TypeScript Types: Mapped Types, Conditional Types?**

**Answer:**

**Mapped Types:**
```typescript
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Optional<T> = {
  [P in keyof T]?: T[P];
};

type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

interface User {
  id: number;
  name: string;
}

type ReadonlyUser = Readonly<User>;
// { readonly id: number; readonly name: string; }
```

**Conditional Types:**
```typescript
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false

// Extract function return type
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

// Remove null and undefined
type NonNullable<T> = T extends null | undefined ? never : T;
```

**Template Literal Types:**
```typescript
type EventName<T extends string> = `on${Capitalize<T>}`;

type ClickEvent = EventName<"click">; // "onClick"
type MouseEvent = EventName<"mouseOver">; // "onMouseOver"
```

**Recursive Types:**
```typescript
type JsonValue = 
  | string
  | number
  | boolean
  | null
  | JsonValue[]
  | { [key: string]: JsonValue };

const data: JsonValue = {
  name: "John",
  age: 30,
  hobbies: ["reading", "coding"],
  address: {
    city: "New York",
    coordinates: [40.7128, -74.0060]
  }
};
```

### 7. **What are TypeScript Enums and their alternatives?**

**Answer:**

**Numeric Enums:**
```typescript
enum Status {
  Pending,      // 0
  Active,       // 1
  Inactive,     // 2
  Completed     // 3
}

const status: Status = Status.Active;
```

**String Enums:**
```typescript
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT"
}
```

**Const Enums:** (More performant, no runtime object)
```typescript
const enum HttpStatus {
  OK = 200,
  BadRequest = 400,
  Unauthorized = 401,
  NotFound = 404
}

// Compiled to: if (status === 200)
if (status === HttpStatus.OK) { }
```

**Alternatives (Union Types - Recommended):**
```typescript
type Status = "pending" | "active" | "inactive" | "completed";

const status: Status = "active"; // Type-safe and lightweight
```

**Const Assertions:**
```typescript
const Direction = {
  Up: "UP",
  Down: "DOWN",
  Left: "LEFT",
  Right: "RIGHT"
} as const;

type Direction = typeof Direction[keyof typeof Direction];
// "UP" | "DOWN" | "LEFT" | "RIGHT"
```

### 8. **Explain TypeScript's `never`, `unknown`, and `any` types?**

**Answer:**

**any:** Opts out of type checking (avoid when possible)
```typescript
let value: any = "hello";
value = 42;
value.anyMethod(); // No type checking
```

**unknown:** Type-safe alternative to any
```typescript
let value: unknown = "hello";

// Must narrow type before use
if (typeof value === "string") {
  console.log(value.toUpperCase()); // OK
}

// value.toUpperCase(); // Error: must narrow first
```

**never:** Represents values that never occur
```typescript
// Function that never returns
function throwError(message: string): never {
  throw new Error(message);
}

// Exhaustive type checking
type Shape = Circle | Square;

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.size ** 2;
    default:
      // If we add a new shape type, this will error
      const _exhaustive: never = shape;
      return _exhaustive;
  }
}
```

**Use Cases:**
- `any`: Quick prototyping, gradual migration (use sparingly)
- `unknown`: When type is truly unknown (API responses)
- `never`: Exhaustiveness checking, unreachable code

### 9. **What is TypeScript's `infer` keyword?**

**Answer:**

`infer` is used in conditional types to extract and name types:

**Extract Return Type:**
```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() {
  return { id: 1, name: "John" };
}

type User = ReturnType<typeof getUser>;
// { id: number; name: string; }
```

**Extract Parameter Types:**
```typescript
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

function createUser(name: string, age: number) {}

type Params = Parameters<typeof createUser>;
// [string, number]
```

**Extract Array Element Type:**
```typescript
type ElementType<T> = T extends (infer U)[] ? U : T;

type StringArray = ElementType<string[]>; // string
type NumberType = ElementType<number>; // number
```

**Extract Promise Type:**
```typescript
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type User = UnwrapPromise<Promise<{ id: number }>>; // { id: number }
```

**Practical Example:**
```typescript
type ApiResponse<T> = {
  data: T;
  status: number;
};

type ExtractData<T> = T extends ApiResponse<infer U> ? U : never;

type UserData = ExtractData<ApiResponse<User>>; // User
```

### 10. **Explain TypeScript's Type Assertion vs Type Casting?**

**Answer:**

**Type Assertion (TypeScript only):**
Tells the compiler to treat a value as a specific type:

```typescript
// Angle-bracket syntax
let value: any = "hello";
let length: number = (<string>value).length;

// as syntax (preferred in JSX/TSX)
let length2: number = (value as string).length;
```

**Common Use Cases:**
```typescript
// DOM manipulation
const input = document.getElementById('username') as HTMLInputElement;
input.value = "John";

// API responses
const response = await fetch('/api/user');
const user = await response.json() as User;

// Non-null assertion
function getValue(id?: string) {
  return id!.toUpperCase(); // Assert id is not null/undefined
}
```

**const Assertions:**
```typescript
// Without const assertion
const config = {
  url: "https://api.example.com",
  timeout: 5000
};
// { url: string; timeout: number; }

// With const assertion
const config = {
  url: "https://api.example.com",
  timeout: 5000
} as const;
// { readonly url: "https://api.example.com"; readonly timeout: 5000; }
```

**Type Casting vs Assertion:**
TypeScript has no runtime type casting. Assertions are compile-time only:

```typescript
let value: any = 42;
let str = value as string; // No runtime conversion
console.log(typeof str); // "number" (not "string")

// For actual conversion, use JavaScript:
let actualStr = String(value);
```

**Best Practices:**
- Use type guards instead of assertions when possible
- Avoid `any` and assertions in favor of proper typing
- Use non-null assertion (!) only when absolutely certain

---

## JavaScript Interview Questions (Advanced Level)

### 1. **Explain JavaScript Event Loop, Call Stack, and Task Queue?**

**Answer:**

JavaScript is single-threaded but handles async operations through the Event Loop:

**Components:**
1. **Call Stack:** Executes synchronous code (LIFO)
2. **Web APIs:** Handle async operations (setTimeout, HTTP, DOM events)
3. **Task Queue (Macrotask):** Callbacks from setTimeout, setInterval, I/O
4. **Microtask Queue:** Promises, MutationObserver (higher priority)
5. **Event Loop:** Monitors and moves tasks to call stack

```javascript
console.log('1'); // Call stack

setTimeout(() => {
  console.log('2'); // Task queue (macrotask)
}, 0);

Promise.resolve().then(() => {
  console.log('3'); // Microtask queue
});

console.log('4'); // Call stack

// Output: 1, 4, 3, 2
```

**Execution Order:**
1. Execute all synchronous code
2. Execute all microtasks
3. Execute one macrotask
4. Repeat from step 2

**Real Example:**
```javascript
console.log('Start');

setTimeout(() => console.log('Timeout 1'), 0);

Promise.resolve()
  .then(() => console.log('Promise 1'))
  .then(() => console.log('Promise 2'));

setTimeout(() => console.log('Timeout 2'), 0);

console.log('End');

// Output:
// Start
// End
// Promise 1
// Promise 2
// Timeout 1
// Timeout 2
```

### 2. **Explain Closures and their practical applications?**

**Answer:**

A closure is a function that has access to variables in its outer lexical scope, even after the outer function has returned.

**Basic Example:**
```javascript
function createCounter() {
  let count = 0;
  
  return {
    increment() {
      return ++count;
    },
    decrement() {
      return --count;
    },
    getCount() {
      return count;
    }
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.getCount());  // 2
// count is private, cannot access directly
```

**Practical Applications:**

**1. Data Privacy:**
```javascript
function bankAccount(initialBalance) {
  let balance = initialBalance;
  
  return {
    deposit(amount) {
      balance += amount;
      return balance;
    },
    withdraw(amount) {
      if (amount <= balance) {
        balance -= amount;
        return balance;
      }
      throw new Error('Insufficient funds');
    },
    getBalance() {
      return balance;
    }
  };
}

const account = bankAccount(1000);
account.deposit(500);   // 1500
account.withdraw(200);  // 1300
// balance is private
```

**2. Function Factories:**
```javascript
function createMultiplier(multiplier) {
  return function(value) {
    return value * multiplier;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

**3. Memoization:**
```javascript
function memoize(fn) {
  const cache = {};
  
  return function(...args) {
    const key = JSON.stringify(args);
    if (key in cache) {
      return cache[key];
    }
    const result = fn.apply(this, args);
    cache[key] = result;
    return result;
  };
}

const expensiveCalculation = memoize((n) => {
  console.log('Calculating...');
  return n * n;
});

expensiveCalculation(5); // Logs "Calculating..." returns 25
expensiveCalculation(5); // Returns 25 (from cache)
```

### 3. **Explain Promises, async/await, and error handling?**

**Answer:**

**Promises:**
```javascript
const fetchUser = (id) => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (id > 0) {
        resolve({ id, name: 'John' });
      } else {
        reject(new Error('Invalid ID'));
      }
    }, 1000);
  });
};

fetchUser(1)
  .then(user => console.log(user))
  .catch(error => console.error(error))
  .finally(() => console.log('Done'));
```

**Promise Methods:**
```javascript
// Promise.all - All must succeed
Promise.all([promise1, promise2, promise3])
  .then(results => console.log(results))
  .catch(error => console.error('One failed:', error));

// Promise.allSettled - Wait for all regardless of outcome
Promise.allSettled([promise1, promise2])
  .then(results => {
    results.forEach(result => {
      if (result.status === 'fulfilled') {
        console.log('Success:', result.value);
      } else {
        console.log('Failed:', result.reason);
      }
    });
  });

// Promise.race - First to complete wins
Promise.race([promise1, promise2])
  .then(result => console.log('First:', result));

// Promise.any - First to succeed wins
Promise.any([promise1, promise2])
  .then(result => console.log('First success:', result))
  .catch(error => console.log('All failed'));
```

**async/await:**
```javascript
async function fetchUserData(userId) {
  try {
    const user = await fetchUser(userId);
    const posts = await fetchUserPosts(user.id);
    const comments = await fetchPostComments(posts[0].id);
    
    return { user, posts, comments };
  } catch (error) {
    console.error('Error:', error.message);
    throw error;
  } finally {
    console.log('Cleanup');
  }
}

// Parallel execution with async/await
async function fetchMultipleUsers(ids) {
  const promises = ids.map(id => fetchUser(id));
  const users = await Promise.all(promises);
  return users;
}
```

**Error Handling Patterns:**
```javascript
// Try-catch with async/await
async function handleRequest() {
  try {
    const data = await fetchData();
    return data;
  } catch (error) {
    if (error.status === 404) {
      return null;
    }
    throw error; // Re-throw if not handled
  }
}

// Promise chain error handling
fetchData()
  .then(data => processData(data))
  .then(result => saveResult(result))
  .catch(error => {
    // Handles errors from any step
    console.error('Pipeline error:', error);
  });
```

### 4. **Explain Prototypal Inheritance in JavaScript?**

**Answer:**

JavaScript uses prototypal inheritance where objects inherit from other objects:

**Prototype Chain:**
```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.greet = function() {
  return `Hello, I'm ${this.name}`;
};

const john = new Person('John', 30);
console.log(john.greet()); // Hello, I'm John
console.log(john.__proto__ === Person.prototype); // true
```

**ES6 Classes (Syntactic Sugar):**
```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  greet() {
    return `Hello, I'm ${this.name}`;
  }
  
  static species() {
    return 'Homo Sapiens';
  }
}

class Employee extends Person {
  constructor(name, age, jobTitle) {
    super(name, age);
    this.jobTitle = jobTitle;
  }
  
  greet() {
    return `${super.greet()}, I'm a ${this.jobTitle}`;
  }
}

const emp = new Employee('Alice', 28, 'Developer');
console.log(emp.greet()); // Hello, I'm Alice, I'm a Developer
```

**Object.create():**
```javascript
const personProto = {
  greet() {
    return `Hello, I'm ${this.name}`;
  }
};

const john = Object.create(personProto);
john.name = 'John';
john.age = 30;

console.log(john.greet()); // Hello, I'm John
```

**Prototype Methods:**
```javascript
// Check prototype
Object.getPrototypeOf(john) === personProto; // true

// Set prototype
Object.setPrototypeOf(john, anotherProto);

// Check if property exists on object (not prototype)
john.hasOwnProperty('name'); // true
john.hasOwnProperty('greet'); // false

// Check if object is in prototype chain
personProto.isPrototypeOf(john); // true
```

### 5. **Explain `this` keyword and binding in JavaScript?**

**Answer:**

The value of `this` depends on how a function is called:

**1. Global Context:**
```javascript
console.log(this); // Window (browser) or global (Node.js)

function regularFunction() {
  console.log(this); // Window in non-strict mode, undefined in strict mode
}
```

**2. Object Method:**
```javascript
const obj = {
  name: 'John',
  greet() {
    console.log(this.name); // 'John'
  }
};

obj.greet(); // 'John'

const greetFn = obj.greet;
greetFn(); // undefined (lost context)
```

**3. Constructor:**
```javascript
function Person(name) {
  this.name = name; // this refers to new instance
}

const john = new Person('John');
```

**4. Arrow Functions:**
```javascript
const obj = {
  name: 'John',
  greet: () => {
    console.log(this.name); // undefined (lexical this from outer scope)
  },
  delayedGreet() {
    setTimeout(() => {
      console.log(this.name); // 'John' (arrow function captures this)
    }, 1000);
  }
};
```

**Explicit Binding:**

**call():**
```javascript
function greet(greeting, punctuation) {
  console.log(`${greeting}, I'm ${this.name}${punctuation}`);
}

const person = { name: 'John' };
greet.call(person, 'Hello', '!'); // Hello, I'm John!
```

**apply():**
```javascript
greet.apply(person, ['Hi', '.']); // Hi, I'm John.
```

**bind():**
```javascript
const boundGreet = greet.bind(person);
boundGreet('Hey', '!'); // Hey, I'm John!

// Partial application
const boundHello = greet.bind(person, 'Hello');
boundHello('!!!'); // Hello, I'm John!!!
```

**React Component Example:**
```javascript
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    // Bind in constructor
    this.handleClick = this.handleClick.bind(this);
  }
  
  handleClick() {
    console.log(this.props); // Works
  }
  
  // Alternative: Arrow function property
  handleClick2 = () => {
    console.log(this.props); // Works
  }
}
```

### 6. **Explain ES6+ features: Destructuring, Spread, Rest?**

**Answer:**

**Destructuring:**
```javascript
// Array destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];
console.log(first); // 1
console.log(rest);  // [3, 4, 5]

// Object destructuring
const user = { name: 'John', age: 30, city: 'NYC' };
const { name, age, city = 'Unknown' } = user;

// Nested destructuring
const { address: { street, city } } = user;

// Rename variables
const { name: userName, age: userAge } = user;

// Function parameters
function greet({ name, age }) {
  console.log(`Hello ${name}, you are ${age}`);
}
```

**Spread Operator:**
```javascript
// Array spread
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]

// Object spread
const user = { name: 'John', age: 30 };
const updatedUser = { ...user, age: 31, city: 'NYC' };

// Function arguments
const numbers = [1, 2, 3];
Math.max(...numbers); // 3

// Copy arrays/objects (shallow)
const copy = [...arr1];
const objCopy = { ...user };
```

**Rest Parameters:**
```javascript
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

sum(1, 2, 3, 4); // 10

// With other parameters
function greet(greeting, ...names) {
  return `${greeting} ${names.join(', ')}!`;
}

greet('Hello', 'John', 'Jane', 'Bob'); // Hello John, Jane, Bob!
```

**Practical Examples:**
```javascript
// Swapping variables
let a = 1, b = 2;
[a, b] = [b, a];

// Removing properties
const { password, ...userWithoutPassword } = user;

// Merging objects
const defaults = { theme: 'light', lang: 'en' };
const userPrefs = { theme: 'dark' };
const config = { ...defaults, ...userPrefs }; // { theme: 'dark', lang: 'en' }

// Default parameters with destructuring
function createUser({ 
  name, 
  age = 18, 
  role = 'user' 
} = {}) {
  return { name, age, role };
}
```

### 7. **Explain JavaScript Modules (ES6 import/export)?**

**Answer:**

**Named Exports:**
```javascript
// utils.js
export const PI = 3.14159;

export function square(x) {
  return x * x;
}

export class Calculator {
  add(a, b) {
    return a + b;
  }
}

// Import
import { PI, square, Calculator } from './utils.js';
import { square as sq } from './utils.js'; // Rename
import * as Utils from './utils.js'; // Import all
```

**Default Export:**
```javascript
// user.js
export default class User {
  constructor(name) {
    this.name = name;
  }
}

// Import
import User from './user.js';
import MyUser from './user.js'; // Can rename default import
```

**Mixed Exports:**
```javascript
// api.js
export const API_URL = 'https://api.example.com';

export function get(endpoint) {
  return fetch(`${API_URL}${endpoint}`);
}

export default {
  get,
  post: (endpoint, data) => fetch(`${API_URL}${endpoint}`, {
    method: 'POST',
    body: JSON.stringify(data)
  })
};

// Import
import api, { API_URL, get } from './api.js';
```

**Dynamic Imports:**
```javascript
// Lazy loading
async function loadModule() {
  const module = await import('./heavy-module.js');
  module.doSomething();
}

// Conditional loading
if (condition) {
  const { feature } = await import('./feature.js');
  feature();
}

// Code splitting in bundlers
button.addEventListener('click', async () => {
  const { default: Chart } = await import('./chart.js');
  new Chart(data);
});
```

**Re-exporting:**
```javascript
// index.js - Barrel export
export { User } from './user.js';
export { Product } from './product.js';
export * from './constants.js';

// Usage
import { User, Product, API_URL } from './index.js';
```

### 8. **Explain Array and Object methods (map, filter, reduce, etc.)?**

**Answer:**

**Array.map():** Transform elements
```javascript
const numbers = [1, 2, 3, 4];
const doubled = numbers.map(n => n * 2); // [2, 4, 6, 8]

const users = [{ name: 'John', age: 30 }];
const names = users.map(user => user.name);
```

**Array.filter():** Select elements
```javascript
const numbers = [1, 2, 3, 4, 5];
const evens = numbers.filter(n => n % 2 === 0); // [2, 4]

const adults = users.filter(user => user.age >= 18);
```

**Array.reduce():** Accumulate values
```javascript
const numbers = [1, 2, 3, 4];
const sum = numbers.reduce((acc, n) => acc + n, 0); // 10

// Group by
const grouped = users.reduce((acc, user) => {
  const key = user.age >= 18 ? 'adults' : 'minors';
  acc[key] = acc[key] || [];
  acc[key].push(user);
  return acc;
}, {});

// Count occurrences
const fruits = ['apple', 'banana', 'apple'];
const count = fruits.reduce((acc, fruit) => {
  acc[fruit] = (acc[fruit] || 0) + 1;
  return acc;
}, {});
```

**Array.find() / findIndex():**
```javascript
const users = [{ id: 1, name: 'John' }, { id: 2, name: 'Jane' }];
const user = users.find(u => u.id === 2); // { id: 2, name: 'Jane' }
const index = users.findIndex(u => u.id === 2); // 1
```

**Array.some() / every():**
```javascript
const numbers = [1, 2, 3, 4];
const hasEven = numbers.some(n => n % 2 === 0); // true
const allEven = numbers.every(n => n % 2 === 0); // false
```

**Array.flat() / flatMap():**
```javascript
const nested = [1, [2, [3, 4]]];
nested.flat(); // [1, 2, [3, 4]]
nested.flat(2); // [1, 2, 3, 4]
nested.flat(Infinity); // [1, 2, 3, 4]

const users = [{ id: 1, hobbies: ['reading', 'coding'] }];
const hobbies = users.flatMap(u => u.hobbies);
```

**Object methods:**
```javascript
const obj = { a: 1, b: 2, c: 3 };

Object.keys(obj); // ['a', 'b', 'c']
Object.values(obj); // [1, 2, 3]
Object.entries(obj); // [['a', 1], ['b', 2], ['c', 3]]

// Object.fromEntries
const entries = [['a', 1], ['b', 2]];
Object.fromEntries(entries); // { a: 1, b: 2 }

// Clone and merge
const clone = Object.assign({}, obj);
const merged = Object.assign({}, obj1, obj2);
```

**Chaining:**
```javascript
const result = users
  .filter(u => u.age >= 18)
  .map(u => ({ ...u, adult: true }))
  .sort((a, b) => a.age - b.age)
  .slice(0, 10);
```

### 9. **Explain Debouncing and Throttling?**

**Answer:**

Both are techniques to limit function execution rate:

**Debouncing:** Execute after quiet period
```javascript
function debounce(func, delay) {
  let timeoutId;
  
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
}

// Usage: Search input
const searchInput = document.getElementById('search');
const handleSearch = debounce((value) => {
  console.log('Searching for:', value);
  // API call
}, 500);

searchInput.addEventListener('input', (e) => {
  handleSearch(e.target.value);
});
```

**Throttling:** Execute at most once per interval
```javascript
function throttle(func, limit) {
  let inThrottle;
  
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}

// Usage: Scroll handler
const handleScroll = throttle(() => {
  console.log('Scroll position:', window.scrollY);
}, 1000);

window.addEventListener('scroll', handleScroll);
```

**When to use:**
- **Debounce:** Search inputs, window resize, keystroke events
- **Throttle:** Scroll events, mouse movement, button clicks

**Lodash versions:**
```javascript
import { debounce, throttle } from 'lodash';

const debouncedSearch = debounce(search, 500);
const throttledScroll = throttle(handleScroll, 1000);
```

### 10. **Explain Memory Leaks in JavaScript and how to prevent them?**

**Answer:**

Memory leaks occur when memory is no longer needed but not released:

**Common Causes:**

**1. Global Variables:**
```javascript
// Bad
function createUser() {
  user = { name: 'John' }; // Implicit global (no var/let/const)
}

// Good
function createUser() {
  const user = { name: 'John' };
}
```

**2. Event Listeners:**
```javascript
// Bad - listener never removed
element.addEventListener('click', handler);

// Good - remove when done
element.addEventListener('click', handler);
// Later...
element.removeEventListener('click', handler);

// Or use AbortController
const controller = new AbortController();
element.addEventListener('click', handler, { signal: controller.signal });
// Later...
controller.abort(); // Removes all listeners with this signal
```

**3. Timers:**
```javascript
// Bad - timer never cleared
const intervalId = setInterval(() => {
  console.log('Running...');
}, 1000);

// Good - clear when done
const intervalId = setInterval(() => {
  console.log('Running...');
}, 1000);

// Later...
clearInterval(intervalId);
```

**4. Closures:**
```javascript
// Bad - closure holds large object
function createHandler() {
  const largeData = new Array(1000000).fill('data');
  
  return function() {
    console.log(largeData[0]); // Holds entire array
  };
}

// Good - extract only what's needed
function createHandler() {
  const largeData = new Array(1000000).fill('data');
  const firstItem = largeData[0];
  
  return function() {
    console.log(firstItem); // Only holds one value
  };
}
```

**5. Detached DOM elements:**
```javascript
// Bad - element removed from DOM but still referenced
let element = document.getElementById('myElement');
document.body.removeChild(element);
// element still in memory

// Good - clear reference
element.remove();
element = null;
```

**Angular-specific prevention:**
```typescript
export class MyComponent implements OnDestroy {
  private destroy$ = new Subject<void>();
  
  ngOnInit() {
    this.service.getData()
      .pipe(takeUntil(this.destroy$))
      .subscribe();
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

**Detection Tools:**
- Chrome DevTools Memory Profiler
- Heap snapshots
- Performance timeline

---

## HTML Interview Questions

### 1. **Explain Semantic HTML and its importance?**

**Answer:**

Semantic HTML uses tags that convey meaning about the content:

**Semantic Elements:**
```html
<!-- Semantic Structure -->
<header>
  <nav>
    <ul>
      <li><a href="/">Home</a></li>
    </ul>
  </nav>
</header>

<main>
  <article>
    <header>
      <h1>Article Title</h1>
      <time datetime="2026-01-10">January 10, 2026</time>
    </header>
    
    <section>
      <h2>Section Title</h2>
      <p>Content...</p>
    </section>
    
    <aside>
      <p>Related content</p>
    </aside>
    
    <footer>
      <p>Author: John Doe</p>
    </footer>
  </article>
</main>

<footer>
  <p>&copy; 2026 Company</p>
</footer>
```

**Benefits:**
- **SEO:** Search engines understand content better
- **Accessibility:** Screen readers navigate better
- **Maintainability:** Code is self-documenting
- **Consistency:** Standardized structure

**Non-Semantic vs Semantic:**
```html
<!-- Non-Semantic -->
<div class="header">
  <div class="nav">...</div>
</div>

<!-- Semantic -->
<header>
  <nav>...</nav>
</header>
```

### 2. **Explain HTML5 Form Validation and Input Types?**

**Answer:**

**Input Types:**
```html
<input type="email" required />
<input type="url" required />
<input type="tel" pattern="[0-9]{10}" />
<input type="number" min="1" max="100" step="5" />
<input type="date" min="2026-01-01" />
<input type="time" />
<input type="color" />
<input type="range" min="0" max="100" />
<input type="search" />
```

**Validation Attributes:**
```html
<form novalidate> <!-- Disable browser validation -->
  <input 
    type="text"
    required
    minlength="3"
    maxlength="20"
    pattern="[A-Za-z]{3,}"
    placeholder="Username"
  />
  
  <input 
    type="email"
    required
    pattern="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$"
  />
  
  <button type="submit">Submit</button>
</form>
```

**JavaScript Validation API:**
```javascript
const input = document.getElementById('email');

// Check validity
if (!input.validity.valid) {
  if (input.validity.valueMissing) {
    input.setCustomValidity('Email is required');
  } else if (input.validity.typeMismatch) {
    input.setCustomValidity('Please enter a valid email');
  }
}

// Validate on submit
form.addEventListener('submit', (e) => {
  if (!form.checkValidity()) {
    e.preventDefault();
    // Show errors
  }
});
```

**Custom Validation:**
```html
<input type="password" id="password" />
<input type="password" id="confirmPassword" />

<script>
const password = document.getElementById('password');
const confirm = document.getElementById('confirmPassword');

confirm.addEventListener('input', () => {
  if (password.value !== confirm.value) {
    confirm.setCustomValidity('Passwords do not match');
  } else {
    confirm.setCustomValidity('');
  }
});
</script>
```

### 3. **Explain HTML Accessibility (ARIA) attributes?**

**Answer:**

ARIA (Accessible Rich Internet Applications) makes web content accessible:

**ARIA Roles:**
```html
<div role="navigation">
  <ul>
    <li><a href="/">Home</a></li>
  </ul>
</div>

<div role="main">
  <article role="article">...</article>
</div>

<div role="complementary">
  Sidebar content
</div>

<div role="alert">
  Error message
</div>
```

**ARIA States and Properties:**
```html
<!-- aria-label: Accessible name -->
<button aria-label="Close dialog">
  <span aria-hidden="true">×</span>
</button>

<!-- aria-labelledby: Reference to label -->
<h2 id="dialog-title">Confirmation</h2>
<div role="dialog" aria-labelledby="dialog-title">
  ...
</div>

<!-- aria-describedby: Additional description -->
<input 
  type="email" 
  aria-describedby="email-help"
/>
<span id="email-help">We'll never share your email</span>

<!-- aria-expanded: Disclosure state -->
<button 
  aria-expanded="false" 
  aria-controls="menu"
>
  Menu
</button>
<ul id="menu" hidden>...</ul>

<!-- aria-selected: Selection state -->
<div role="tablist">
  <button role="tab" aria-selected="true">Tab 1</button>
  <button role="tab" aria-selected="false">Tab 2</button>
</div>

<!-- aria-disabled: Disabled state -->
<button aria-disabled="true">Submit</button>

<!-- aria-live: Dynamic content -->
<div aria-live="polite">
  <p>Updated content</p>
</div>
```

**Best Practices:**
```html
<!-- Use semantic HTML first -->
<nav> <!-- Better than <div role="navigation"> -->

<!-- Skip to main content -->
<a href="#main" class="skip-link">Skip to main content</a>
<main id="main">...</main>

<!-- Form labels -->
<label for="username">Username</label>
<input id="username" type="text" />

<!-- Image alt text -->
<img src="logo.png" alt="Company Logo" />

<!-- Decorative images -->
<img src="decoration.png" alt="" role="presentation" />
```

**Angular ARIA Example:**
```typescript
@Component({
  template: `
    <button 
      [attr.aria-expanded]="isExpanded"
      [attr.aria-controls]="menuId"
      (click)="toggle()"
    >
      Menu
    </button>
    
    <ul [id]="menuId" [hidden]="!isExpanded">
      <li>Item 1</li>
    </ul>
  `
})
export class MenuComponent {
  isExpanded = false;
  menuId = 'menu-' + Math.random();
  
  toggle() {
    this.isExpanded = !this.isExpanded;
  }
}
```

### 4. **Explain HTML5 Storage APIs (localStorage, sessionStorage, IndexedDB)?**

**Answer:**

**localStorage:** Persists data permanently
```javascript
// Set item
localStorage.setItem('user', JSON.stringify({ name: 'John' }));

// Get item
const user = JSON.parse(localStorage.getItem('user'));

// Remove item
localStorage.removeItem('user');

// Clear all
localStorage.clear();

// Iterate
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  console.log(key, localStorage.getItem(key));
}

// Storage event (listen to changes in other tabs)
window.addEventListener('storage', (e) => {
  console.log(`Key ${e.key} changed from ${e.oldValue} to ${e.newValue}`);
});
```

**sessionStorage:** Persists only for session
```javascript
// Same API as localStorage
sessionStorage.setItem('temp', 'value');

// Cleared when tab closes
```

**Limitations:**
- 5-10 MB storage limit
- Synchronous (blocks main thread)
- String storage only
- Same-origin only

**IndexedDB:** Asynchronous, large storage
```javascript
// Open database
const request = indexedDB.open('MyDatabase', 1);

request.onupgradeneeded = (event) => {
  const db = event.target.result;
  
  // Create object store
  const objectStore = db.createObjectStore('users', { keyPath: 'id', autoIncrement: true });
  
  // Create indexes
  objectStore.createIndex('email', 'email', { unique: true });
  objectStore.createIndex('name', 'name', { unique: false });
};

request.onsuccess = (event) => {
  const db = event.target.result;
  
  // Add data
  const transaction = db.transaction(['users'], 'readwrite');
  const objectStore = transaction.objectStore('users');
  const addRequest = objectStore.add({ name: 'John', email: 'john@example.com' });
  
  addRequest.onsuccess = () => {
    console.log('User added');
  };
  
  // Get data
  const getRequest = objectStore.get(1);
  getRequest.onsuccess = () => {
    console.log(getRequest.result);
  };
  
  // Query by index
  const index = objectStore.index('email');
  const queryRequest = index.get('john@example.com');
  
  // Iterate with cursor
  const cursorRequest = objectStore.openCursor();
  cursorRequest.onsuccess = (event) => {
    const cursor = event.target.result;
    if (cursor) {
      console.log(cursor.value);
      cursor.continue();
    }
  };
};
```

**Comparison:**
- **localStorage:** Simple key-value, persistent
- **sessionStorage:** Simple key-value, temporary
- **IndexedDB:** Complex queries, large data, async
- **Cookies:** Small data, sent with requests

### 5. **Explain HTML5 APIs (Geolocation, Web Workers, WebSockets)?**

**Answer:**

**Geolocation API:**
```javascript
if ('geolocation' in navigator) {
  navigator.geolocation.getCurrentPosition(
    (position) => {
      console.log('Latitude:', position.coords.latitude);
      console.log('Longitude:', position.coords.longitude);
      console.log('Accuracy:', position.coords.accuracy);
    },
    (error) => {
      console.error('Error:', error.message);
    },
    {
      enableHighAccuracy: true,
      timeout: 5000,
      maximumAge: 0
    }
  );
  
  // Watch position (continuous tracking)
  const watchId = navigator.geolocation.watchPosition(successCallback);
  
  // Stop watching
  navigator.geolocation.clearWatch(watchId);
}
```

**Web Workers:** Run scripts in background thread
```javascript
// main.js
const worker = new Worker('worker.js');

worker.postMessage({ type: 'START', data: largeDataset });

worker.onmessage = (event) => {
  console.log('Result from worker:', event.data);
};

worker.onerror = (error) => {
  console.error('Worker error:', error);
};

// Terminate worker
worker.terminate();
```

```javascript
// worker.js
self.onmessage = (event) => {
  const { type, data } = event.data;
  
  if (type === 'START') {
    // Heavy computation
    const result = processLargeDataset(data);
    self.postMessage(result);
  }
};

function processLargeDataset(data) {
  // CPU-intensive task
  return data.map(item => item * 2);
}
```

**WebSockets:** Real-time bi-directional communication
```javascript
const socket = new WebSocket('wss://example.com/socket');

socket.onopen = () => {
  console.log('Connected');
  socket.send(JSON.stringify({ type: 'subscribe', channel: 'updates' }));
};

socket.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Received:', data);
};

socket.onerror = (error) => {
  console.error('WebSocket error:', error);
};

socket.onclose = (event) => {
  console.log('Connection closed:', event.code, event.reason);
  
  // Reconnect logic
  setTimeout(() => {
    connectWebSocket();
  }, 1000);
};

// Send data
socket.send(JSON.stringify({ message: 'Hello' }));

// Close connection
socket.close(1000, 'Normal closure');
```

**Other HTML5 APIs:**
- **Fetch API:** Modern HTTP requests
- **Canvas API:** 2D drawing
- **Web Audio API:** Audio processing
- **Notification API:** Desktop notifications
- **Drag and Drop API:** Drag interactions
- **History API:** Browser history manipulation
- **Intersection Observer:** Visibility detection
- **Resize Observer:** Element size changes

### 6. **Explain Shadow DOM and Web Components?**

**Answer:**

**Web Components** consist of:
1. Custom Elements
2. Shadow DOM
3. HTML Templates

**Custom Element:**
```javascript
class UserCard extends HTMLElement {
  constructor() {
    super();
    
    // Attach shadow DOM
    this.attachShadow({ mode: 'open' });
    
    this.shadowRoot.innerHTML = `
      <style>
        .card {
          border: 1px solid #ccc;
          padding: 20px;
          border-radius: 8px;
        }
        .name {
          font-weight: bold;
          font-size: 18px;
        }
      </style>
      
      <div class="card">
        <div class="name"></div>
        <div class="email"></div>
      </div>
    `;
  }
  
  connectedCallback() {
    // Element added to DOM
    this.render();
  }
  
  static get observedAttributes() {
    return ['name', 'email'];
  }
  
  attributeChangedCallback(name, oldValue, newValue) {
    // Attribute changed
    this.render();
  }
  
  render() {
    const name = this.getAttribute('name') || '';
    const email = this.getAttribute('email') || '';
    
    this.shadowRoot.querySelector('.name').textContent = name;
    this.shadowRoot.querySelector('.email').textContent = email;
  }
}

// Register custom element
customElements.define('user-card', UserCard);
```

**Usage:**
```html
<user-card name="John Doe" email="john@example.com"></user-card>
```

**HTML Template:**
```html
<template id="user-card-template">
  <style>
    .card { padding: 20px; }
  </style>
  <div class="card">
    <slot name="name"></slot>
    <slot name="email"></slot>
  </div>
</template>

<script>
class UserCard extends HTMLElement {
  constructor() {
    super();
    const template = document.getElementById('user-card-template');
    const content = template.content.cloneNode(true);
    
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.appendChild(content);
  }
}

customElements.define('user-card', UserCard);
</script>

<user-card>
  <span slot="name">John Doe</span>
  <span slot="email">john@example.com</span>
</user-card>
```

**Shadow DOM Benefits:**
- **Encapsulation:** CSS/JS doesn't leak out
- **Reusability:** Self-contained components
- **Composition:** Combine components
- **Scoped Styling:** No global CSS conflicts

**Angular Elements:** Export Angular components as Web Components
```typescript
import { createCustomElement } from '@angular/elements';

@Component({
  selector: 'app-user-card',
  template: `<div>{{ name }}</div>`
})
export class UserCardComponent {
  @Input() name: string;
}

// In module
const UserCardElement = createCustomElement(UserCardComponent, { injector });
customElements.define('user-card', UserCardElement);
```

### 7. **Explain HTML Performance Optimization?**

**Answer:**

**1. Resource Hints:**
```html
<!-- DNS Prefetch -->
<link rel="dns-prefetch" href="https://api.example.com">

<!-- Preconnect -->
<link rel="preconnect" href="https://cdn.example.com">

<!-- Prefetch (low priority) -->
<link rel="prefetch" href="/next-page.html">

<!-- Preload (high priority) -->
<link rel="preload" href="critical.css" as="style">
<link rel="preload" href="hero.jpg" as="image">

<!-- Prerender -->
<link rel="prerender" href="/next-page.html">
```

**2. Script Loading:**
```html
<!-- Defer: Download in parallel, execute after parsing -->
<script src="script.js" defer></script>

<!-- Async: Download and execute asynchronously -->
<script src="analytics.js" async></script>

<!-- Module: Deferred by default -->
<script type="module" src="app.js"></script>
```

**3. Image Optimization:**
```html
<!-- Lazy loading -->
<img src="image.jpg" loading="lazy" alt="Description">

<!-- Responsive images -->
<img 
  srcset="
    small.jpg 300w,
    medium.jpg 600w,
    large.jpg 1200w
  "
  sizes="(max-width: 600px) 300px, (max-width: 1200px) 600px, 1200px"
  src="medium.jpg"
  alt="Description"
>

<!-- Picture element -->
<picture>
  <source media="(max-width: 600px)" srcset="small.webp" type="image/webp">
  <source media="(max-width: 600px)" srcset="small.jpg">
  <source srcset="large.webp" type="image/webp">
  <img src="large.jpg" alt="Description">
</picture>
```

**4. Critical CSS:**
```html
<head>
  <!-- Inline critical CSS -->
  <style>
    /* Above-the-fold styles */
    .header { ... }
  </style>
  
  <!-- Load rest of CSS async -->
  <link rel="preload" href="styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
  <noscript><link rel="stylesheet" href="styles.css"></noscript>
</head>
```

**5. Compression:**
```html
<!-- Use modern formats -->
<img src="image.webp" alt="Description">
<video src="video.webm"></video>
```

**6. Reduce HTTP Requests:**
- Combine CSS/JS files
- Use CSS sprites
- Inline small assets
- Use SVG instead of images

**7. Caching:**
```html
<meta http-equiv="Cache-Control" content="max-age=31536000">
```

**Performance Metrics:**
- FCP (First Contentful Paint)
- LCP (Largest Contentful Paint)
- FID (First Input Delay)
- CLS (Cumulative Layout Shift)
- TTI (Time to Interactive)

---

## CSS Interview Questions

### 1. **Explain CSS Flexbox with practical examples?**

**Answer:**

Flexbox provides one-dimensional layout (row or column):

**Basic Flexbox:**
```css
.container {
  display: flex;
  flex-direction: row; /* row | column | row-reverse | column-reverse */
  justify-content: space-between; /* main axis alignment */
  align-items: center; /* cross axis alignment */
  flex-wrap: wrap; /* wrap | nowrap | wrap-reverse */
  gap: 20px; /* spacing between items */
}

.item {
  flex: 1; /* flex-grow flex-shrink flex-basis */
  flex-grow: 1; /* grow to fill space */
  flex-shrink: 0; /* don't shrink */
  flex-basis: 200px; /* initial size */
}
```

**Common Patterns:**

**1. Center Element:**
```css
.center {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
```

**2. Navigation Bar:**
```css
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem;
}

.nav-items {
  display: flex;
  gap: 2rem;
}
```

**3. Card Layout:**
```css
.card-container {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.card {
  flex: 1 1 300px; /* grow, shrink, base width */
  min-width: 0; /* prevent overflow */
}
```

**4. Sticky Footer:**
```css
body {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

main {
  flex: 1; /* grow to fill space */
}

footer {
  flex-shrink: 0;
}
```

**Properties:**
- **Container:** display, flex-direction, justify-content, align-items, flex-wrap, gap
- **Items:** flex-grow, flex-shrink, flex-basis, order, align-self

### 2. **Explain CSS Grid with practical examples?**

**Answer:**

Grid provides two-dimensional layout:

**Basic Grid:**
```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* 3 equal columns */
  grid-template-rows: 100px auto 50px;
  gap: 20px;
  
  /* Alternative column definitions */
  grid-template-columns: 200px 1fr 2fr; /* fixed, flexible */
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); /* responsive */
}

.item {
  grid-column: 1 / 3; /* span columns 1-2 */
  grid-row: 1 / 2;
  
  /* Shorthand */
  grid-area: 1 / 1 / 2 / 3; /* row-start / col-start / row-end / col-end */
}
```

**Named Grid Areas:**
```css
.container {
  display: grid;
  grid-template-areas:
    "header header header"
    "sidebar main main"
    "footer footer footer";
  grid-template-columns: 200px 1fr 1fr;
  grid-template-rows: auto 1fr auto;
  gap: 20px;
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.footer { grid-area: footer; }
```

**Responsive Grid:**
```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
}

/* auto-fit: collapse empty tracks */
/* auto-fill: keep empty tracks */
```

**Common Patterns:**

**1. Holy Grail Layout:**
```css
.layout {
  display: grid;
  grid-template-areas:
    "header header header"
    "nav content aside"
    "footer footer footer";
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}
```

**2. Card Grid:**
```css
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 2rem;
}
```

**3. Masonry-like Layout:**
```css
.masonry {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-auto-rows: 100px;
  gap: 10px;
}

.item {
  grid-row: span 2; /* span multiple rows */
}
```

**Flexbox vs Grid:**
- **Flexbox:** One-dimensional, content-first
- **Grid:** Two-dimensional, layout-first
- Use both together for complex layouts

### 3. **Explain CSS Positioning (static, relative, absolute, fixed, sticky)?**

**Answer:**

**Static (default):**
```css
.element {
  position: static; /* Normal document flow */
}
```

**Relative:**
```css
.element {
  position: relative;
  top: 10px; /* Offset from normal position */
  left: 20px;
  z-index: 1;
}
```

**Absolute:**
```css
.parent {
  position: relative; /* Positioning context */
}

.child {
  position: absolute;
  top: 0;
  right: 0;
  /* Positioned relative to nearest positioned ancestor */
}
```

**Fixed:**
```css
.header {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 100;
  /* Fixed to viewport */
}
```

**Sticky:**
```css
.navbar {
  position: sticky;
  top: 0;
  z-index: 100;
  /* Sticky until scroll threshold */
}
```

**Practical Examples:**

**1. Modal Overlay:**
```css
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1000;
}

.modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 1001;
}
```

**2. Tooltip:**
```css
.tooltip-container {
  position: relative;
}

.tooltip {
  position: absolute;
  bottom: 100%;
  left: 50%;
  transform: translateX(-50%);
  margin-bottom: 10px;
}
```

**3. Badge:**
```css
.icon-container {
  position: relative;
}

.badge {
  position: absolute;
  top: -8px;
  right: -8px;
  background: red;
  border-radius: 50%;
}
```

### 4. **Explain CSS Specificity and Cascade?**

**Answer:**

**Specificity Hierarchy (low to high):**
1. Universal selector (*), combinators (+, >, ~)
2. Type selectors (div, h1)
3. Class selectors (.class), attribute selectors ([type]), pseudo-classes (:hover)
4. ID selectors (#id)
5. Inline styles (style="")
6. !important

**Calculation:**
```css
/* Specificity: 0-0-1 */
div { }

/* Specificity: 0-1-0 */
.class { }

/* Specificity: 1-0-0 */
#id { }

/* Specificity: 0-2-1 */
div.class.another { }

/* Specificity: 1-1-1 */
#id .class div { }

/* Specificity: 1-0-0 */
[id="id"] { } /* Attribute selector */
```

**Cascade Order:**
1. Origin and importance (user-agent → user → author → !important)
2. Specificity
3. Order of appearance (last wins)

**Best Practices:**
```css
/* Bad: Over-specific */
div#container .content ul li a { }

/* Good: Lower specificity */
.nav-link { }

/* Avoid !important */
.button { color: red !important; } /* Bad */

/* Use cascade properly */
.button { color: blue; }
.button-primary { color: red; } /* Specific variant */
```

**Managing Specificity:**
```css
/* BEM naming convention */
.block { }
.block__element { }
.block__element--modifier { }

/* Utility classes (low specificity) */
.mt-4 { margin-top: 1rem; }

/* Component classes (medium specificity) */
.card { }
.card-header { }
```

### 5. **Explain CSS Preprocessors (SASS/SCSS) features?**

**Answer:**

**Variables:**
```scss
// Variables
$primary-color: #007bff;
$font-size-base: 16px;
$spacing-unit: 8px;

.button {
  background: $primary-color;
  font-size: $font-size-base;
  padding: $spacing-unit * 2;
}

// CSS Variables (native)
:root {
  --primary-color: #007bff;
}

.button {
  background: var(--primary-color);
}
```

**Nesting:**
```scss
.nav {
  background: white;
  
  &__item {
    padding: 10px;
    
    &:hover {
      background: gray;
    }
    
    &--active {
      font-weight: bold;
    }
  }
}

// Output:
// .nav { background: white; }
// .nav__item { padding: 10px; }
// .nav__item:hover { background: gray; }
// .nav__item--active { font-weight: bold; }
```

**Mixins:**
```scss
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

@mixin respond-to($breakpoint) {
  @if $breakpoint == mobile {
    @media (max-width: 768px) { @content; }
  }
}

.container {
  @include flex-center;
  
  @include respond-to(mobile) {
    flex-direction: column;
  }
}
```

**Functions:**
```scss
@function px-to-rem($px) {
  @return $px / 16px * 1rem;
}

.text {
  font-size: px-to-rem(24px); // 1.5rem
}
```

**Extend/Inheritance:**
```scss
%button-base {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
}

.button-primary {
  @extend %button-base;
  background: blue;
  color: white;
}

.button-secondary {
  @extend %button-base;
  background: gray;
  color: black;
}
```

**Partials and Import:**
```scss
// _variables.scss
$primary-color: blue;

// _mixins.scss
@mixin flex-center { ... }

// main.scss
@import 'variables';
@import 'mixins';

// Modern: @use (better namespacing)
@use 'variables' as vars;
.button { background: vars.$primary-color; }
```

### 6. **Explain CSS Animations and Transitions?**

**Answer:**

**Transitions:** Animate property changes
```css
.button {
  background: blue;
  transition: background 0.3s ease, transform 0.2s ease-out;
}

.button:hover {
  background: darkblue;
  transform: scale(1.1);
}

/* Shorthand: property duration timing-function delay */
transition: all 0.3s ease 0.1s;
```

**Timing Functions:**
```css
transition-timing-function: ease; /* default */
transition-timing-function: linear;
transition-timing-function: ease-in;
transition-timing-function: ease-out;
transition-timing-function: ease-in-out;
transition-timing-function: cubic-bezier(0.42, 0, 0.58, 1);
```

**Animations:** Define keyframes
```css
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.element {
  animation: fadeIn 0.5s ease-out forwards;
}

/* Multiple steps */
@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.1); }
  100% { transform: scale(1); }
}

.icon {
  animation: pulse 2s infinite;
}
```

**Animation Properties:**
```css
.element {
  animation-name: fadeIn;
  animation-duration: 1s;
  animation-timing-function: ease-out;
  animation-delay: 0.5s;
  animation-iteration-count: infinite; /* or number */
  animation-direction: alternate; /* normal | reverse | alternate */
  animation-fill-mode: forwards; /* none | forwards | backwards | both */
  animation-play-state: paused; /* running | paused */
  
  /* Shorthand */
  animation: fadeIn 1s ease-out 0.5s infinite alternate forwards;
}
```

**Practical Examples:**

**1. Loading Spinner:**
```css
@keyframes spin {
  to { transform: rotate(360deg); }
}

.spinner {
  animation: spin 1s linear infinite;
}
```

**2. Skeleton Loading:**
```css
@keyframes shimmer {
  0% { background-position: -1000px 0; }
  100% { background-position: 1000px 0; }
}

.skeleton {
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 1000px 100%;
  animation: shimmer 2s infinite;
}
```

**3. Slide In:**
```css
@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(-100%);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.slide-in {
  animation: slideIn 0.5s ease-out;
}
```

**Performance:**
- Animate `transform` and `opacity` (GPU accelerated)
- Avoid animating `width`, `height`, `top`, `left` (causes reflow)
- Use `will-change` sparingly for hints

```css
.optimized {
  will-change: transform, opacity;
  transform: translateZ(0); /* Force GPU acceleration */
}
```

### 7. **Explain CSS Responsive Design and Media Queries?**

**Answer:**

**Mobile-First Approach:**
```css
/* Base styles (mobile) */
.container {
  padding: 1rem;
}

/* Tablet and up */
@media (min-width: 768px) {
  .container {
    padding: 2rem;
  }
}

/* Desktop and up */
@media (min-width: 1024px) {
  .container {
    padding: 3rem;
    max-width: 1200px;
    margin: 0 auto;
  }
}
```

**Common Breakpoints:**
```css
/* Small devices (mobile) */
@media (max-width: 767px) { }

/* Medium devices (tablet) */
@media (min-width: 768px) and (max-width: 1023px) { }

/* Large devices (desktop) */
@media (min-width: 1024px) { }

/* Extra large devices */
@media (min-width: 1440px) { }
```

**Other Media Queries:**
```css
/* Orientation */
@media (orientation: landscape) { }
@media (orientation: portrait) { }

/* High DPI screens */
@media (-webkit-min-device-pixel-ratio: 2),
       (min-resolution: 192dpi) {
  /* Retina styles */
}

/* Dark mode */
@media (prefers-color-scheme: dark) {
  body {
    background: #1a1a1a;
    color: white;
  }
}

/* Reduced motion */
@media (prefers-reduced-motion: reduce) {
  * {
    animation: none !important;
    transition: none !important;
  }
}

/* Print */
@media print {
  .no-print { display: none; }
  body { font-size: 12pt; }
}
```

**Container Queries (Modern):**
```css
.card-container {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 1fr 2fr;
  }
}
```

**Responsive Units:**
```css
/* Viewport units */
.hero {
  height: 100vh; /* viewport height */
  width: 100vw; /* viewport width */
  font-size: 5vw; /* 5% of viewport width */
}

/* Relative units */
.text {
  font-size: 1rem; /* relative to root font-size */
  padding: 2em; /* relative to element font-size */
}

/* Modern viewport units */
.header {
  height: 100dvh; /* dynamic viewport height (accounts for mobile UI) */
}
```

**Responsive Images:**
```css
img {
  max-width: 100%;
  height: auto;
}

.responsive-image {
  width: 100%;
  height: auto;
  object-fit: cover;
  aspect-ratio: 16 / 9;
}
```

### 8. **Explain CSS Architecture (BEM, SMACSS, OOCSS)?**

**Answer:**

**BEM (Block Element Modifier):**
```css
/* Block */
.card { }

/* Element */
.card__header { }
.card__body { }
.card__footer { }

/* Modifier */
.card--featured { }
.card--large { }

/* Usage */
<div class="card card--featured">
  <div class="card__header">Title</div>
  <div class="card__body">Content</div>
</div>
```

**Benefits:**
- Clear naming convention
- Avoids specificity issues
- Self-documenting code
- Easy to understand relationships

**SMACSS (Scalable and Modular Architecture for CSS):**
```css
/* Base */
body, h1, p { }

/* Layout */
.l-header { }
.l-sidebar { }

/* Module */
.button { }
.card { }

/* State */
.is-active { }
.is-hidden { }

/* Theme */
.theme-dark { }
```

**OOCSS (Object-Oriented CSS):**
```css
/* Separate structure and skin */

/* Structure */
.button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
}

/* Skin */
.button-primary {
  background: blue;
  color: white;
}

.button-secondary {
  background: gray;
  color: black;
}

/* Separate container and content */
.media {
  display: flex;
}

.media-image {
  margin-right: 10px;
}

.media-body {
  flex: 1;
}
```

**Utility-First (Tailwind CSS approach):**
```css
/* Utility classes */
.mt-4 { margin-top: 1rem; }
.p-2 { padding: 0.5rem; }
.flex { display: flex; }
.items-center { align-items: center; }

/* Usage */
<div class="flex items-center p-4 bg-blue-500 text-white">
  Content
</div>
```

**CSS Modules (Component-scoped):**
```css
/* component.module.css */
.button {
  background: blue;
}

/* Compiled to unique class name */
/* button_ab123 */
```

**Best Practices:**
- Keep specificity low
- Use consistent naming
- Organize code logically
- Avoid deep nesting
- Document complex patterns

---

## Leadership & Mentorship Questions

### 1. **How do you conduct code reviews effectively?**

**Answer:**

**Code Review Checklist:**

**1. Functionality:**
- Does the code meet requirements?
- Are edge cases handled?
- Is error handling appropriate?

**2. Code Quality:**
- Is the code readable and maintainable?
- Are naming conventions followed?
- Is there unnecessary complexity?
- Are there code smells (duplicated code, long methods)?

**3. Architecture:**
- Does it follow project architecture?
- Are design patterns used appropriately?
- Is the code modular and reusable?

**4. Performance:**
- Are there performance bottlenecks?
- Is memory management appropriate?
- Are expensive operations optimized?

**5. Security:**
- Are inputs validated?
- Is sensitive data protected?
- Are there potential vulnerabilities (XSS, CSRF)?

**6. Testing:**
- Are there adequate unit tests?
- Is test coverage sufficient?
- Are tests meaningful?

**Best Practices:**
```typescript
// Example review comments

// ❌ Bad feedback (not constructive)
// "This code is bad"

// ✅ Good feedback (specific, constructive)
// "Consider extracting this logic into a separate function for better 
// testability and reusability. Example: extractUserData(response)"

// ❌ Bad
// "You're wrong"

// ✅ Good
// "What do you think about using a guard clause here to reduce nesting?
// if (!user) return;
// This makes the happy path clearer."

// Praise good work
// "Great use of RxJS operators here! The combination of switchMap 
// and catchError handles the async flow elegantly."
```

**Code Review Process:**
1. Understand the context and requirements
2. Review at a reasonable pace (200-400 lines per hour)
3. Focus on learning, not just finding errors
4. Be respectful and constructive
5. Distinguish between preferences and issues
6. Ask questions instead of making demands
7. Approve when ready, request changes when needed

### 2. **How do you mentor junior developers?**

**Answer:**

**Mentoring Strategies:**

**1. Set Clear Expectations:**
- Define goals and milestones
- Establish regular 1-on-1 meetings
- Create a safe environment for questions

**2. Teach Through Code Reviews:**
```typescript
// Instead of just pointing out issues, explain the reasoning

// ❌ Basic feedback
// "Use const instead of let"

// ✅ Mentoring feedback
// "We prefer const over let when the variable won't be reassigned.
// This makes the code more predictable and prevents accidental mutations.
// const declares: 'this value won't change', while let says: 'this might change'.
// This small practice helps prevent bugs in larger codebases."
```

**3. Pair Programming:**
- Work together on complex features
- Let them drive, you navigate
- Explain your thought process
- Show debugging techniques

**4. Provide Resources:**
- Recommend articles, courses, and documentation
- Share best practices and style guides
- Curate learning paths based on their goals

**5. Gradual Complexity:**
```
Week 1-2: Bug fixes, small enhancements
Week 3-4: Complete feature (with guidance)
Week 5-6: Feature with code review only
Week 7+: Independent features, review their design
```

**6. Encourage Questions:**
- "No question is too simple"
- Regular check-ins
- Open door policy

**7. Give Ownership:**
- Assign meaningful tasks
- Trust them with responsibility
- Let them make (safe) mistakes

**8. Provide Feedback:**
- Regular constructive feedback
- Celebrate wins and progress
- Address issues promptly but kindly

**Real Example:**
```typescript
// Junior developer's code
getUserData() {
  this.http.get('/api/users').subscribe(data => {
    this.users = data;
  });
}

// Mentoring approach:
// "This works well! Let me show you how we can make it more robust:
// 1. Handle errors with catchError
// 2. Unsubscribe to prevent memory leaks
// 3. Add type safety
// Let's refactor together..."

getUserData() {
  this.destroy$ = new Subject<void>();
  
  this.http.get<User[]>('/api/users')
    .pipe(
      catchError(error => {
        console.error('Failed to fetch users:', error);
        return of([]);
      }),
      takeUntil(this.destroy$)
    )
    .subscribe(data => {
      this.users = data;
    });
}

// "Now you understand why we do this. Try applying this pattern 
// to the posts component and I'll review it."
```

### 3. **How do you make technical design decisions?**

**Answer:**

**Decision-Making Framework:**

**1. Understand Requirements:**
- Functional requirements
- Non-functional requirements (performance, scalability, security)
- Business constraints (budget, timeline)

**2. Evaluate Options:**
```typescript
// Example: State Management Decision

// Option 1: Component State (Simple)
✅ Pros: Simple, no dependencies, fast
❌ Cons: Hard to share, doesn't scale

// Option 2: Service with BehaviorSubject
✅ Pros: Shared state, reactive, Angular native
❌ Cons: Can become complex, no devtools

// Option 3: NgRx
✅ Pros: Predictable, devtools, time-travel debugging
❌ Cons: Boilerplate, learning curve, overkill for simple apps

// Decision: 
// - Small app (<5 components sharing state): Service
// - Medium app (5-15 components): Service with patterns
// - Large app (15+ components, complex state): NgRx
```

**3. Consider Trade-offs:**
- Performance vs Maintainability
- Speed vs Quality
- Flexibility vs Simplicity
- Short-term vs Long-term

**4. Document Decision:**
```markdown
# Architecture Decision Record (ADR)

## Context
We need to implement real-time updates for the trading dashboard.

## Decision
Use WebSockets with RxJS subjects for state management.

## Alternatives Considered
1. Polling: Simple but inefficient, high server load
2. Server-Sent Events (SSE): One-way only, we need bi-directional
3. WebSockets: Chosen - bi-directional, real-time, efficient

## Consequences
✅ Real-time updates with low latency
✅ Reduced server load vs polling
✅ Works well with Angular's reactive patterns
❌ Need fallback for older browsers
❌ More complex error handling
❌ Need to manage connection lifecycle

## Implementation Notes
- Use reconnection logic with exponential backoff
- Implement heartbeat for connection health
- Handle offline/online scenarios
```

**5. Validation:**
- Prototype if needed
- Review with team
- Consider scalability
- Plan for failures

**6. Iterate:**
- Monitor after implementation
- Gather feedback
- Be willing to change if needed

### 4. **How do you handle technical debt?**

**Answer:**

**Technical Debt Management:**

**1. Identify Technical Debt:**
```typescript
// Examples of technical debt:

// Debt 1: Outdated dependencies
"@angular/core": "^10.0.0" // Current: 17.x

// Debt 2: No error handling
getData() {
  return this.http.get('/api/data'); // No error handling
}

// Debt 3: Tight coupling
class UserComponent {
  constructor(private http: HttpClient) {
    // Direct HTTP calls instead of service
  }
}

// Debt 4: Missing tests
// user.component.spec.ts doesn't exist

// Debt 5: Duplicated logic
// Same validation logic in 5 different components
```

**2. Categorize and Prioritize:**
```
High Priority:
- Security vulnerabilities
- Performance bottlenecks
- Blocking future features
- Frequent bug sources

Medium Priority:
- Outdated dependencies (non-critical)
- Missing documentation
- Test coverage gaps
- Code duplication

Low Priority:
- Style inconsistencies
- Minor refactoring opportunities
- Nice-to-have optimizations
```

**3. Allocate Time:**
```
Sprint Planning:
- 70% New features
- 20% Technical debt
- 10% Bug fixes

Or:
- Dedicated tech debt sprints quarterly
- "Boy Scout Rule": Leave code better than you found it
```

**4. Track Technical Debt:**
```typescript
// Add TODO comments with context
// TODO: [TECH-DEBT] Replace with state management
// TODO: [PERFORMANCE] Optimize this query - currently O(n²)
// TODO: [SECURITY] Add input sanitization

// Create tickets
// [DEBT-123] Upgrade to Angular 17
// [DEBT-124] Refactor user service to use repository pattern
```

**5. Prevent New Debt:**
- Code reviews
- Automated linting
- Definition of Done includes tests
- Architecture guidelines
- Regular refactoring

**Example Refactoring Plan:**
```typescript
// Before (Technical Debt)
@Component({...})
export class UserListComponent {
  users: any;
  
  ngOnInit() {
    // Tightly coupled, no error handling, no types
    this.http.get('http://localhost:3000/api/users')
      .subscribe(data => this.users = data);
  }
}

// Refactoring Plan:
// 1. Add types
// 2. Extract to service
// 3. Add error handling
// 4. Add loading states
// 5. Add tests

// After (Refactored)
@Component({...})
export class UserListComponent implements OnInit, OnDestroy {
  users$ = this.userService.getUsers().pipe(
    catchError(error => {
      this.notificationService.error('Failed to load users');
      return of([]);
    })
  );
  
  constructor(
    private userService: UserService,
    private notificationService: NotificationService
  ) {}
}

@Injectable({ providedIn: 'root' })
export class UserService {
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users');
  }
}
```

### 5. **How do you ensure code quality in your team?**

**Answer:**

**Code Quality Practices:**

**1. Code Standards:**
```typescript
// .eslintrc.json
{
  "extends": ["airbnb-typescript"],
  "rules": {
    "no-console": "error",
    "max-lines": ["error", 300],
    "@typescript-eslint/explicit-function-return-type": "error"
  }
}

// .prettierrc
{
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100
}
```

**2. Automated Checks:**
```json
// package.json
{
  "scripts": {
    "lint": "eslint src/**/*.ts",
    "format": "prettier --write src/**/*.ts",
    "test": "jest --coverage",
    "test:watch": "jest --watch"
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "pre-push": "npm test"
    }
  },
  "lint-staged": {
    "*.ts": ["eslint --fix", "prettier --write"]
  }
}
```

**3. Code Review Process:**
- Mandatory peer reviews
- Review checklist
- Automated checks must pass
- At least one approval required

**4. Testing Standards:**
```typescript
// Minimum coverage requirements
jest.config.js:
{
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  }
}

// Test structure
describe('UserService', () => {
  describe('getUsers', () => {
    it('should return users on success', () => {});
    it('should handle network errors', () => {});
    it('should handle empty response', () => {});
  });
});
```

**5. Architecture Guidelines:**
```
src/
├── core/           # Singleton services, guards
├── shared/         # Reusable components, directives, pipes
├── features/       # Feature modules
│   └── users/
│       ├── components/
│       ├── services/
│       ├── models/
│       └── users.module.ts
└── styles/         # Global styles
```

**6. Documentation:**
```typescript
/**
 * Fetches user data from the API
 * @param userId - The unique identifier of the user
 * @returns Observable<User> - User data or error
 * @throws {UserNotFoundException} When user is not found
 * @example
 * this.userService.getUser(123).subscribe(user => {
 *   console.log(user.name);
 * });
 */
getUser(userId: number): Observable<User> {
  return this.http.get<User>(`/api/users/${userId}`);
}
```

**7. Knowledge Sharing:**
- Regular tech talks
- Code review sessions
- Architecture discussions
- Documentation wiki
- Pair programming sessions

**8. Metrics and Monitoring:**
- Code coverage reports
- Build success rate
- Code review turnaround time
- Bug escape rate
- Technical debt ratio

### 6. **How do you handle disagreements in technical discussions?**

**Answer:**

**Conflict Resolution Approach:**

**1. Listen Actively:**
- Understand their perspective
- Ask clarifying questions
- Don't interrupt

**2. Focus on Facts:**
```typescript
// Instead of: "Your approach is wrong"
// Say: "Let's compare the approaches objectively"

Option A (Your proposal):
✅ Simpler implementation
❌ Doesn't scale beyond 1000 items

Option B (My proposal):
✅ Handles large datasets efficiently
❌ More complex, takes longer to implement

// Decision factors:
// - Current requirement: 100 items (A wins)
// - Future requirement: 10,000 items (B wins)
// - Timeline: 2 days (A wins)
```

**3. Seek to Understand:**
"Help me understand why you prefer this approach?"
"What problems are you trying to solve?"
"What are your concerns with my proposal?"

**4. Find Common Ground:**
"We both agree that performance is important"
"We both want maintainable code"
"Let's optimize for X"

**5. Data-Driven Decisions:**
```typescript
// Instead of opinions, use data

// Benchmark performance
console.time('Option A');
// ... run option A
console.timeEnd('Option A'); // 150ms

console.time('Option B');
// ... run option B
console.timeEnd('Option B'); // 50ms

// Result: Option B is 3x faster
```

**6. Escalate if Needed:**
- Involve architect or tech lead
- Present both options objectively
- Accept the decision

**7. Learn from Disagreements:**
- "What did I learn from this?"
- "Was my assumption wrong?"
- "How can I present better next time?"

**Example:**
```
Scenario: Choosing between NgRx and Service-based state management

Developer A: "We should use NgRx because it's industry standard"
Developer B (Me): "I understand NgRx is powerful, but let me share my concerns..."

My Approach:
1. Acknowledge: "I agree NgRx is widely used and has great devtools"

2. Present concerns objectively:
   - Our app has only 3 components sharing state
   - Team has no NgRx experience (learning curve)
   - Adds 50+ lines of boilerplate per feature
   - Timeline is tight (2 weeks)

3. Propose alternative:
   - Use service with BehaviorSubject now
   - Refactor to NgRx later if needed
   - Team learns NgRx in parallel

4. Seek compromise:
   "What if we use services now, and revisit NgRx in Sprint 3 
   when we have more complex state requirements?"

5. Accept decision:
   If team decides on NgRx, support it fully and help with implementation
```

---

## Scenario-Based Questions (Financial Services & Security)

### 1. **How would you build a real-time stock trading dashboard?**

**Answer:**

**Requirements Analysis:**
- Real-time price updates (WebSocket)
- Display multiple stocks simultaneously
- High performance (60 FPS)
- Historical charts
- Buy/Sell actions
- Authentication & authorization

**Architecture:**
```typescript
// 1. WebSocket Service
@Injectable({ providedIn: 'root' })
export class StockWebSocketService {
  private socket$: WebSocketSubject<any>;
  private reconnectAttempts = 0;
  private readonly MAX_RECONNECT_ATTEMPTS = 5;
  
  constructor(private authService: AuthService) {}
  
  connect(): Observable<StockUpdate> {
    if (!this.socket$ || this.socket$.closed) {
      this.socket$ = this.createWebSocket();
    }
    
    return this.socket$.pipe(
      map(data => this.transformData(data)),
      retry({
        count: this.MAX_RECONNECT_ATTEMPTS,
        delay: (error, retryCount) => {
          const delay = Math.min(1000 * Math.pow(2, retryCount), 30000);
          console.log(`Reconnecting in ${delay}ms...`);
          return timer(delay);
        }
      }),
      catchError(error => {
        console.error('WebSocket error:', error);
        return EMPTY;
      })
    );
  }
  
  private createWebSocket(): WebSocketSubject<any> {
    return webSocket({
      url: `wss://api.stocks.com/stream?token=${this.authService.getToken()}`,
      openObserver: {
        next: () => {
          console.log('WebSocket connected');
          this.reconnectAttempts = 0;
        }
      },
      closeObserver: {
        next: () => console.log('WebSocket closed')
      }
    });
  }
  
  subscribe(symbols: string[]): void {
    this.socket$.next({ 
      action: 'subscribe', 
      symbols 
    });
  }
  
  unsubscribe(symbols: string[]): void {
    this.socket$.next({ 
      action: 'unsubscribe', 
      symbols 
    });
  }
}

// 2. Stock State Management
interface StockState {
  stocks: { [symbol: string]: Stock };
  loading: boolean;
  error: string | null;
}

@Injectable({ providedIn: 'root' })
export class StockStateService {
  private state = new BehaviorSubject<StockState>({
    stocks: {},
    loading: false,
    error: null
  });
  
  state$ = this.state.asObservable();
  
  updateStock(symbol: string, data: Partial<Stock>): void {
    const currentState = this.state.value;
    this.state.next({
      ...currentState,
      stocks: {
        ...currentState.stocks,
        [symbol]: { ...currentState.stocks[symbol], ...data }
      }
    });
  }
}

// 3. Dashboard Component with Performance Optimization
@Component({
  selector: 'app-stock-dashboard',
  template: `
    <div class="dashboard">
      <app-stock-card
        *ngFor="let stock of stocks$ | async; trackBy: trackBySymbol"
        [stock]="stock"
        (trade)="handleTrade($event)"
      ></app-stock-card>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class StockDashboardComponent implements OnInit, OnDestroy {
  stocks$ = this.stockState.state$.pipe(
    map(state => Object.values(state.stocks)),
    shareReplay(1)
  );
  
  private destroy$ = new Subject<void>();
  
  constructor(
    private wsService: StockWebSocketService,
    private stockState: StockStateService
  ) {}
  
  ngOnInit(): void {
    const symbols = ['AAPL', 'GOOGL', 'MSFT', 'AMZN'];
    
    this.wsService.connect().pipe(
      takeUntil(this.destroy$)
    ).subscribe(update => {
      this.stockState.updateStock(update.symbol, update);
    });
    
    this.wsService.subscribe(symbols);
  }
  
  trackBySymbol(index: number, stock: Stock): string {
    return stock.symbol;
  }
  
  handleTrade(event: TradeEvent): void {
    // Handle buy/sell with proper validation
  }
  
  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}

// 4. Optimized Stock Card Component
@Component({
  selector: 'app-stock-card',
  template: `
    <div class="stock-card" [class.positive]="isPositive" [class.negative]="!isPositive">
      <div class="symbol">{{ stock.symbol }}</div>
      <div class="price">{{ stock.price | currency }}</div>
      <div class="change">{{ stock.change | number:'1.2-2' }}%</div>
      <canvas #chart></canvas>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class StockCardComponent implements AfterViewInit {
  @Input() stock: Stock;
  @ViewChild('chart') chartCanvas: ElementRef<HTMLCanvasElement>;
  
  get isPositive(): boolean {
    return this.stock.change >= 0;
  }
  
  ngAfterViewInit(): void {
    this.renderChart();
  }
  
  private renderChart(): void {
    // Use lightweight charting library or Canvas API
    // Implement with requestAnimationFrame for smooth updates
  }
}
```

**Performance Optimizations:**
1. OnPush change detection
2. Virtual scrolling for large lists
3. Web Workers for calculations
4. Canvas for charts (better than SVG for real-time)
5. TrackBy for ngFor
6. Debounce rapid updates
7. Unsubscribe on destroy

**Security:**
1. WSS (secure WebSocket)
2. Token-based authentication
3. Rate limiting
4. Input validation
5. XSS protection

### 2. **How would you implement authentication and authorization for a banking application?**

**Answer:**

**Complete Authentication Flow:**

```typescript
// 1. Auth Service
@Injectable({ providedIn: 'root' })
export class AuthService {
  private readonly TOKEN_KEY = 'auth_token';
  private readonly REFRESH_TOKEN_KEY = 'refresh_token';
  
  private currentUserSubject = new BehaviorSubject<User | null>(null);
  currentUser$ = this.currentUserSubject.asObservable();
  
  constructor(
    private http: HttpClient,
    private router: Router
  ) {
    this.loadUser();
  }
  
  login(credentials: LoginCredentials): Observable<AuthResponse> {
    return this.http.post<AuthResponse>('/api/auth/login', credentials).pipe(
      tap(response => {
        this.setSession(response);
        this.currentUserSubject.next(response.user);
      }),
      catchError(error => {
        if (error.status === 401) {
          throw new Error('Invalid credentials');
        }
        throw error;
      })
    );
  }
  
  loginWithMFA(code: string): Observable<AuthResponse> {
    return this.http.post<AuthResponse>('/api/auth/mfa', { code }).pipe(
      tap(response => this.setSession(response))
    );
  }
  
  private setSession(authResult: AuthResponse): void {
    // Use secure storage
    const encryptedToken = this.encryptToken(authResult.token);
    sessionStorage.setItem(this.TOKEN_KEY, encryptedToken);
    localStorage.setItem(this.REFRESH_TOKEN_KEY, authResult.refreshToken);
    
    // Set token expiry timer
    this.scheduleTokenRefresh(authResult.expiresIn);
  }
  
  private scheduleTokenRefresh(expiresIn: number): void {
    // Refresh 5 minutes before expiry
    const refreshTime = (expiresIn - 300) * 1000;
    
    timer(refreshTime).subscribe(() => {
      this.refreshToken().subscribe();
    });
  }
  
  refreshToken(): Observable<AuthResponse> {
    const refreshToken = localStorage.getItem(this.REFRESH_TOKEN_KEY);
    
    return this.http.post<AuthResponse>('/api/auth/refresh', { refreshToken }).pipe(
      tap(response => this.setSession(response)),
      catchError(() => {
        this.logout();
        return throwError(() => new Error('Session expired'));
      })
    );
  }
  
  logout(): void {
    sessionStorage.removeItem(this.TOKEN_KEY);
    localStorage.removeItem(this.REFRESH_TOKEN_KEY);
    this.currentUserSubject.next(null);
    this.router.navigate(['/login']);
  }
  
  getToken(): string | null {
    const encrypted = sessionStorage.getItem(this.TOKEN_KEY);
    return encrypted ? this.decryptToken(encrypted) : null;
  }
  
  isAuthenticated(): boolean {
    const token = this.getToken();
    return !!token && !this.isTokenExpired(token);
  }
  
  private isTokenExpired(token: string): boolean {
    const payload = JSON.parse(atob(token.split('.')[1]));
    return payload.exp * 1000 < Date.now();
  }
}

// 2. Auth Interceptor
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(private authService: AuthService) {}
  
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // Skip auth for public endpoints
    if (this.isPublicEndpoint(req.url)) {
      return next.handle(req);
    }
    
    const token = this.authService.getToken();
    
    if (token) {
      req = req.clone({
        setHeaders: {
          Authorization: `Bearer ${token}`,
          'X-Request-ID': this.generateRequestId(),
          'X-CSRF-Token': this.getCSRFToken()
        }
      });
    }
    
    return next.handle(req).pipe(
      catchError((error: HttpErrorResponse) => {
        if (error.status === 401) {
          // Token expired, try refresh
          return this.handle401Error(req, next);
        }
        return throwError(() => error);
      })
    );
  }
  
  private handle401Error(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return this.authService.refreshToken().pipe(
      switchMap(() => {
        const token = this.authService.getToken();
        req = req.clone({
          setHeaders: { Authorization: `Bearer ${token}` }
        });
        return next.handle(req);
      }),
      catchError(() => {
        this.authService.logout();
        return throwError(() => new Error('Authentication failed'));
      })
    );
  }
  
  private getCSRFToken(): string {
    return document.querySelector<HTMLMetaElement>('meta[name="csrf-token"]')?.content || '';
  }
}

// 3. Auth Guard
@Injectable({ providedIn: 'root' })
export class AuthGuard implements CanActivate {
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}
  
  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): Observable<boolean> {
    return this.authService.currentUser$.pipe(
      map(user => {
        if (!user) {
          this.router.navigate(['/login'], { 
            queryParams: { returnUrl: state.url } 
          });
          return false;
        }
        return true;
      }),
      take(1)
    );
  }
}

// 4. Role-Based Authorization Guard
@Injectable({ providedIn: 'root' })
export class RoleGuard implements CanActivate {
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}
  
  canActivate(route: ActivatedRouteSnapshot): Observable<boolean> {
    const requiredRoles = route.data['roles'] as string[];
    
    return this.authService.currentUser$.pipe(
      map(user => {
        if (!user) {
          this.router.navigate(['/login']);
          return false;
        }
        
        const hasRole = requiredRoles.some(role => user.roles.includes(role));
        
        if (!hasRole) {
          this.router.navigate(['/unauthorized']);
          return false;
        }
        
        return true;
      }),
      take(1)
    );
  }
}

// 5. Route Configuration
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule),
    canActivate: [AuthGuard, RoleGuard],
    data: { roles: ['ADMIN'] }
  },
  {
    path: 'transactions',
    loadChildren: () => import('./transactions/transactions.module').then(m => m.TransactionsModule),
    canActivate: [AuthGuard, RoleGuard],
    data: { roles: ['USER', 'ADMIN'] }
  }
];

// 6. Directive for UI-level permissions
@Directive({
  selector: '[appHasRole]'
})
export class HasRoleDirective implements OnInit {
  @Input() appHasRole: string[];
  
  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef,
    private authService: AuthService
  ) {}
  
  ngOnInit(): void {
    this.authService.currentUser$.subscribe(user => {
      this.viewContainer.clear();
      
      if (user && this.hasRequiredRole(user)) {
        this.viewContainer.createEmbeddedView(this.templateRef);
      }
    });
  }
  
  private hasRequiredRole(user: User): boolean {
    return this.appHasRole.some(role => user.roles.includes(role));
  }
}

// Usage in template
<button *appHasRole="['ADMIN']">Delete User</button>
```

**Security Best Practices:**
1. **Token Storage:** Use sessionStorage (not localStorage) for access tokens
2. **HTTPS Only:** All communication over HTTPS
3. **CSRF Protection:** Include CSRF tokens
4. **XSS Protection:** Sanitize all inputs, use Angular's built-in sanitization
5. **Secure Headers:** Set Content-Security-Policy, X-Frame-Options
6. **Rate Limiting:** Prevent brute force attacks
7. **MFA:** Multi-factor authentication for sensitive operations
8. **Session Timeout:** Auto logout after inactivity
9. **Audit Logging:** Log all authentication attempts
10. **Encryption:** Encrypt sensitive data in transit and at rest

### 3. **How would you optimize a slow Angular application?**

**Answer:**

**Performance Audit Process:**

**1. Identify Bottlenecks:**
```typescript
// Use Chrome DevTools Performance tab
// Run Lighthouse audit
// Check Angular DevTools profiler

// Common issues found:
// - Change detection running too often
// - Large bundles
// - Memory leaks
// - Inefficient rendering
// - Too many HTTP requests
```

**2. Change Detection Optimization:**
```typescript
// Before: Default change detection
@Component({
  selector: 'app-user-list',
  template: `
    <div *ngFor="let user of users">
      {{ user.name }} - {{ calculateAge(user.birthDate) }}
    </div>
  `
})
export class UserListComponent {
  users: User[];
  
  // Called on every change detection cycle!
  calculateAge(birthDate: Date): number {
    return new Date().getFullYear() - birthDate.getFullYear();
  }
}

// After: OnPush + pure pipe
@Component({
  selector: 'app-user-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div *ngFor="let user of users; trackBy: trackById">
      {{ user.name }} - {{ user.birthDate | age }}
    </div>
  `
})
export class UserListComponent {
  @Input() users: User[];
  
  trackById(index: number, user: User): number {
    return user.id;
  }
}

@Pipe({ name: 'age', pure: true })
export class AgePipe implements PipeTransform {
  transform(birthDate: Date): number {
    return new Date().getFullYear() - birthDate.getFullYear();
  }
}
```

**3. Bundle Size Optimization:**
```typescript
// Analyze bundle
// npm install -g webpack-bundle-analyzer
// ng build --stats-json
// webpack-bundle-analyzer dist/stats.json

// Solutions:

// a) Lazy loading
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule)
  }
];

// b) Remove unused imports
// Before
import * as _ from 'lodash'; // Entire library

// After
import { debounce } from 'lodash-es'; // Tree-shakeable

// c) Use lighter alternatives
// Replace moment.js (67KB) with date-fns (13KB)

// d) Code splitting
// Use dynamic imports for heavy components
async loadHeavyComponent() {
  const { HeavyComponent } = await import('./heavy.component');
  // Use component
}
```

**4. Virtual Scrolling for Large Lists:**
```typescript
// Before: Rendering 10,000 items
<div *ngFor="let item of items">{{ item.name }}</div>

// After: Virtual scrolling (renders only visible items)
<cdk-virtual-scroll-viewport itemSize="50" class="viewport">
  <div *cdkVirtualFor="let item of items">
    {{ item.name }}
  </div>
</cdk-virtual-scroll-viewport>

// Result: Renders ~20 items instead of 10,000
```

**5. Optimize HTTP Requests:**
```typescript
// Before: Multiple requests
this.http.get('/api/users').subscribe();
this.http.get('/api/posts').subscribe();
this.http.get('/api/comments').subscribe();

// After: Batch requests
this.http.post('/api/batch', {
  requests: [
    { endpoint: '/users' },
    { endpoint: '/posts' },
    { endpoint: '/comments' }
  ]
}).subscribe();

// Or use forkJoin for parallel requests
forkJoin({
  users: this.http.get('/api/users'),
  posts: this.http.get('/api/posts'),
  comments: this.http.get('/api/comments')
}).subscribe(({ users, posts, comments }) => {
  // All data loaded
});

// Cache frequently accessed data
@Injectable({ providedIn: 'root' })
export class CachedDataService {
  private cache = new Map<string, { data: any; timestamp: number }>();
  private readonly CACHE_DURATION = 5 * 60 * 1000; // 5 minutes
  
  getData(key: string): Observable<any> {
    const cached = this.cache.get(key);
    
    if (cached && Date.now() - cached.timestamp < this.CACHE_DURATION) {
      return of(cached.data);
    }
    
    return this.http.get(`/api/${key}`).pipe(
      tap(data => {
        this.cache.set(key, { data, timestamp: Date.now() });
      })
    );
  }
}
```

**6. Memory Leak Prevention:**
```typescript
// Before: Memory leak
export class LeakyComponent implements OnInit {
  ngOnInit() {
    this.dataService.getData().subscribe(data => {
      this.data = data; // Subscription never unsubscribed!
    });
    
    setInterval(() => {
      console.log('Running...'); // Never cleared!
    }, 1000);
  }
}

// After: Proper cleanup
export class OptimizedComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  private intervalId: number;
  
  ngOnInit() {
    this.dataService.getData()
      .pipe(takeUntil(this.destroy$))
      .subscribe(data => {
        this.data = data;
      });
    
    this.intervalId = window.setInterval(() => {
      console.log('Running...');
    }, 1000);
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
    clearInterval(this.intervalId);
  }
}
```

**7. Optimize Images:**
```html
<!-- Before -->
<img src="large-image.jpg" alt="Image">

<!-- After: Lazy loading + responsive images + modern formats -->
<img 
  srcset="
    small.webp 300w,
    medium.webp 600w,
    large.webp 1200w
  "
  sizes="(max-width: 600px) 300px, (max-width: 1200px) 600px, 1200px"
  src="medium.jpg"
  loading="lazy"
  alt="Image"
>
```

**Performance Metrics:**
```
Before optimization:
- Initial load: 8.5s
- Bundle size: 3.2MB
- FCP: 4.2s
- LCP: 6.1s

After optimization:
- Initial load: 2.1s (75% improvement)
- Bundle size: 850KB (73% reduction)
- FCP: 1.2s (71% improvement)
- LCP: 1.8s (70% improvement)
```

### 4. **How would you handle a data migration from legacy system to Angular application?**

**Answer:**

**Migration Strategy:**

**1. Assessment Phase:**
```typescript
// Analyze legacy system
Legacy System:
- Technology: jQuery + Server-side rendering
- Data: SQL database
- APIs: SOAP-based
- Users: 5000 daily active users
- Critical features: Reports, user management, transactions
```

**2. Migration Approach - Strangler Fig Pattern:**
```typescript
// Incrementally replace legacy system

Phase 1: Create Angular shell
- Set up routing
- Implement authentication
- Create shared components

Phase 2: Migrate one feature at a time
- Start with non-critical features
- Run both systems in parallel
- Gradually switch traffic

Phase 3: Data synchronization
- Implement two-way sync
- Validate data consistency
- Monitor for discrepancies

// Implementation
@Injectable({ providedIn: 'root' })
export class MigrationService {
  migrateUserData(userId: number): Observable<void> {
    return forkJoin({
      legacyData: this.getLegacyUserData(userId),
      newSystemReady: this.validateNewSystem()
    }).pipe(
      switchMap(({ legacyData }) => {
        const transformedData = this.transformData(legacyData);
        return this.saveToNewSystem(transformedData);
      }),
      tap(() => this.logMigration(userId, 'SUCCESS')),
      catchError(error => {
        this.logMigration(userId, 'FAILED', error);
        throw error;
      })
    );
  }
  
  private transformData(legacyData: any): User {
    // Transform legacy format to new format
    return {
      id: legacyData.USER_ID,
      name: `${legacyData.FIRST_NAME} ${legacyData.LAST_NAME}`,
      email: legacyData.EMAIL_ADDR.toLowerCase(),
      // Handle nulls, format dates, etc.
      createdAt: new Date(legacyData.CREATE_DATE)
    };
  }
}
```

**3. API Adapter Pattern:**
```typescript
// Wrap legacy SOAP API with REST-like service

@Injectable({ providedIn: 'root' })
export class LegacyApiAdapter {
  constructor(private http: HttpClient) {}
  
  // Convert SOAP to REST-like interface
  getUser(id: number): Observable<User> {
    const soapRequest = this.buildSOAPRequest('GetUser', { id });
    
    return this.http.post('/legacy/soap', soapRequest, {
      headers: { 'Content-Type': 'text/xml' }
    }).pipe(
      map(response => this.parseSOAPResponse(response)),
      map(data => this.transformToModernFormat(data))
    );
  }
  
  private buildSOAPRequest(method: string, params: any): string {
    return `
      <?xml version="1.0"?>
      <soap:Envelope>
        <soap:Body>
          <${method}>
            ${Object.entries(params).map(([key, value]) => 
              `<${key}>${value}</${key}>`
            ).join('')}
          </${method}>
        </soap:Body>
      </soap:Envelope>
    `;
  }
}
```

**4. Dual-Write Strategy:**
```typescript
// Write to both systems during transition

@Injectable({ providedIn: 'root' })
export class DualWriteService {
  saveUser(user: User): Observable<User> {
    return forkJoin({
      legacy: this.legacyApi.saveUser(user),
      modern: this.modernApi.saveUser(user)
    }).pipe(
      map(({ modern }) => modern),
      catchError(error => {
        // If one fails, rollback both
        return this.rollback(user).pipe(
          switchMap(() => throwError(() => error))
        );
      })
    );
  }
  
  private rollback(user: User): Observable<void> {
    return forkJoin({
      legacyRollback: this.legacyApi.deleteUser(user.id),
      modernRollback: this.modernApi.deleteUser(user.id)
    }).pipe(map(() => undefined));
  }
}
```

**5. Feature Flags:**
```typescript
// Toggle features between old and new systems

@Injectable({ providedIn: 'root' })
export class FeatureFlagService {
  private flags = new BehaviorSubject<FeatureFlags>({
    useNewUserManagement: false,
    useNewReports: true,
    useNewTransactions: false
  });
  
  isEnabled(feature: string): Observable<boolean> {
    return this.flags.pipe(
      map(flags => flags[feature] || false)
    );
  }
}

// Usage in component
@Component({...})
export class UserManagementComponent {
  constructor(
    private featureFlags: FeatureFlagService,
    private legacyService: LegacyUserService,
    private modernService: ModernUserService
  ) {}
  
  getUsers(): Observable<User[]> {
    return this.featureFlags.isEnabled('useNewUserManagement').pipe(
      switchMap(useNew => 
        useNew ? this.modernService.getUsers() : this.legacyService.getUsers()
      )
    );
  }
}
```

**6. Data Validation:**
```typescript
@Injectable({ providedIn: 'root' })
export class DataValidationService {
  validateMigration(userId: number): Observable<ValidationResult> {
    return forkJoin({
      legacy: this.legacyApi.getUser(userId),
      modern: this.modernApi.getUser(userId)
    }).pipe(
      map(({ legacy, modern }) => this.compareData(legacy, modern))
    );
  }
  
  private compareData(legacy: any, modern: User): ValidationResult {
    const discrepancies = [];
    
    if (legacy.USER_ID !== modern.id) {
      discrepancies.push({ field: 'id', legacy: legacy.USER_ID, modern: modern.id });
    }
    
    // Compare all fields...
    
    return {
      isValid: discrepancies.length === 0,
      discrepancies
    };
  }
}
```

**Migration Checklist:**
- [ ] Data backup and rollback plan
- [ ] Parallel run period (4-6 weeks)
- [ ] User training and documentation
- [ ] Performance testing
- [ ] Security audit
- [ ] Data validation scripts
- [ ] Monitoring and alerting
- [ ] Gradual rollout (10% → 50% → 100%)
- [ ] Legacy system decommissioning plan

---

## Additional Preparation Tips

### Company-Specific Research

**BNY Mellon:**
- Focus on financial domain knowledge
- Understand securities and investment management
- Review compliance and regulatory requirements
- Emphasize security and data protection

**LTI Mindtree:**
- Known for consulting approach
- Emphasize problem-solving methodology
- Demonstrate communication skills
- Show ability to work with cross-functional teams

### Common Interview Patterns

1. **Technical Deep Dive** (45-60 min)
   - Core Angular concepts
   - System design
   - Code review scenarios

2. **Coding Challenge** (60-90 min)
   - Live coding or take-home assignment
   - Focus on clean code, testing, documentation

3. **Leadership Discussion** (30-45 min)
   - Mentoring experience
   - Conflict resolution
   - Technical decision-making

4. **Behavioral** (30 min)
   - STAR method (Situation, Task, Action, Result)
   - Team collaboration
   - Handling pressure

### Key Points to Emphasize

- **6+ Years Experience:** Share specific examples from past projects
- **Financial Services:** Any domain knowledge or willingness to learn
- **Mentorship:** Concrete examples of helping junior developers
- **Architecture:** Decisions you've made and their outcomes
- **Problem Solving:** How you approach complex technical challenges
- **Communication:** Ability to explain technical concepts clearly

### Questions to Ask Interviewer

1. "What does the tech stack look like for this project?"
2. "How is the team structured? How many frontend developers?"
3. "What are the biggest technical challenges the team is facing?"
4. "How do you approach mentorship and knowledge sharing?"
5. "What does success look like for this role in the first 3-6 months?"
6. "How do you balance technical debt with feature development?"
7. "What's the deployment process and release cycle?"

---

**Good luck with your interview! 🚀**

Remember:
- Be confident but humble
- It's okay to say "I don't know, but here's how I'd find out"
- Ask clarifying questions before answering
- Think out loud during problem-solving
- Show enthusiasm for learning and growth

