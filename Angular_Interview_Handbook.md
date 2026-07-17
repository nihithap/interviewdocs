# Angular Interview Handbook
### A complete Q&A prep guide covering core Angular concepts

---

## Table of Contents

1. Angular Architecture & Bootstrap
2. Components & Templates
3. Data Binding
4. Directives
5. Pipes
6. Dependency Injection
7. Services & Providers
8. Component Lifecycle Hooks
9. Change Detection (Zone.js & OnPush)
10. Angular Signals
11. RxJS & Observables in Angular
12. Routing & Navigation
13. Forms (Template-driven vs Reactive)
14. HTTP Client & Interceptors
15. State Management
16. Modules vs Standalone Components
17. Performance Optimization
18. Testing
19. Security
20. SSR / Angular Universal & Hydration
21. Build, CLI & Deployment

---

# 1. Angular Architecture & Bootstrap

**Q1: What happens when an Angular application bootstraps?**

A:
1. `main.ts` calls `bootstrapApplication(AppComponent, config)` (standalone, Angular 14+) or historically `platformBrowserDynamic().bootstrapModule(AppModule)`.
2. Angular creates the **injector hierarchy** — root providers (from `providers` in `bootstrapApplication`, or `AppModule`) are instantiated lazily as needed.
3. Angular compiles the component tree (AOT — Ahead-of-Time — by default in production builds; the Ivy compiler generates instruction-based render functions per component).
4. The root component (`AppComponent`) is instantiated, its template is rendered, `ngOnInit` and other lifecycle hooks fire, and it's inserted into the DOM at the matching selector element (e.g., `<app-root>`).
5. **Zone.js** (if used) patches async browser APIs (`setTimeout`, `addEventListener`, Promises, XHR) so Angular knows when to run change detection.

**Q2: What is Ivy, and what changed with it (vs the older View Engine)?**

A: Ivy is Angular's rendering/compilation pipeline (default since Angular 9). Key differences:
- **Locality** — each component compiles independently into its own set of instructions, rather than View Engine's global "one metadata file describes everything" model. This enables faster incremental builds.
- **Tree-shakeable** — unused directives/components/Angular features aren't included in the bundle; smaller output.
- **`ngcc` (Angular Compatibility Compiler)** was introduced to convert View-Engine-format libraries to Ivy format (largely no longer needed as the ecosystem moved to Ivy).
- Enabled features like **higher-order/dynamic component creation**, better debugging (Ivy instructions map more directly to readable stack traces), and eventually standalone components.

**Q3: Explain the injector hierarchy — root, module, and element injectors.**

A: Angular has a **hierarchical DI system**:
- **Root injector** — created at bootstrap; services `providedIn: 'root'` live here as app-wide singletons.
- **Platform injector** — sits above root, shared across multiple Angular apps on the same page (rare, mostly for micro-frontend setups).
- **Element injectors** — created per component/directive instance based on their `providers`/`viewProviders` arrays, forming a tree that mirrors the component tree. A service provided at a component level gets a **new instance per component instance**, shadowing any same-token provider higher up.

Resolution walks **up** the element injector tree first, then falls back to the module/root injector if not found locally.

---

# 2. Components & Templates

**Q1: What's the difference between a Component and a Directive?**

A: A **Component** is a directive **with a template** (`@Component` extends the concept of `@Directive` by adding `template`/`templateUrl`, styles, and view encapsulation). A plain **Directive** (`@Directive`) has no template of its own — it modifies the behavior or appearance of an existing host element (attribute directives) or manipulates the DOM structure around a `<ng-template>` (structural directives).

**Q2: What are the View Encapsulation modes?**

A:
- **Emulated** (default) — Angular adds unique attributes to elements and scopes CSS selectors to those attributes, simulating style isolation without native Shadow DOM.
- **ShadowDom** — uses the browser's native Shadow DOM API for true style/DOM encapsulation.
- **None** — no encapsulation; styles are global and can leak in/out of the component.

**Q3: What's `@ViewChild` vs `@ContentChild`?**

A: `@ViewChild` queries an element/component/directive defined in the component's **own template** (its view). `@ContentChild` queries an element **projected into** the component via `<ng-content>` (i.e., content passed by the parent). Both have `static` option (query resolves before `ngOnInit` if `static: true`, or after view init otherwise) and plural variants `@ViewChildren`/`@ContentChildren` returning a `QueryList`.

**Q4: What is `ng-content` and how does multi-slot content projection work?**

A: `<ng-content>` is a placeholder where a parent component's projected markup gets inserted. Multi-slot projection uses the `select` attribute to route different projected elements to different slots based on a CSS selector: `<ng-content select="[header]"></ng-content>` picks up only elements marked `header`, while a plain `<ng-content>` (no select) catches everything else not matched by a specific selector.

**Q5: What's the difference between `<ng-template>`, `<ng-container>`, and `<ng-content>`?**

A:
- **`<ng-template>`** — defines a template block that is **not rendered by default**; it's only rendered when explicitly instantiated (by a structural directive like `*ngIf`/`*ngFor` under the hood, or manually via `ViewContainerRef.createEmbeddedView()`).
- **`<ng-container>`** — a logical grouping element that renders **no actual DOM element** — useful for applying a structural directive to a group of elements without introducing an extra wrapper `<div>`.
- **`<ng-content>`** — the content projection slot described above.

---

# 3. Data Binding

**Q1: List Angular's data binding types with syntax.**

A:
- **Interpolation**: `{{ value }}` — one-way, component → view.
- **Property binding**: `[property]="value"` — one-way, component → view (DOM property, not attribute).
- **Event binding**: `(event)="handler($event)"` — one-way, view → component.
- **Two-way binding**: `[(ngModel)]="value"` — sugar combining property + event binding (`banana in a box` syntax); works with any component implementing an `@Input() x` + `@Output() xChange` pair.

**Q2: What's the difference between property binding and attribute binding?**

A: Property binding (`[value]="x"`) sets a **DOM property** directly on the element object (e.g., `inputEl.value = x`). Attribute binding (`[attr.aria-label]="x"`) sets an **HTML attribute**. Most cases use property binding since it's faster and type-checked, but some things only exist as attributes (no corresponding DOM property) — like `aria-*`, `colspan`, or custom/SVG attributes — which require `attr.` binding.

**Q3: How do you implement two-way binding on a custom component?**

