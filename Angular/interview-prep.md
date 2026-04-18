# Angular — Interview Prep

---

## 1. Angular Overview

**Q: What is Angular and how is it different from React?**

Angular is a full-featured **opinionated framework** built by Google, written in TypeScript. React is a UI library — you choose your own router, state management, forms library, etc. Angular comes with everything built in: routing, HTTP client, forms, animations, dependency injection, testing utilities.

| | Angular | React |
|---|---|---|
| Type | Full framework | UI library |
| Language | TypeScript (required) | JavaScript/TypeScript |
| Data binding | Two-way (forms) + one-way | One-way |
| Dependency Injection | Built-in, first-class | External (Context, libraries) |
| Change detection | Zone.js / Signals | Virtual DOM diffing |
| Learning curve | Steeper | Gentler |
| Best for | Large enterprise apps with teams | Flexible, broad use cases |

Angular's opinions reduce decision fatigue in large teams — everyone uses the same patterns.

---

**Q: What is the Angular architecture at a high level?**

An Angular app is a tree of **modules** (NgModules) containing **components**, **services**, **directives**, and **pipes**.

```
AppModule (root)
├── AppComponent
│   ├── NavbarComponent
│   └── RouterOutlet → (lazy-loaded feature modules)
│       ├── HomeModule → HomeComponent
│       └── UserModule → UserComponent, UserService
```

- **Modules** — group related features, declare components/directives/pipes, import other modules.
- **Components** — control a part of the UI (template + logic + styles).
- **Services** — business logic and data access, shared via DI.
- **Directives** — extend HTML behavior.
- **Pipes** — transform values in templates.
- **Router** — maps URLs to component trees.

---

## 2. Components

**Q: What makes up an Angular component?**

A component has three parts:

```typescript
@Component({
    selector: 'app-user',           // how you use it: <app-user>
    templateUrl: './user.component.html',  // or template: `...`
    styleUrls: ['./user.component.scss']   // or styles: [`...`]
})
export class UserComponent implements OnInit {
    user: User;

    constructor(private userService: UserService) {}

    ngOnInit(): void {
        this.userService.getUser().subscribe(u => this.user = u);
    }
}
```

- **`@Component` decorator** — metadata telling Angular how to instantiate and render the component.
- **Class** — logic, state, lifecycle methods.
- **Template** — HTML with Angular syntax (`{{ }}`, `*ngIf`, `*ngFor`, `(click)`, `[src]`).
- **Styles** — scoped to this component by default (view encapsulation).

---

**Q: What is the component lifecycle? Which hooks are most important?**

Angular calls lifecycle hooks in this order:

| Hook | When it runs |
|---|---|
| `ngOnChanges` | Input property changes (before ngOnInit, and whenever inputs change) |
| `ngOnInit` | After first ngOnChanges; component initialized. **Most used — do init work here, not in constructor** |
| `ngDoCheck` | Every change detection cycle (use carefully — runs often) |
| `ngAfterContentInit` | After projected content (`<ng-content>`) initialized |
| `ngAfterContentChecked` | After projected content checked |
| `ngAfterViewInit` | After component's view (and child views) initialized. **Use for DOM access via ViewChild** |
| `ngAfterViewChecked` | After component's view checked |
| `ngOnDestroy` | Before Angular destroys the component. **Unsubscribe here to avoid memory leaks** |

**Constructor vs ngOnInit:** The constructor is for DI only — services are injected there. `ngOnInit` is where you do initialization work (load data, set up subscriptions). At constructor time, inputs are not yet bound.

---

**Q: What is `@Input` and `@Output`?**

These decorators enable parent-child communication:

```typescript
// Child component
@Component({ selector: 'app-card', ... })
export class CardComponent {
    @Input() title: string;           // receives data from parent
    @Input() user: User;

    @Output() selected = new EventEmitter<User>();  // sends events to parent

    select() {
        this.selected.emit(this.user);
    }
}
```

```html
<!-- Parent template -->
<app-card
    [title]="cardTitle"           <!-- property binding → @Input -->
    [user]="currentUser"
    (selected)="onUserSelected($event)">  <!-- event binding → @Output -->
</app-card>
```

`@Input()` = props going **in**. `@Output()` + `EventEmitter` = events going **out**.

---

## 3. Data Binding

**Q: What are the four types of data binding in Angular?**

