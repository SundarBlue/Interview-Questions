# Angular & RxJS Interview Questions

## 1. Change Detection Strategies
**Concept:** How Angular checks for changes to update the DOM.
- **Default (`CheckAlways`)**: Checks every component in the tree from top to bottom on every event (click, http, timer).
- **OnPush**: Checks the component *only* if:
  1. Input reference changes (Immutability is key).
  2. An event originated from the component or its children.
  3. `markForCheck()` is manually called.
  4. `async` pipe emits a new value.

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

## 2. Dependency Injection Decorators (@Self, @SkipSelf, @Optional)
**Concept:** These control how the DI system resolves dependencies.
- **`@Self()`**: Looks for the dependency *only* in the current component's injector. Throws error if not found.
- **`@SkipSelf()`**: Skips the current injector and looks in the parent injector(s).
- **`@Optional()`**: Does not throw an error if the dependency is not found; returns `null`.

**Example:**
```typescript
constructor(
  @Self() private serviceA: MyService, 
  @Optional() @SkipSelf() private parentService: MyService
) {}
```

## 3. RxJS Operators: `takeUntil` vs `takeUntilDestroyed`
**Concept:** Managing subscription leaks.
- **`takeUntil(notifier$)`**: Emits values until the `notifier$` Observable emits. Commonly used with a `destroy$` Subject in `ngOnDestroy`.
- **`takeUntilDestroyed`** (Angular 16+): An operator that automatically completes the observable when the current context (component/directive) is destroyed. Requires injection context or passing `DestroyRef`.

**Example:**
```typescript
// Old Pattern
private destroy$ = new Subject<void>();
data$.pipe(takeUntil(this.destroy$)).subscribe();
ngOnDestroy() { this.destroy$.next(); }

// New Pattern (Angular 16+)
data$.pipe(takeUntilDestroyed()).subscribe();
```

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

## 7. Chaining API Calls (Flattening Operators)
**Concept:** When an API call depends on the result of another, use flattening operators (`switchMap`, `mergeMap`, `concatMap`, `exhaustMap`) instead of nested subscriptions.

- **`switchMap`**: Cancels previous inner observable (good for search).
- **`mergeMap`**: Runs all in parallel (good for independent saves).
- **`concatMap`**: Runs in sequence (good for order-sensitive operations).

**Example:**
```typescript
this.route.params.pipe(
  switchMap(params => this.apiService.getUser(params['id'])),
  switchMap(user => this.apiService.getPosts(user.id))
).subscribe(posts => {
  // Handle posts
});
```

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

## 9. Handling Large Lists (Virtual Scrolling)
**Concept:** Rendering 100,000 items (1 lakh) in the DOM will crash the browser. Use **Virtual Scrolling** (CDK Virtual Scroll) to render only the items currently visible in the viewport.

**Example:**
```html
<cdk-virtual-scroll-viewport itemSize="50" class="example-viewport">
  <div *cdkVirtualFor="let item of items" class="example-item">{{item}}</div>
</cdk-virtual-scroll-viewport>
```

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

## 11. Handling Multiple API Failures
**Concept:**
- **`forkJoin`**: If one fails, everything fails. Use `catchError` on *inner* observables to prevent the main stream from dying.
- **`combineLatest`**: Similar behavior, needs error handling on sources.

**Example (Resilient forkJoin):**
```typescript
forkJoin([
  this.api.getA().pipe(catchError(err => of(null))), // Return null on fail
  this.api.getB().pipe(catchError(err => of(null)))
]).subscribe(([resA, resB]) => {
  if (!resA) console.log('API A failed');
  // Continue processing
});
```

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
```