A: Define an `@Input()` and a matching `@Output()` named `<inputName>Change`:
```typescript
@Input() value!: string;
@Output() valueChange = new EventEmitter<string>();

updateValue(v: string) {
  this.value = v;
  this.valueChange.emit(v);
}
```
Then consumers can use `[(value)]="parentValue"`, which Angular desugars to `[value]="parentValue" (valueChange)="parentValue = $event"`.

---

# 4. Directives

**Q1: Structural vs Attribute directives — what's the difference?**

A: **Structural directives** (`*ngIf`, `*ngFor`, `*ngSwitch`, or a custom one) change the DOM **structure** — adding/removing elements — by manipulating a `<ng-template>` via `ViewContainerRef` and `TemplateRef`. The `*` prefix is syntactic sugar Angular desugars into an `<ng-template>` wrapper. **Attribute directives** (`ngClass`, `ngStyle`, or custom) change the **appearance or behavior** of an existing element without adding/removing DOM nodes.

**Q2: How would you write a custom attribute directive (e.g., a highlight-on-hover directive)?**

A:
```typescript
@Directive({
  selector: '[appHighlight]',
  standalone: true
})
export class HighlightDirective {
  private el = inject(ElementRef);

  @HostListener('mouseenter') onEnter() {
    this.el.nativeElement.style.backgroundColor = 'yellow';
  }
  @HostListener('mouseleave') onLeave() {
    this.el.nativeElement.style.backgroundColor = '';
  }
}
```
`@HostListener` binds to a DOM event on the host element; `@HostBinding` similarly binds a host element property/class/style to a directive property.

**Q3: What's the modern control-flow syntax (`@if`, `@for`, `@switch`) introduced in Angular 17, and how does it differ from `*ngIf`/`*ngFor`?**

A: Angular 17+ introduced **built-in control flow** as a template syntax feature (not a directive import):
```html
@if (user(); as u) {
  <p>Welcome, {{ u.name }}</p>
} @else {
  <p>Please log in</p>
}

@for (item of items(); track item.id) {
  <li>{{ item.name }}</li>
} @empty {
  <li>No items</li>
}
```
Benefits over `*ngIf`/`*ngFor`: no need to import `CommonModule`/`NgIf`/`NgFor` in standalone components, **mandatory `track` expression** in `@for` (forces you to think about identity, improving diffing performance vs the optional `trackBy` of `*ngFor`), better type narrowing, and measurably faster rendering since it's compiled more directly rather than going through a structural directive's view container abstraction.

**Q4: Why does `*ngFor` need a `trackBy` function, and what happens without one?**

A: Without `trackBy`, Angular's default tracking is **by object identity/reference**. If the array is replaced with a new array of new object references (e.g., after an immutable state update), Angular can't tell which items are "the same, just updated" vs "new" — it destroys and recreates the DOM nodes for the whole list, losing things like input focus, animation state, or component-internal state, and costing performance. `trackBy: (index, item) => item.id` lets Angular match by a stable identity, so it only patches what actually changed.

---

# 5. Pipes

**Q1: What are pipes, and what's the difference between pure and impure pipes?**

A: Pipes transform data for display in a template (`{{ value | pipeName:arg }}`). **Pure pipes** (default) only re-run when Angular detects a change in the **input reference** (primitive value change, or object/array reference change) — cheap, since they skip re-execution on every change detection cycle. **Impure pipes** (`pure: false`) re-run on **every** change detection cycle regardless of whether the input actually changed — necessary for pipes that need to react to mutations within the same object/array reference (e.g., filtering a mutated array), but potentially expensive if overused.

**Q2: Why is filtering/sorting in a pipe often discouraged, and what's the alternative?**

A: A pipe used in a template re-runs on every change-detection cycle by default when impure (or whenever a reference changes for pure ones) — doing an expensive filter/sort operation there can hurt performance since it's easy to trigger far more often than intended, and it also mixes presentation logic into what should probably be more explicit data-transformation logic. The more idiomatic modern alternative is to keep the transformed/filtered list as a `computed()` signal (or a memoized observable pipeline) in the component, so it only recalculates when its actual dependencies change.

**Q3: How do you create a custom pipe?**

A:
```typescript
@Pipe({ name: 'truncate', standalone: true })
export class TruncatePipe implements PipeTransform {
  transform(value: string, limit = 20): string {
    return value.length > limit ? value.slice(0, limit) + '…' : value;
  }
}
```
Usage: `{{ longText | truncate:50 }}`.

---

# 6. Dependency Injection

**Q1: What are the ways to provide a service, and how do they differ in scope?**