```html
<!-- 1. Interpolation — component → template (display value) -->
<p>{{ user.name }}</p>
<p>{{ 2 + 2 }}</p>

<!-- 2. Property binding — component → DOM property -->
<img [src]="user.avatarUrl">
<button [disabled]="isLoading">Submit</button>

<!-- 3. Event binding — DOM → component -->
<button (click)="handleClick()">Click me</button>
<input (input)="onInput($event)">
<form (ngSubmit)="onSubmit()">

<!-- 4. Two-way binding — sync component ↔ input (requires FormsModule) -->
<input [(ngModel)]="username">
<!-- [(ngModel)] = [ngModel] + (ngModelChange) — shorthand for both directions -->
```

Two-way binding is shorthand for property + event binding combined. `[(x)]` is called "banana in a box" syntax.

---

**Q: What is the difference between `[property]`, `(event)`, and `{{interpolation}}`?**

- `{{ value }}` — interpolation, converts to string and displays.
- `[property]="expr"` — evaluates `expr` as JavaScript and sets the DOM property. Use this over `attr.="..."` for dynamic values.
- `(event)="handler()"` — attaches an event listener.
- `[attr.aria-label]="expr"` — binds to an HTML attribute (not a DOM property) — needed for accessibility attributes, SVG, etc.

---

## 4. Directives

**Q: What are the three types of directives?**

1. **Component** — a directive with a template. Every component is a directive.

2. **Structural directives** — change the DOM structure (add/remove elements). Marked with `*`.
```html
<div *ngIf="isLoggedIn; else loginBlock">Welcome!</div>
<ng-template #loginBlock><p>Please log in</p></ng-template>

<li *ngFor="let item of items; let i = index; trackBy: trackById">
    {{ i }}: {{ item.name }}
</li>

<div [ngSwitch]="status">
    <p *ngSwitchCase="'active'">Active</p>
    <p *ngSwitchDefault>Inactive</p>
</div>
```

3. **Attribute directives** — change the appearance or behavior of an element without adding/removing it.
```html
<p [ngClass]="{ 'highlight': isActive, 'disabled': !isActive }">Text</p>
<div [ngStyle]="{ color: textColor, fontSize: fontSize + 'px' }">Styled</div>
```

**Custom attribute directive:**
```typescript
@Directive({ selector: '[appHighlight]' })
export class HighlightDirective {
    @HostListener('mouseenter') onEnter() {
        this.el.nativeElement.style.background = 'yellow';
    }
    constructor(private el: ElementRef) {}
}
```

---

**Q: What is `trackBy` in `*ngFor` and why does it matter?**

By default, when `*ngFor` re-renders a list, it destroys and recreates all DOM elements whenever the array reference changes — even if only one item changed.

`trackBy` tells Angular how to identify items. Angular only updates the DOM for items that actually changed.

```typescript
// Component
trackById(index: number, item: Item): number {
    return item.id;
}
```
```html
<li *ngFor="let item of items; trackBy: trackById">{{ item.name }}</li>
```

Without `trackBy`: list of 1000 items → 1000 DOM operations on every sort/filter.  
With `trackBy`: only changed items → potentially 0-few DOM operations.

---

## 5. Services & Dependency Injection

**Q: What is a service in Angular?**

A service is a class annotated with `@Injectable` that holds **business logic, data access, or shared state** — anything that doesn't belong in a component. Services are provided once (singleton by default) and injected wherever needed via Angular's DI system.

```typescript
@Injectable({
    providedIn: 'root'  // singleton, available app-wide, tree-shakeable
})
export class UserService {
    private users: User[] = [];

    constructor(private http: HttpClient) {}

    getUsers(): Observable<User[]> {
        return this.http.get<User[]>('/api/users');
    }
}
```

`providedIn: 'root'` is the preferred way — it registers the service with the root injector and enables tree-shaking (if unused, bundler removes it).

---

**Q: How does Angular's dependency injection work?**

Angular has a **hierarchical injector tree** — each module, component, and directive has its own injector that can override parent-provided services.

```
Root Injector (providedIn: 'root')
└── Module Injector (providers: [] in NgModule)
    └── Component Injector (@Component providers: [])
        └── Child Component Injector
```

When a component requests a service in its constructor, Angular walks up the injector tree until it finds a provider. This means:
- `providedIn: 'root'` → one instance shared everywhere
- Providing a service in a component's `providers: []` → a **new instance** scoped to that component and its children

```typescript
// Component-scoped service — fresh instance per component
@Component({
    selector: 'app-form',
    providers: [FormStateService]  // not singleton — unique per instance
})
```

---

## 6. Pipes

**Q: What is a pipe and how do you use one?**

A pipe transforms a value in a template without modifying the underlying data. Angular provides many built-in pipes.

```html
{{ birthday | date:'mediumDate' }}              <!-- Mar 15, 2024 -->
{{ price | currency:'USD':'symbol':'1.2-2' }}   <!-- $1,234.57 -->
{{ name | uppercase }}                          <!-- ALICE -->
{{ description | slice:0:100 }}                 <!-- first 100 chars -->
{{ data | json }}                               <!-- pretty-print object (debugging) -->
{{ obs$ | async }}                              <!-- subscribe + auto-unsubscribe -->
```

**Custom pipe:**
```typescript
@Pipe({ name: 'truncate' })
export class TruncatePipe implements PipeTransform {
    transform(value: string, limit: number = 50): string {
        return value.length > limit ? value.slice(0, limit) + '...' : value;
    }
}
```
```html
{{ article.body | truncate:100 }}
```

---

**Q: What is the `async` pipe and why is it important?**

The `async` pipe subscribes to an Observable or Promise and returns the latest value. Crucially, it **automatically unsubscribes when the component is destroyed** — eliminating a common source of memory leaks.

```typescript
// Component
users$ = this.userService.getUsers();  // Observable<User[]>
```
```html
<!-- Template -->
<div *ngIf="users$ | async as users; else loading">
    <li *ngFor="let user of users">{{ user.name }}</li>
</div>
<ng-template #loading>Loading...</ng-template>
```

Without `async`, you'd subscribe manually in `ngOnInit` and have to unsubscribe in `ngOnDestroy`. `async` handles all of that for you.

---

## 7. RxJS & Observables

**Q: What is an Observable? How is it different from a Promise?**

| | Observable | Promise |
|---|---|---|
| Values | Multiple values over time | Single value |
| Lazy | Yes — nothing runs until subscribed | Eager — runs immediately |
| Cancellable | Yes — unsubscribe | No |
| Operators | Yes — `map`, `filter`, `mergeMap`, etc. | Chained `.then()` only |
| Use in Angular | HttpClient, Router events, forms | Rare |

```typescript
// Observable — lazy, can emit many values
const obs = new Observable(observer => {
    observer.next(1);
    observer.next(2);
    setTimeout(() => observer.next(3), 1000);
    setTimeout(() => observer.complete(), 2000);
});

obs.subscribe({
    next: value => console.log(value),
    error: err => console.error(err),
    complete: () => console.log("done")
});
```

Angular's `HttpClient` returns Observables — they complete after one response and are cold (don't run until subscribed).

---

**Q: What are the most important RxJS operators and when do you use them?**

```typescript
import { map, filter, switchMap, mergeMap, catchError, debounceTime, takeUntil } from 'rxjs/operators';

// map — transform each emitted value
this.http.get<User[]>('/api/users').pipe(
    map(users => users.filter(u => u.active))
);

// switchMap — map to inner Observable, cancel previous if new emission arrives
// Perfect for search: only care about the latest query's results
this.searchControl.valueChanges.pipe(
    debounceTime(300),            // wait 300ms after last keystroke
    switchMap(query => this.search(query))   // cancel previous request
);

// mergeMap — map to inner Observable, keep all concurrent subscriptions
// Good when order doesn't matter (parallel API calls)

// catchError — handle errors and recover
this.http.get('/api/data').pipe(
    catchError(err => of([]))   // return empty array on error
);

// takeUntil — complete when another Observable emits (clean unsubscription)
private destroy$ = new Subject<void>();
ngOnDestroy() { this.destroy$.next(); this.destroy$.complete(); }

this.someObs.pipe(
    takeUntil(this.destroy$)   // auto-unsubscribe on component destroy
).subscribe(...);
```

**`switchMap` vs `mergeMap` vs `concatMap`:**
- `switchMap` — cancel previous, only latest matters (search, navigation)
- `mergeMap` — run all concurrently, don't cancel (parallel saves)
- `concatMap` — queue and run sequentially (order-sensitive operations)

---

## 8. Forms

**Q: Template-driven vs Reactive forms — what's the difference?**