A:
- **`providedIn: 'root'`** (on `@Injectable`) — app-wide singleton, tree-shakeable (excluded from the bundle entirely if never injected anywhere).
- **`providedIn: 'platform'`** — shared across multiple Angular apps on one page.
- **Component/directive `providers` array** — new instance scoped to that component's subtree (element injector).
- **Route-level `providers`** (in a route config, standalone routing) — scoped to that route and its children, useful for feature-specific singletons that shouldn't pollute the root injector.
- **`viewProviders`** — like `providers`, but not visible to content-projected children (only to the component's own view).

**Q2: What is an `InjectionToken`, and when do you need one?**

A: Classes can be used directly as DI tokens, but for things that **aren't classes** (interfaces, primitive config values, string constants) TypeScript interfaces don't exist at runtime, so you can't use them as a DI token. `InjectionToken<T>` creates a unique, type-safe token to register/inject such values:
```typescript
export const API_URL = new InjectionToken<string>('API_URL');
// provide:
{ provide: API_URL, useValue: 'https://api.example.com' }
// inject:
private apiUrl = inject(API_URL);
```

**Q3: Explain `useClass`, `useValue`, `useExisting`, and `useFactory` provider syntaxes.**

A:
- `useClass` — instantiate a given class when the token is requested (useful for swapping an implementation, e.g., providing a `MockAuthService` in tests in place of `AuthService`).
- `useValue` — provide a static, already-constructed value (config objects, constants).
- `useExisting` — alias one token to resolve to the **same instance** as another already-registered token (avoids creating a second instance).
- `useFactory` — provide a factory function (optionally with its own `deps`) that computes/returns the instance, useful when construction needs logic or runtime info (e.g., choosing an implementation based on environment).

**Q4: What is the `inject()` function, and how does it differ from constructor injection?**

A: `inject()` (Angular 14+) retrieves a dependency from the current injection context **without needing a constructor parameter** — usable in field initializers, factory functions, and functional guards/resolvers/interceptors. It relies on Angular temporarily setting an "active injector" during construction, so it must be called synchronously during that context (class field initializer, constructor, or a function explicitly run within an injection context via `runInInjectionContext`) — it cannot be called later, e.g., inside a `setTimeout` callback.

---

# 7. Services & Providers

**Q1: Why are services typically used for cross-component communication instead of `@Input`/`@Output` chains?**

A: `@Input`/`@Output` work well for direct parent-child communication, but become unwieldy ("prop drilling") across deeply nested or sibling components. A shared, injectable service (often exposing an RxJS `Subject`/`BehaviorSubject` or a `signal`) acts as a single source of truth that any component in scope can inject and subscribe to/read from, decoupling components from each other's position in the tree.

**Q2: How do you share state between sibling components with a service?**

A: Provide the service at a **common ancestor** (or `root` if truly global) so both siblings resolve to the same instance via the injector hierarchy, then expose state via a `BehaviorSubject`/signal with a public read-only accessor and methods to mutate it:
```typescript
@Injectable({ providedIn: 'root' })
export class CartService {
  private _items = signal<Item[]>([]);
  readonly items = this._items.asReadonly();
  addItem(item: Item) { this._items.update(items => [...items, item]); }
}
```

**Q3: What's a singleton pitfall to watch for with `providedIn: 'root'` in a lazy-loaded feature module scenario?**

A: Historically (pre-standalone, with `NgModule`-based lazy loading), providing a service in a lazy-loaded module's `providers` array (rather than `providedIn: 'root'`) created a **separate instance scoped to that lazy module's injector**, which could surprise developers expecting a single app-wide instance. With standalone components and the modern router, this is cleaner (route-level `providers` explicitly scope things), but it's still a common interview gotcha to understand: `providedIn: 'root'` is always a true singleton; putting a service in a module's/route's `providers` array creates a new instance for that scope.

---

# 8. Component Lifecycle Hooks

**Q1: List the lifecycle hooks in order of execution and what each is for.**

A:
1. **`ngOnChanges`** — called before `ngOnInit` and whenever a bound `@Input()` value changes (reference change for objects); receives a `SimpleChanges` object with previous/current values.
2. **`ngOnInit`** — called once, after the first `ngOnChanges`; the standard place for initialization logic (fetching initial data, setting up subscriptions) — not the constructor, since `@Input()` values aren't guaranteed to be set yet at construction time.
3. **`ngDoCheck`** — called on every change detection run; for custom change detection logic Angular's default checks wouldn't catch (rarely used, and expensive if not careful).
4. **`ngAfterContentInit`** — after content projected via `<ng-content>` has been initialized (first time `@ContentChild` queries are resolved).
5. **`ngAfterContentChecked`** — after every check of projected content.
6. **`ngAfterViewInit`** — after the component's own view (and child views) have been initialized (first time `@ViewChild` queries are resolved) — the right place to interact with view-queried elements.
7. **`ngAfterViewChecked`** — after every check of the component's view.
8. **`ngOnDestroy`** — called just before the component is destroyed; the place to unsubscribe from observables, clear timers/intervals, and detach event listeners to prevent memory leaks.

**Q2: Why is `ngOnInit` preferred over the constructor for initialization logic?**

A: The constructor should be reserved for **dependency injection only** — Angular hasn't yet set input bindings or initialized the component's data-bound properties at construction time. `ngOnInit` runs after the first round of input binding, guaranteeing `@Input()` values are populated, and separates "wiring dependencies" from "using them," which also makes constructor-based unit testing cleaner (you can construct the class without triggering business logic side effects).

**Q3: What's a common memory leak pattern related to lifecycle hooks, and how do you avoid it?**

A: Subscribing to an observable (e.g., in `ngOnInit`) without unsubscribing in `ngOnDestroy` leaks the subscription — if the observable is long-lived (a service-level Subject, a WebSocket stream, a router event stream), the component instance (and everything it references) is kept alive by that subscription even after the component is destroyed and removed from the DOM. Fixes:
- Manually track subscriptions and call `.unsubscribe()` in `ngOnDestroy`.
- Use the `async` pipe in the template, which auto-subscribes/unsubscribes with the component's lifecycle.
- Use `takeUntilDestroyed()` (Angular 16+, from `@angular/core/rxjs-interop`) to automatically complete the subscription tied to `DestroyRef`.
- Prefer signals for state that doesn't need the full RxJS operator toolkit — signals don't have a subscription-leak failure mode in the same way.

---

# 9. Change Detection (Zone.js & OnPush)

**Q1: How does Angular's default change detection work with Zone.js?**

A: Zone.js **monkey-patches** async browser APIs (`setTimeout`, `Promise.then`, DOM event listeners, XHR, fetch, etc.) so that whenever any of these fire, Angular knows "something might have changed" and triggers a change detection pass. In the default strategy, Angular walks the **entire component tree top-to-bottom**, checking every component's template bindings against their last known values (dirty-checking) and updating the DOM where they differ. This is simple and always-correct but can be wasteful at scale — checking many components that didn't actually change.

**Q2: How does `ChangeDetectionStrategy.OnPush` improve performance, and what triggers a check under it?**

A: `OnPush` tells Angular to **skip** checking a component (and its subtree) unless one of these occurs:
- An `@Input()` reference changes (primitive value change, or new object/array reference — **mutating** an object in place won't trigger it).
- An event originates from within the component's own template (a `(click)` handler, etc.).
- An observable bound via the `async` pipe emits.
- Change detection is manually triggered (`ChangeDetectorRef.markForCheck()` / `detectChanges()`), or a signal read in the template changes.

This is why OnPush requires **immutable data patterns** — you must replace objects/arrays rather than mutate them, or Angular won't detect the change (since it's comparing by reference, not deep equality).

**Q3: How does the new Signals-based reactivity model change (or reduce dependence on) Zone.js-driven change detection?**

A: Signals track their own dependents at a fine-grained level — when a signal's value changes, Angular knows **exactly** which specific template bindings depend on it and can update just those, without needing to walk/dirty-check the whole tree via Zone.js. This underpins Angular's move toward **zoneless change detection** (`provideExperimentalZonelessChangeDetection()` / now stabilizing) — with enough of the app driven by signals, Angular doesn't need Zone.js's blanket "something async happened, recheck everything" approach at all, reducing overhead and bundle size (Zone.js patching has real runtime cost).

**Q4: What's the difference between `markForCheck()` and `detectChanges()` on `ChangeDetectorRef`?**

A: `markForCheck()` flags the component (and marks the path from it up to the root as dirty) so that it **will** be included in the **next** change detection cycle — used with `OnPush` when you know data changed outside Angular's normal triggers (e.g., inside a raw WebSocket callback not patched by Zone). `detectChanges()` **immediately and synchronously** runs change detection on that component and its children right now, without waiting for/relying on the next cycle — useful in tests, or narrow cases needing an immediate DOM sync.

---

# 10. Angular Signals

**Q1: What problem do Signals solve, and how do they differ conceptually from RxJS Observables?**

A: Signals are a **synchronous, fine-grained reactive primitive** built into Angular core (stabilized Angular 17+) for representing state that the framework can track dependencies on automatically. Unlike Observables (which model **streams of events over time**, are lazy/subscription-based, and need explicit subscription management/operators), a Signal is more like a **reactive variable** — you read it by calling it (`count()`), and Angular's rendering automatically re-runs wherever that signal is read, with no subscribe/unsubscribe lifecycle to manage and no risk of the memory-leak-via-forgotten-subscription pattern.

**Q2: Explain `signal()`, `computed()`, and `effect()`.**

A:
```typescript
const count = signal(0);              // writable signal
count.set(5);
count.update(v => v + 1);

const doubled = computed(() => count() * 2);  // derived, read-only, memoized, lazy

effect(() => {
  console.log(`count is now ${count()}`);      // re-runs whenever a signal it reads changes
});
```
- **`signal()`** — a writable, observable value container.
- **`computed()`** — a **derived** signal, automatically recalculated (and memoized) only when its dependencies change; must be pure (no side effects).
- **`effect()`** — runs a side effect in response to signal changes (logging, syncing to localStorage, imperative DOM work); runs at least once immediately and then whenever a dependency changes. Effects should generally avoid writing to signals they also read, to prevent infinite loops.

**Q3: What are `input()`, `output()`, and `model()` signal-based APIs (Angular 17.1+/19), and how do they compare to decorator-based `@Input`/`@Output`?**

A: These are the newer **signal-based component API**, meant to eventually supersede the decorator forms:
```typescript
// New signal-based:
value = input<string>('');           // read-only signal input
valueRequired = input.required<string>();
changed = output<string>();          // typed event emitter, similar ergonomics to EventEmitter
model = model<string>('');           // combines input + output for two-way binding in one declaration
```
Benefits: `input()` returns an actual signal (composable with `computed()`/`effect()` directly, rather than needing `ngOnChanges` to react to changes), `model()` collapses the old `@Input()` + `@Output() xChange` two-way-binding boilerplate into a single declaration, and the whole API is more consistent with the rest of the signals ecosystem — while `@Input()`/`@Output()` decorators remain fully supported for backward compatibility.

**Q4: How do Signals interoperate with RxJS?**

A: `@angular/core/rxjs-interop` provides:
- **`toSignal(observable$)`** — converts an Observable into a Signal (useful for consuming HTTP calls or existing RxJS pipelines in a signals-based component without manual subscription management).
- **`toObservable(signal)`** — converts a Signal into an Observable (useful when you need RxJS operators like `debounceTime`/`switchMap` on top of signal-driven state).
- **`takeUntilDestroyed()`** — auto-unsubscribes an Observable subscription tied to the component/service's `DestroyRef`, simplifying manual cleanup in `ngOnDestroy`.

---

# 11. RxJS & Observables in Angular

**Q1: Where does Angular use Observables pervasively, and why (vs Promises)?**

A: `HttpClient` responses, the `Router`'s event stream and route params/data, reactive forms' `valueChanges`/`statusChanges`, and any custom event-stream service. Observables are preferred over Promises here because they support: **cancellation** (unsubscribing an in-flight HTTP request, e.g., when a component is destroyed or a newer request supersedes it), **multiple values over time** (not just a single resolved value), and a rich **operator toolkit** for composing/transforming async flows declaratively (debouncing, retrying, combining multiple streams).

**Q2: Explain the difference between `switchMap`, `mergeMap`, `concatMap`, and `exhaustMap`, with a use case for each.**

A:
- **`switchMap`** — cancels the previous inner observable when a new source value arrives; use for "only the latest matters" (e.g., a search-as-you-type autocomplete, where you want to cancel a stale in-flight request).
- **`mergeMap`** — runs all inner observables concurrently, merging results as they arrive; use when order doesn't matter and you want parallelism (e.g., firing off several independent uploads at once).
- **`concatMap`** — queues inner observables, running them strictly one after another in order; use when order/sequencing matters and requests must not overlap (e.g., sequential form-save operations that must apply in order).
- **`exhaustMap`** — ignores new source values while an inner observable is still running; use to prevent duplicate submissions (e.g., ignoring repeated clicks on a submit button while the first request is still in flight).

**Q3: What's the difference between `Subject`, `BehaviorSubject`, `ReplaySubject`, and `AsyncSubject`?**

A:
- **`Subject`** — no initial value, no memory; late subscribers only get values emitted **after** they subscribe.
- **`BehaviorSubject`** — requires an initial value, always holds and immediately emits the **current/latest** value to new subscribers — most common for representing "current state."
- **`ReplaySubject(n)`** — buffers and replays the last `n` emitted values to new subscribers.
- **`AsyncSubject`** — only emits the **final** value, and only upon completion — rarely used directly.

**Q4: How do you avoid memory leaks with manual subscriptions, beyond just calling `.unsubscribe()`?**

A: Prefer patterns that manage the lifecycle for you rather than manual bookkeeping:
- The **`async` pipe** in templates — Angular subscribes on render and unsubscribes automatically on destroy.
- **`takeUntilDestroyed()`** (signals-interop) tied to `DestroyRef`.
- For multiple subscriptions, a `Subscription` object accumulating via `.add()`, unsubscribed once in `ngOnDestroy`, instead of tracking many separate subscription variables.

---

# 12. Routing & Navigation

**Q1: How does route configuration work with standalone components (modern Angular)?**

A:
```typescript
export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'products/:id', component: ProductDetailComponent },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes').then(m => m.ADMIN_ROUTES),
    canActivate: [authGuard]
  }
];
// main.ts
bootstrapApplication(AppComponent, {
  providers: [provideRouter(routes)]
});
```
`loadChildren` (or `loadComponent` for a single lazy component) enables **lazy loading** — that route's code is fetched in a separate JS chunk only when the user navigates there, keeping the initial bundle smaller.

**Q2: What are route guards, and what does each type control?**

A:
- **`canActivate`** — controls whether a route can be entered at all (e.g., auth check).
- **`canActivateChild`** — same, but applied to child routes of the guarded route.
- **`canDeactivate`** — controls whether navigation **away** from the current route is allowed (e.g., "you have unsaved changes, are you sure?").
- **`canMatch`** — controls whether a route configuration is even considered a match during route resolution (useful for conditionally choosing between multiple route configs for the same path, e.g., feature-flagged routes).
- **`resolve`** — not strictly a guard, but pre-fetches data before the route activates, so the component doesn't render in a loading state for that initial data.

Modern Angular favors **functional guards** (plain functions using `inject()`) over the older class-based guard interfaces:
```typescript
export const authGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  const router = inject(Router);
  return auth.isLoggedIn() || router.parseUrl('/login');
};
```

**Q3: What's the difference between `routerLink` and calling `Router.navigate()` programmatically?**

A: `routerLink` is a directive for declarative navigation in templates (renders as a real anchor `href` when combined with `RouterLinkActive`/`routerLink`, supporting things like middle-click-to-open-in-new-tab natively). `Router.navigate(['/path'])` is imperative, used when navigation needs to happen as a result of logic (after a successful form submission, in response to an async event) rather than a direct user click on a link element. `Router.navigateByUrl()` takes a full URL string/tree instead of a path array.

**Q4: How do you access route parameters, and what's the difference between snapshot and observable access?**

A: `ActivatedRoute.snapshot.paramMap.get('id')` gives a **one-time** read of the current value — fine if the component is always destroyed/recreated on param change (default router behavior when navigating to a *different* route). But if the **same component instance is reused** across a param change (e.g., navigating from `/products/1` to `/products/2` where the route config matches the same component), the snapshot won't update — you need `ActivatedRoute.paramMap` as an **Observable**, subscribing (or using `toSignal`) to react to subsequent changes without the component being torn down and recreated.

---

# 13. Forms (Template-driven vs Reactive)

**Q1: Compare Template-driven and Reactive forms.**

A:
- **Template-driven** — form structure/validation defined largely in the **template** via directives (`ngModel`, `required`, `minlength`), with Angular building the form model behind the scenes. Simpler for basic forms, but harder to unit test (logic lives in the template) and less suited to complex/dynamic validation logic.
- **Reactive forms** — form structure defined explicitly in the **component class** using `FormGroup`/`FormControl`/`FormArray`, with the template just binding to that pre-built model (`formControlName`, `formGroup`). More verbose upfront, but far more testable, supports complex/dynamic validation and cross-field validators cleanly, and integrates naturally with RxJS (`valueChanges`, `statusChanges`) and Signals (`toSignal`).

Reactive forms are generally recommended for anything beyond the simplest forms.

**Q2: How do you write a custom validator, and a custom async validator?**

A:
```typescript
// Synchronous, cross-field validator
export function passwordsMatch(group: AbstractControl): ValidationErrors | null {
  const pass = group.get('password')?.value;
  const confirm = group.get('confirm')?.value;
  return pass === confirm ? null : { passwordsMismatch: true };
}

// Async validator (e.g., checking username availability against a server)
export function uniqueUsername(userService: UserService): AsyncValidatorFn {
  return (control: AbstractControl) =>
    userService.checkUsername(control.value).pipe(
      map(exists => (exists ? { taken: true } : null)),
      catchError(() => of(null))
    );
}
```
A validator returns `null` if valid, or an error object (`{ errorKey: details }`) if invalid. Async validators return an Observable/Promise and are only re-run after synchronous validators pass, typically debounced internally by the form control (`updateOn: 'blur'` or similar is often paired with async validators to avoid hammering the server on every keystroke).

**Q3: What's the difference between `FormControl`, `FormGroup`, and `FormArray`?**

A: `FormControl` wraps a single form value + its validation state. `FormGroup` is a keyed collection of controls (including nested groups), representing an object-shaped chunk of the form. `FormArray` is an ordered, indexable collection of controls (of potentially varying/dynamic count), used for repeatable form sections (e.g., a dynamic list of phone numbers where the user can add/remove entries).

**Q4: How would you disable a submit button until the form is valid, and show field-level errors only after the user has interacted with the field?**

A:
```html
<button [disabled]="form.invalid">Submit</button>

<div *ngIf="form.get('email')?.invalid && form.get('email')?.touched">
  Please enter a valid email.
</div>
```
`touched` becomes true after the control has been blurred at least once (vs `dirty`, which becomes true once the value has changed) — using `touched` avoids showing an error message immediately on page load before the user has even had a chance to type.

---

# 14. HTTP Client & Interceptors

**Q1: How do you make a typed HTTP GET request, and why does `HttpClient` return Observables instead of Promises?**

A:
```typescript
private http = inject(HttpClient);

getUsers(): Observable<User[]> {
  return this.http.get<User[]>('/api/users');
}
```
Returning Observables (rather than Promises, which `fetch` uses natively) gives cancellability (unsubscribing aborts the underlying XHR/fetch request — useful when a component is destroyed mid-request, or `switchMap` cancels a stale request), and composability with RxJS operators (retry, timeout, combining multiple requests) directly in the data-fetching pipeline.

**Q2: What are HTTP Interceptors, and what are common use cases?**

A: Interceptors sit in the request/response pipeline, able to inspect/modify outgoing requests and incoming responses (or errors) globally, without every call site needing to repeat that logic. Common uses:
- **Attaching auth tokens** to every outgoing request's headers.
- **Global error handling** (e.g., redirecting to login on a 401, showing a toast on 5xx).
- **Loading indicators** (increment/decrement a global "requests in flight" counter).
- **Logging/telemetry** (timing requests, correlation IDs).
- **Caching** GET responses.

```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).token();
  const cloned = token ? req.clone({ setHeaders: { Authorization: `Bearer ${token}` } }) : req;
  return next(cloned);
};
// registered via:
provideHttpClient(withInterceptors([authInterceptor]))
```
Note requests are **immutable** — you must `.clone()` a request to modify it rather than mutating it directly.

**Q3: How do you handle errors from an HTTP call, and retry transient failures?**

A:
```typescript
this.http.get<User[]>('/api/users').pipe(
  retry({ count: 3, delay: 1000 }),
  catchError(err => {
    console.error('Failed to load users', err);
    return of([]); // graceful fallback
  })
);
```
`catchError` must return a new Observable (often `of(fallbackValue)` or `throwError(() => err)` to re-propagate after logging) — it can't just log and fall through, since RxJS pipelines need an explicit resulting stream.

---

# 15. State Management

**Q1: What are the common approaches to state management in Angular, from simplest to most structured?**

A:
1. **Component state** (local `signal`/class fields) — for state only relevant to one component/subtree.
2. **Shared service with Signals/BehaviorSubject** — a `providedIn: 'root'` service exposing readonly state and mutation methods; the most common "lightweight state management" approach for small-to-medium apps.
3. **NgRx (Redux-pattern)** — centralized store, actions, reducers, selectors, effects (for async side effects) — provides strict unidirectional data flow, time-travel debugging, and strong conventions, at the cost of boilerplate; best suited to large apps with complex, cross-cutting state and teams that benefit from enforced structure.
4. **NgRx SignalStore / Component Store** — a lighter-weight, more ergonomic alternative to full NgRx, built on Signals, for feature-scoped state without the full global-store ceremony.
5. **Third-party alternatives** — Akita, Elf, or simply well-organized signal-based services, depending on team preference.

**Q2: In NgRx, explain the flow: action → reducer → selector → effect.**

A:
- **Action** — a plain object describing "what happened" (`{ type: '[Cart] Add Item', item }`), dispatched from components/services.
- **Reducer** — a pure function `(state, action) => newState` that computes the new state immutably in response to an action; the only place state actually changes.
- **Selector** — a memoized function that reads/derives a slice of state from the store for components to consume (`createSelector`), re-computing only when its inputs change.
- **Effect** — listens for actions and performs **side effects** (HTTP calls, logging) that reducers can't do (since reducers must be pure/synchronous); an effect typically dispatches a **new** action (e.g., a success/failure action) once the side effect completes, which then flows through a reducer to update state.

**Q3: When would you choose a simple signal-based service over full NgRx?**

A: For small-to-medium apps, or state that's genuinely local to a feature and doesn't need cross-cutting visibility, time-travel debugging, or a large team's need for enforced structure — a signal-based service is far less boilerplate, easier to onboard new developers to, and sufficiently reactive/testable on its own. NgRx earns its complexity in larger apps where many features read/write overlapping state, strict traceability of "what changed and why" matters (e.g., via Redux DevTools), or there's substantial async orchestration best modeled as effects.

---

# 16. Modules vs Standalone Components

**Q1: What problem do standalone components (Angular 14+, default since Angular 17) solve compared to `NgModule`-based architecture?**

A: `NgModule`s required every component/directive/pipe to be declared in exactly one module, and that module had to import whatever other modules provided the directives/pipes it used in templates — this indirection (needing to hunt through module `imports`/`declarations` to understand what's available where) added boilerplate and a learning-curve barrier, especially for beginners and for simple apps that didn't need the org-level grouping benefits. **Standalone components** (`standalone: true`, now the default — no need for the flag in latest Angular) declare their own dependencies directly:
```typescript
@Component({
  selector: 'app-user-card',
  imports: [CommonModule, RouterLink, UserAvatarComponent],
  template: `...`
})
export class UserCardComponent {}
```
This removes the need for `NgModule` entirely in a fully-standalone app, simplifies lazy loading (`loadComponent` instead of `loadChildren` + a whole feature module), and makes dependencies explicit and local to each component.

**Q2: Can standalone components and NgModules coexist during migration?**

A: Yes — standalone components can be declared/imported inside an existing `NgModule`-based app, and vice versa (an `NgModule` can still export things for standalone components to import), so teams can migrate incrementally rather than needing a big-bang rewrite. The Angular CLI provides an `ng generate @angular/core:standalone` schematic to help automate this migration.

**Q3: With standalone components, how do root-level providers get configured (replacing `AppModule`'s `providers` array)?**

A: Via `bootstrapApplication`'s config object, typically composed from feature-specific `provide*` functions:
```typescript
bootstrapApplication(AppComponent, {
  providers: [
    provideRouter(routes),
    provideHttpClient(withInterceptors([authInterceptor])),
    provideAnimations(),
    provideStore(reducers), // if using NgRx
  ]
});
```
This "provider function" pattern (`provideRouter`, `provideHttpClient`, etc.) replaces the old pattern of importing a whole `NgModule` (like `RouterModule.forRoot()`) just to get its providers.

---

# 17. Performance Optimization

**Q1: What are the main levers for improving Angular app performance?**

A:
- **`OnPush` change detection** (or fully signals-based/zoneless) to avoid unnecessary checks.
- **Lazy loading routes** (`loadComponent`/`loadChildren`) to shrink the initial bundle.
- **`trackBy` / mandatory `track` in `@for`** to avoid unnecessary DOM recreation in lists.
- **`async` pipe** over manual subscriptions, which also implicitly triggers narrower change detection with `OnPush`.
- **Deferrable views (`@defer`, Angular 17+)** to lazily load and render non-critical template sections (e.g., below-the-fold widgets, heavy charts) only when needed (on viewport visibility, interaction, idle, or a timer).
- **Pure pipes** over impure ones, and avoiding expensive function calls directly in templates (a template expression like `{{ getFilteredList() }}` re-runs on every change detection cycle).
- **Virtual scrolling** (`cdk-virtual-scroll-viewport` from Angular CDK) for very long lists, rendering only visible items.
- **Image optimization** — `NgOptimizedImage` directive for automatic lazy loading, responsive sizing, and preventing layout shift.
- **Bundle analysis** (`source-map-explorer`, `webpack-bundle-analyzer`) to catch unexpectedly large dependencies.

**Q2: What is `@defer`, and what triggers are available?**

A: `@defer` (Angular 17+) declaratively splits a template section into a separate lazy-loaded chunk, rendered only when a trigger condition is met:
```html
@defer (on viewport) {
  <heavy-chart [data]="data()" />
} @placeholder {
  <div class="skeleton"></div>
} @loading (minimum 200ms) {
  <spinner />
} @error {
  <p>Failed to load chart.</p>
}
```
Triggers: `on idle` (default, browser idle time), `on viewport` (element scrolls into view), `on interaction` (click/keydown on the element or a specified trigger element), `on hover`, `on timer(Xs)`, or `on immediate`; can also be combined with `when someCondition()` for a custom boolean trigger.

**Q3: Why does calling a method directly in a template binding (`{{ calculateTotal() }}`) hurt performance, and what's the fix?**

A: Under default (or even `OnPush`, once triggered) change detection, every bound expression in the template — including method calls — is **re-evaluated every change detection cycle**. If `calculateTotal()` does real work (looping, filtering), that work re-runs far more often than the underlying data actually changes, especially with Zone.js's broad triggers. Fix: replace it with a memoized `computed()` signal, or a pre-computed pure pipe, so the expensive computation only re-runs when its actual inputs change.

**Q4: How does zoneless change detection improve performance, and what's the migration path?**

A: Removing Zone.js eliminates the overhead of monkey-patching every async browser API and the "recheck everything on any async event" trigger model; combined with signals (which know exactly which bindings depend on which state), Angular can update only what actually changed, and drop Zone.js from the bundle (saving real KB and parse/execution time). Migration involves adopting `provideExperimentalZonelessChangeDetection()` (moving toward stable APIs as the feature matures), and ensuring state changes flow through signals (or explicit `markForCheck()` calls) rather than relying on Zone.js to "just notice" a mutation happened somewhere.

---

# 18. Testing

**Q1: What's the role of `TestBed`, and how does it differ from testing a plain TypeScript class?**

A: `TestBed` configures and creates an **Angular testing module** — a lightweight, test-scoped injector/component-compilation environment — so you can instantiate components with their full DI graph, templates, and change detection behavior, rather than just `new`-ing up a class directly. For a service with no Angular-specific dependencies, you often can just `new` it directly in a test; `TestBed` becomes necessary once you need DI resolution, template rendering, or Angular lifecycle behavior.

```typescript
TestBed.configureTestingModule({
  imports: [UserCardComponent], // standalone component
  providers: [{ provide: UserService, useValue: mockUserService }]
});
const fixture = TestBed.createComponent(UserCardComponent);
fixture.detectChanges(); // triggers ngOnInit + initial render
```

**Q2: How do you test a component that depends on an async Observable-returning service?**

A: Mock the service to return a controlled Observable (often `of(mockData)` for synchronous-style tests, or a `Subject` you manually emit on to test loading/intermediate states), then use `fixture.detectChanges()` (and `fakeAsync`/`tick()` or `waitForAsync`/`fixture.whenStable()` for genuinely async timing) to drive change detection and assert on the rendered DOM or component state afterward.

```typescript
it('should display users after load', fakeAsync(() => {
  mockUserService.getUsers.and.returnValue(of([{ id: 1, name: 'Alice' }]));
  fixture.detectChanges();
  tick();
  fixture.detectChanges();
  const names = fixture.debugElement.queryAll(By.css('.user-name'));
  expect(names[0].nativeElement.textContent).toContain('Alice');
}));
```

**Q3: What's the difference between `fakeAsync`/`tick()` and `waitForAsync`?**

A: `fakeAsync` runs the test in a special zone that lets you **synchronously control simulated time** — `tick(ms)` manually advances virtual time, flushing pending `setTimeout`/Promise microtasks deterministically, without actually waiting in real time. `waitForAsync` (older API, largely superseded by `fakeAsync` for most cases) wraps the test in a zone that waits for real async operations to complete before finishing, useful mainly when `fakeAsync`'s simulated timing can't be used (e.g., genuine `XMLHttpRequest` calls not going through the Angular testing HTTP mocking).

**Q4: How do you test HTTP calls without hitting a real backend?**

A: Use `HttpClientTestingModule` (or `provideHttpClientTesting()` for standalone setups) with `HttpTestingController`:
```typescript
const httpMock = TestBed.inject(HttpTestingController);
service.getUsers().subscribe(users => expect(users.length).toBe(2));

const req = httpMock.expectOne('/api/users');
expect(req.request.method).toBe('GET');
req.flush([{ id: 1 }, { id: 2 }]); // simulate the server response

httpMock.verify(); // ensures no unexpected outstanding requests
```

---

# 19. Security

**Q1: How does Angular protect against XSS by default?**

A: Angular treats all values bound via interpolation (`{{ }}`) and most property bindings as **untrusted data** and automatically **HTML-escapes** them before rendering — so a value like `<script>alert('x')</script>` renders as literal text, not executable markup. For contexts that inherently need to render trusted HTML/URLs/scripts (e.g., `[innerHTML]`), Angular runs the value through its built-in **sanitizer**, stripping dangerous content (script tags, inline event handlers, `javascript:` URLs) based on the security context (HTML, URL, resource URL, style).

**Q2: What is `DomSanitizer.bypassSecurityTrust*`, and why is it risky?**

A: When you genuinely need to render content Angular's sanitizer would otherwise strip (e.g., a trusted, server-controlled rich-HTML block, or an iframe `src` from a known-safe source), `DomSanitizer` provides `bypassSecurityTrustHtml`/`Url`/`ResourceUrl`/`Style`/`Script` to explicitly mark a value as safe, skipping sanitization for it. This is risky because it's an explicit **opt-out of Angular's XSS protection** for that value — it should only ever be used on content that is fully trusted/controlled (never on raw, unescaped user input), since misuse reintroduces exactly the XSS vulnerability Angular's default sanitization exists to prevent.

**Q3: What other security practices matter beyond Angular's built-in XSS protection?**

A: **CSRF protection** — Angular's `HttpClient` has built-in support for the common XSRF-token cookie/header pattern (reading a cookie set by the server and automatically attaching it as a header on outgoing requests) when configured with `withXsrfConfiguration`. **Content Security Policy (CSP)** headers set server-side to restrict script sources as defense-in-depth. Avoiding `bypassSecurityTrust*` on anything user-influenced. Keeping dependencies patched (`npm audit`, Dependabot) since third-party library vulnerabilities are a common real-world attack vector independent of framework-level protections. Never trusting client-side authorization checks alone — route guards improve UX but the **server** must independently enforce authorization on every request, since client-side code can always be bypassed.

---

# 20. SSR / Angular Universal & Hydration

**Q1: What is Server-Side Rendering (SSR) in Angular, and why use it?**

A: SSR (via Angular's built-in SSR support, historically branded "Angular Universal") renders the initial HTML on the **server** (Node.js) and sends a fully-populated HTML page to the browser, rather than shipping a near-empty `<app-root></app-root>` shell that only fills in after JS downloads/executes/bootstraps. Benefits: faster **First Contentful Paint** (users see content immediately), better **SEO** (crawlers see real content without executing JS), and improved performance on slow devices/networks for the critical initial render.

**Q2: What is hydration, and what problem does non-hydrated SSR have?**

A: Without hydration, when the client-side Angular app bootstraps on top of server-rendered HTML, it historically **destroyed and re-created** the entire DOM from scratch (since it had no knowledge the DOM was already correctly rendered) — this caused visible flickering and wasted the SSR performance benefit for anything beyond the very first paint. **Hydration** (`provideClientHydration()`, stable since Angular 17) lets Angular **reuse the existing server-rendered DOM** instead of tearing it down — it "claims" existing elements, attaches event listeners, and reconciles component state onto the already-rendered markup, preserving content (avoiding flicker) while still making the page interactive.

**Q3: What's a common pitfall when writing SSR-compatible components?**

A: Directly accessing browser-only globals (`window`, `document`, `localStorage`, `navigator`) in code paths that run during SSR will throw, since those don't exist in the Node.js server environment. Guard such access with `isPlatformBrowser(this.platformId)` checks (injecting `PLATFORM_ID`), or better, defer browser-only logic to lifecycle points/APIs designed for it (e.g., `afterNextRender()`/`afterRender()`, Angular 16+, which are guaranteed to only run in the browser, never during SSR).

---

# 21. Build, CLI & Deployment

**Q1: What does `ng build --configuration production` do differently from a dev build?**

A: Enables **AOT compilation** (templates compiled to JS ahead of time rather than at runtime — smaller runtime footprint, earlier error detection), **minification and tree-shaking** (dead code elimination, especially effective with Ivy's tree-shakeable architecture), **bundling/code-splitting** per lazy-loaded route, disabling source maps (or generating separate ones) and dev-only checks, and applying **budgets** (configurable warnings/errors if bundle size exceeds thresholds, catching bloat regressions in CI).

**Q2: What is differential loading / modern build output, and how does Angular handle browser compatibility today?**

A: Angular CLI builds output modern ES module bundles by default now that broad browser support for ES modules/modern JS is standard, avoiding the complexity of shipping separate legacy (ES5) and modern bundle sets that older Angular versions used ("differential loading"). The `browserslist` config still lets you control the target compatibility range for autoprefixing/polyfills where genuinely needed.

**Q3: How do environment-specific configs typically work in Angular (e.g., different API URLs for dev/staging/prod)?**

A: Traditionally via `environment.ts`/`environment.prod.ts` files swapped at build time via the `fileReplacements` config per build configuration. This pattern still works but has downsides for true runtime flexibility (values are baked into the JS bundle at build time, meaning you need a separate build per environment). A more flexible alternative for values that shouldn't require a rebuild per environment (e.g., in containerized deployments promoting the *same* build artifact through stages) is fetching a runtime config JSON file (e.g., `config.json` served alongside `index.html`) during app initialization via an `APP_INITIALIZER`/`provideAppInitializer`.

**Q4: What's the Angular CLI builder system, and why does it matter (e.g., for using esbuild/Vite)?**

A: The CLI's build/serve/test commands are implemented as pluggable **builders** (defined in `angular.json`), which is how Angular has been able to migrate its default build pipeline over time — most notably to the new **esbuild-based application builder** (`@angular/build:application`, default for new projects since Angular 17), which offers significantly faster builds and dev-server startup than the previous webpack-based pipeline, while `ng serve` under this builder uses **Vite** for its dev server, giving near-instant HMR. Existing projects can opt in via `ng update`.

---

## Quick-Reference Cheat Sheet

| Topic | One-line takeaway |
|---|---|
| Bootstrap | `bootstrapApplication` builds the injector tree, compiles via Ivy, renders root component |
| Components | Directive + template; `ng-content` projects, `ng-template`/`ng-container` control rendering without extra DOM |
| Data binding | `{{ }}` one-way out, `[ ]` property in, `( )` event out, `[( )]` two-way sugar |
| Directives | Structural changes DOM structure; attribute changes appearance/behavior; `@for`/`@if` replace `*ngFor`/`*ngIf` |
| Pipes | Pure (default, reference-checked) vs impure (every cycle); avoid heavy logic in impure pipes |
| DI | Hierarchical injectors; `providedIn: 'root'` = true singleton; `inject()` needs an injection context |
| Services | Shared, injectable state holders — provide at the right scope to control instance sharing |
| Lifecycle | `ngOnInit` for init logic, `ngOnDestroy` for cleanup — always unsubscribe |
| Change detection | Zone.js triggers tree-wide checks by default; `OnPush` + immutability skips unnecessary ones |
| Signals | Fine-grained, sync reactivity; `signal`/`computed`/`effect`; enables zoneless CD |
| RxJS | `switchMap` (cancel-latest), `mergeMap` (parallel), `concatMap` (ordered), `exhaustMap` (ignore-while-busy) |
| Routing | Functional guards + `inject()`; `loadComponent`/`loadChildren` for lazy loading |
| Forms | Reactive forms for anything non-trivial; validators return `null` or an error object |
| HTTP | Observables enable cancellation; interceptors handle cross-cutting concerns (auth, errors, logging) |
| State mgmt | Signal service for small/medium apps; NgRx for large apps needing strict structure/traceability |
| Standalone | Explicit per-component `imports`; `provide*()` functions replace `NgModule.forRoot()` |
| Performance | `OnPush`, `trackBy`/`track`, `@defer`, virtual scroll, avoid method calls in templates |
| Testing | `TestBed` for DI-aware component tests; `HttpTestingController` to mock HTTP calls |
| Security | Auto-escaping by default; `bypassSecurityTrust*` is an explicit, risky opt-out |
| SSR | Server-renders initial HTML for speed/SEO; hydration reuses that DOM instead of re-rendering it |
| Build/CLI | esbuild + Vite dev server is the modern default; AOT + tree-shaking drive prod bundle size down |

---

*Tip: Angular interviews increasingly probe Signals, standalone components, and the new control-flow syntax (`@if`/`@for`/`@defer`) since these represent the current idiomatic way to write Angular — be ready to explain not just the old NgModule/RxJS-heavy patterns but how the framework has shifted, and why.*