| | Template-driven | Reactive |
|---|---|---|
| Setup | Simple, minimal TypeScript | More TypeScript, explicit setup |
| Validation | HTML attributes + directives | Functions in TypeScript |
| Testing | Harder (need DOM) | Easier (pure TS) |
| Complex scenarios | Cumbersome | Handles well |
| Best for | Simple forms | Complex, dynamic, or heavily validated forms |

**Reactive forms:**
```typescript
this.form = this.fb.group({
    email: ['', [Validators.required, Validators.email]],
    password: ['', [Validators.required, Validators.minLength(8)]],
});

get email() { return this.form.get('email'); }
```
```html
<form [formGroup]="form" (ngSubmit)="onSubmit()">
    <input formControlName="email">
    <div *ngIf="email.invalid && email.touched">Invalid email</div>
    <button [disabled]="form.invalid">Submit</button>
</form>
```

---

## 9. Change Detection

**Q: How does Angular's change detection work?**

By default, Angular runs **change detection on every component** after any asynchronous event (DOM events, HTTP responses, timers) — using Zone.js to intercept these. It walks the component tree from the root and checks every component's bindings.

**`ChangeDetectionStrategy.OnPush`** — a powerful optimization. A component with `OnPush` is only checked when:
- One of its `@Input` references changes (new object reference, not just mutated value)
- An event originates from within the component
- An Observable via the `async` pipe emits a new value
- You manually call `markForCheck()`

```typescript
@Component({
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserCardComponent {
    @Input() user: User;  // must pass new object reference to trigger update
}
```

`OnPush` is the recommended default for performance. It forces immutable data patterns (which are good anyway).

---

## 10. Routing & Lazy Loading

**Q: What is lazy loading and why does it matter?**

By default, Angular bundles everything into one large file. **Lazy loading** splits the app into feature modules that are only downloaded when the user navigates to that route.

```typescript
const routes: Routes = [
    { path: '', component: HomeComponent },
    {
        path: 'admin',
        loadChildren: () => import('./admin/admin.module').then(m => m.AdminModule)
    },
    {
        path: 'user',
        loadChildren: () => import('./user/user.module').then(m => m.UserModule)
    }
];
```

The browser only downloads the admin bundle when the user first navigates to `/admin`. This improves initial load time — critical for large apps.

**Route guards** control access:
```typescript
@Injectable({ providedIn: 'root' })
export class AuthGuard implements CanActivate {
    canActivate(): boolean {
        return this.authService.isLoggedIn()
            ? true
            : (this.router.navigate(['/login']), false);
    }
}

{ path: 'admin', canActivate: [AuthGuard], loadChildren: ... }
```

---

## 11. Thought-Provoking Questions

**Q: If you mutate an object passed as `@Input` to an `OnPush` component, why doesn't the view update?**

`OnPush` compares input references with `===`. Mutating an object doesn't change its reference — Angular sees the same object and skips change detection for that component. You must pass a **new object** (`{ ...existing, updated: value }`) to trigger the check. This is why `OnPush` and immutability go hand in hand.

---

**Q: Why do you need to unsubscribe from Observables, and what happens if you don't?**

HTTP Observables complete automatically — no unsubscribe needed. But Observables from `Subject`, `interval`, router events, or form value changes don't complete on their own. If you subscribe in `ngOnInit` and never unsubscribe, the subscription keeps running even after the component is destroyed — the callback still fires, may try to update destroyed component state, and the component can't be garbage collected. This is a **memory leak**.

Solutions: `takeUntil(this.destroy$)`, `async` pipe (handles it automatically), `takeUntilDestroyed()` (Angular 16+).

---

**Q: What's the difference between `Subject`, `BehaviorSubject`, and `ReplaySubject`?**

All three are Observables that you can also push values into (multicast).

- **`Subject`** — no initial value, new subscribers only get future emissions.
- **`BehaviorSubject(initialValue)`** — holds the current value, new subscribers immediately get the latest value. Best for state that has a present value (e.g., current user).
- **`ReplaySubject(n)`** — buffers the last `n` emissions and replays them to new subscribers.

```typescript
const subject = new BehaviorSubject<User | null>(null);
subject.next(currentUser);

// New subscriber immediately gets currentUser
subject.subscribe(user => console.log(user));

// Read current value synchronously
const user = subject.getValue();
```

`BehaviorSubject` is the most common — it's the foundation for simple state stores in Angular services.
