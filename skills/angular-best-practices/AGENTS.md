# Angular Best Practices - Complete Guide

Comprehensive performance optimization and best practices guide for Angular v21+ applications. This document contains all rules expanded for AI agents and LLMs.

**Version:** 1.0.0
**Angular Version:** 21+
**Last Updated:** January 2026

---

## Table of Contents

1. [Change Detection (CRITICAL)](#1-change-detection-critical)
2. [Bundle Optimization (CRITICAL)](#2-bundle-optimization-critical)
3. [Template Performance (HIGH)](#3-template-performance-high)
4. [Component Architecture (HIGH)](#4-component-architecture-high)
5. [Routing & Navigation (HIGH)](#5-routing--navigation-high)
6. [State Management - Signals (MEDIUM-HIGH)](#6-state-management---signals-medium-high)
7. [State Management - ComponentStore (MEDIUM-HIGH)](#7-state-management---componentstore-medium-high)
8. [Forms (MEDIUM)](#8-forms-medium)
9. [Security (MEDIUM)](#9-security-medium)
10. [Dependency Injection (MEDIUM)](#10-dependency-injection-medium)
11. [HTTP & Data Fetching (MEDIUM)](#11-http--data-fetching-medium)
12. [Style & Conventions (LOW)](#12-style--conventions-low)

---

## 1. Change Detection (CRITICAL)

### 1.1 Use OnPush Change Detection Strategy

**Impact: CRITICAL (2-10x fewer change detection cycles)**

OnPush instructs Angular to run change detection for a component subtree only when it receives new inputs or handles events.

```typescript
// Correct
@Component({
  selector: 'app-user-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `@for (user of users; track user.id) {
    <div>{{ user.name }}</div>
  }`
})
export class UserListComponent {
  @Input() users: User[];
}
```

### 1.2 Adopt Zoneless Change Detection

**Impact: CRITICAL (eliminates Zone.js overhead, smaller bundles)**

Angular v21 introduces zoneless as the default for new applications.

```typescript
// main.ts
bootstrapApplication(AppComponent, {
  providers: [
    provideExperimentalZonelessChangeDetection()
  ]
});
```

### 1.3 Use Signal-Based Inputs

**Impact: CRITICAL (fine-grained reactivity, automatic change detection)**

```typescript
@Component({
  selector: 'app-user-card',
  template: `<h2>{{ user().name }}</h2>`
})
export class UserCardComponent {
  user = input.required<User>();
  displayName = computed(() => `${this.user().firstName} ${this.user().lastName}`);
}
```

### 1.4 Use markForCheck() for Manual Updates

**Impact: HIGH**

When using OnPush, use `markForCheck()` for async updates outside Angular's zone, or prefer signals for automatic tracking.

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<div>{{ data() }}</div>`
})
export class DataComponent {
  data = signal('');

  constructor(private socket: WebSocketService) {
    this.socket.messages$.subscribe(msg => {
      this.data.set(msg); // Automatic with signals
    });
  }
}
```

---

## 2. Bundle Optimization (CRITICAL)

### 2.1 Use @defer for Lazy Loading Components

**Impact: CRITICAL (50-90% reduction in initial bundle size)**

```typescript
@Component({
  template: `
    @defer (on viewport) {
      <app-heavy-chart [data]="chartData" />
    } @placeholder {
      <div class="chart-skeleton">Loading chart...</div>
    } @loading (minimum 200ms) {
      <app-spinner />
    } @error {
      <p>Failed to load. <button (click)="reload()">Retry</button></p>
    }
  `
})
export class DashboardComponent {}
```

**Triggers:** `on idle`, `on viewport`, `on interaction`, `on hover`, `on immediate`, `on timer(Xms)`, `when condition`

### 2.2 Lazy Load Route Modules

**Impact: CRITICAL**

```typescript
export const routes: Routes = [
  {
    path: 'admin',
    loadComponent: () => import('./admin/admin.component')
      .then(m => m.AdminComponent)
  },
  {
    path: 'reports',
    loadChildren: () => import('./reports/reports.routes')
      .then(m => m.REPORTS_ROUTES)
  }
];
```

### 2.3 Prefetch Deferred Content Strategically

**Impact: HIGH**

```typescript
@defer (on interaction; prefetch on idle) {
  <app-modal />
} @placeholder {
  <button>Open Modal</button>
}
```

### 2.4 Use Standalone Components

**Impact: HIGH (better tree-shaking, reduced boilerplate)**

```typescript
@Component({
  selector: 'app-user',
  standalone: true,
  imports: [DatePipe, RouterLink],
  template: `<h2>{{ user().name }}</h2>`
})
export class UserComponent {
  user = input.required<User>();
}
```

---

## 3. Template Performance (HIGH)

### 3.1 Use Native Control Flow Syntax

**Impact: HIGH (better performance, smaller bundles, improved type inference)**

```typescript
// Correct: Native control flow
@if (user; as u) {
  <h2>{{ u.name }}</h2>
} @else {
  <p>No user found</p>
}

@for (item of items; track item.id) {
  <li>{{ item.name }}</li>
} @empty {
  <li>No items available</li>
}

@switch (status) {
  @case ('active') { <p>Active</p> }
  @case ('inactive') { <p>Inactive</p> }
  @default { <p>Unknown</p> }
}
```

### 3.2 Always Use track in @for Loops

**Impact: HIGH (prevents unnecessary DOM recreation, 2-10x faster list updates)**

```typescript
// Correct: track by unique identifier
@for (user of users; track user.id) {
  <app-user-card [user]="user" />
}

// Available variables: $index, $first, $last, $even, $odd, $count
```

### 3.3 Avoid Function Calls in Templates

**Impact: HIGH**

```typescript
// Incorrect: function called every CD cycle
<div>{{ getFullName() }}</div>

// Correct: computed signal
fullName = computed(() => `${this.user().firstName} ${this.user().lastName}`);
<div>{{ fullName() }}</div>
```

### 3.4 Use Pure Pipes for Transformations

**Impact: MEDIUM**

```typescript
@Pipe({ name: 'filter', pure: true })
export class FilterPipe implements PipeTransform {
  transform(items: Item[], searchTerm: string): Item[] {
    return items.filter(i => i.name.includes(searchTerm));
  }
}
```

---

## 4. Component Architecture (HIGH)

### 4.1 Prefer Standalone Components

**Impact: HIGH**

Standalone components are the default in Angular v19+. Use direct imports instead of NgModules.

### 4.2 One Component Per File

**Impact: MEDIUM**

Each file should contain one component, directive, pipe, or service.

### 4.3 Separate Smart and Presentational Components

**Impact: MEDIUM**

```typescript
// Presentational: handles UI only
@Component({
  selector: 'app-user-card',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div (click)="selected.emit()">
      <h3>{{ user().name }}</h3>
    </div>
  `
})
export class UserCardComponent {
  user = input.required<User>();
  selected = output<void>();
}

// Smart: handles data and logic
@Component({
  selector: 'app-user-list',
  imports: [UserCardComponent],
  template: `
    @for (user of users(); track user.id) {
      <app-user-card [user]="user" (selected)="onSelect(user)" />
    }
  `
})
export class UserListComponent {
  private userService = inject(UserService);
  users = toSignal(this.userService.getUsers(), { initialValue: [] });
}
```

### 4.4 Use host Property Over Decorators

**Impact: LOW**

```typescript
@Component({
  selector: 'app-button',
  template: `<ng-content />`,
  host: {
    'role': 'button',
    '[class.active]': 'isActive',
    '[class.disabled]': 'disabled()',
    '(click)': 'onClick($event)'
  }
})
export class ButtonComponent {}
```

---

## 5. Routing & Navigation (HIGH)

### 5.1 Use Functional Guards for Route Protection

**Impact: HIGH (simpler syntax, tree-shakable, testable)**

```typescript
// Functional guard with inject()
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  return authService.isLoggedIn$.pipe(
    map(isLoggedIn => isLoggedIn || router.createUrlTree(['/login']))
  );
};

// Parameterized guard
export const roleGuard = (allowedRoles: string[]): CanActivateFn => {
  return (route, state) => {
    const authService = inject(AuthService);
    return authService.currentUser$.pipe(
      map(user => allowedRoles.includes(user?.role))
    );
  };
};

// Route configuration
{ path: 'admin', loadComponent: ..., canActivate: [authGuard, roleGuard(['admin'])] }
```

### 5.2 Use CanDeactivate to Prevent Unsaved Changes Loss

**Impact: MEDIUM**

```typescript
export interface CanDeactivateComponent {
  canDeactivate(): boolean | Observable<boolean>;
}

export const unsavedChangesGuard: CanDeactivateFn<CanDeactivateComponent> = (component) => {
  return component.canDeactivate ? component.canDeactivate() : true;
};

// Component implementation
@Component({ ... })
export class FormComponent implements CanDeactivateComponent {
  canDeactivate(): boolean {
    return !this.form.dirty || confirm('Discard unsaved changes?');
  }
}
```

### 5.3 Use CanMatch to Control Lazy Loading

**Impact: MEDIUM (prevents unauthorized module downloads)**

```typescript
// Prevents module download for unauthorized users
export const canMatchAdmin: CanMatchFn = (route, segments) => {
  const authService = inject(AuthService);
  return authService.currentUserSnapshot?.isAdmin ?? false;
};

// Route with CanMatch - checked BEFORE download
{ path: 'admin', loadChildren: () => import('./admin/admin.routes'), canMatch: [canMatchAdmin] }
```

### 5.4 Use DI Factories for Route Parameters

**Impact: MEDIUM (testable, reusable)**

```typescript
// With input binding (Angular 16+)
provideRouter(routes, withComponentInputBinding())

@Component({ ... })
export class ProductComponent {
  id = input.required<string>(); // Route param ':id' auto-bound
  filter = input<string>();       // Query param 'filter' auto-bound
}
```

### 5.5 Implement Custom Preloading Strategies

**Impact: HIGH**

```typescript
@Injectable({ providedIn: 'root' })
export class SelectivePreloadStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    return route.data?.['preload'] ? load() : of(null);
  }
}

// Routes with preload data
{ path: 'dashboard', loadComponent: ..., data: { preload: true } }

// Configuration
provideRouter(routes, withPreloading(SelectivePreloadStrategy))
```

### 5.6 Use Resolvers for Pre-fetching Route Data

**Impact: MEDIUM**

```typescript
export const productResolver: ResolveFn<Product> = (route, state) => {
  const productService = inject(ProductService);
  const id = route.paramMap.get('id')!;
  return productService.getById(id);
};

// Route configuration
{ path: 'products/:id', component: ProductComponent, resolve: { product: productResolver } }

// Component - data already available
product = toSignal(this.route.data.pipe(map(d => d['product'])), { requireSync: true });
```

---

## 6. State Management - Signals (MEDIUM-HIGH)

### 6.1 Use computed() for Derived State

**Impact: HIGH**

```typescript
firstName = signal('John');
lastName = signal('Doe');

fullName = computed(() => `${this.firstName()} ${this.lastName()}`);
```

### 6.2 Expose Signals as Readonly

**Impact: MEDIUM**

```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  private _currentUser = signal<User | null>(null);
  readonly currentUser = this._currentUser.asReadonly();
  readonly isLoggedIn = computed(() => this._currentUser() !== null);
}
```

### 6.3 Use effect() Sparingly for Side Effects

**Impact: MEDIUM**

```typescript
// Correct: effect for side effects only
effect(() => {
  localStorage.setItem('cart', JSON.stringify(this.items()));
});

// Incorrect: don't use effect to update state
effect(() => {
  this.total.set(this.items().reduce((s, i) => s + i.price, 0)); // Bad!
});
// Use computed() instead for derived state
```

### 6.4 Use untracked() to Prevent Dependency Tracking

**Impact: MEDIUM**

```typescript
effect(() => {
  const term = this.searchTerm(); // Tracked
  const cfg = untracked(() => this.config()); // Not tracked
  this.performSearch(term, cfg);
});
```

---

## 7. State Management - ComponentStore (MEDIUM-HIGH)

### 7.1 Use ComponentStore for Local Component State

**Impact: HIGH (encapsulated state management)**

```typescript
interface ProductListState {
  products: Product[];
  loading: boolean;
  error: string | null;
}

@Injectable()
export class ProductListStore extends ComponentStore<ProductListState> {
  constructor() {
    super({ products: [], loading: false, error: null });
  }

  // Selectors
  readonly products$ = this.select(state => state.products);
  readonly loading$ = this.select(state => state.loading);

  // View model selector
  readonly vm$ = this.select({
    products: this.products$,
    loading: this.loading$
  });

  // Updaters
  readonly setLoading = this.updater((state, loading: boolean) => ({
    ...state, loading
  }));

  // Effects
  readonly loadProducts = this.effect<void>(trigger$ =>
    trigger$.pipe(
      tap(() => this.patchState({ loading: true })),
      switchMap(() => this.productService.getAll().pipe(
        tapResponse(
          products => this.patchState({ products, loading: false }),
          error => this.patchState({ error: error.message, loading: false })
        )
      ))
    )
  );
}

// Component with scoped store
@Component({
  providers: [ProductListStore],
  template: `@if (store.vm$ | async; as vm) { ... }`
})
export class ProductListComponent {
  store = inject(ProductListStore);
}
```

### 7.2 Use Selectors for State Access

**Impact: MEDIUM**

```typescript
// Combined selectors for derived state
readonly filteredProducts$ = this.select(
  this.products$,
  this.select(state => state.filter),
  (products, filter) => products.filter(p => p.name.includes(filter))
);
```

### 7.3 Use Updaters for Synchronous Changes

**Impact: MEDIUM**

```typescript
// Updater with payload
readonly addItem = this.updater((state, item: CartItem) => ({
  ...state,
  items: [...state.items, item]
}));

// patchState for simple updates
this.patchState({ loading: true });
```

### 7.4 Use Effects with tapResponse for Async Operations

**Impact: HIGH**

```typescript
readonly loadUser = this.effect<string>(userId$ =>
  userId$.pipe(
    switchMap(id => this.http.get<User>(`/api/users/${id}`).pipe(
      tapResponse(
        user => this.patchState({ user, loading: false }),
        error => this.patchState({ error: error.message, loading: false })
      )
    ))
  )
);
```

---

## 8. Forms (MEDIUM)

### 8.1 Prefer Reactive Forms for Complex Scenarios

**Impact: MEDIUM**

```typescript
interface UserForm {
  name: FormControl<string>;
  email: FormControl<string>;
  addresses: FormArray<FormGroup<AddressForm>>;
}

@Component({ ... })
export class UserFormComponent {
  private fb = inject(FormBuilder);

  form = this.fb.group<UserForm>({
    name: this.fb.control('', { nonNullable: true, validators: [Validators.required] }),
    email: this.fb.control('', { nonNullable: true, validators: [Validators.email] }),
    addresses: this.fb.array<FormGroup<AddressForm>>([])
  });
}
```

### 8.2 Use Strongly Typed Forms

**Impact: MEDIUM**

Use `NonNullableFormBuilder` and interface definitions for full type safety.

### 8.3 Centralize Validation Logic

**Impact: MEDIUM**

Create reusable validator functions and centralize error messages.

### 8.4 Clean Up Form Subscriptions

**Impact: MEDIUM**

```typescript
// Use takeUntilDestroyed
this.form.valueChanges.pipe(
  takeUntilDestroyed(this.destroyRef)
).subscribe();

// Or use toSignal
searchValue = toSignal(this.searchControl.valueChanges, { initialValue: '' });
```

---

## 9. Security (MEDIUM)

### 9.1 Trust Angular's Automatic Sanitization

Angular automatically sanitizes untrusted values for HTML, Style, URL, and Resource URL contexts.

### 9.2 Use bypassSecurityTrust* with Extreme Caution

Only use for content YOU control, never for user input. Document why it's safe.

### 9.3 Configure Content Security Policy

```typescript
// Express.js example
res.setHeader('Content-Security-Policy', [
  "default-src 'self'",
  `script-src 'self' 'nonce-${nonce}'`,
  `style-src 'self' 'nonce-${nonce}'`
].join('; '));
```

### 9.4 Never Generate Templates Dynamically

Use static templates with data binding. Never construct templates from user input.

---

## 10. Dependency Injection (MEDIUM)

### 10.1 Prefer inject() Over Constructor Injection

**Impact: MEDIUM**

```typescript
@Component({ ... })
export class UserComponent {
  private userService = inject(UserService);
  private router = inject(Router);
}
```

### 10.2 Use providedIn: 'root' for Singleton Services

**Impact: MEDIUM**

```typescript
@Injectable({ providedIn: 'root' })
export class AuthService {}
```

### 10.3 Use InjectionToken for Non-Class Dependencies

**Impact: LOW**

```typescript
export const API_URL = new InjectionToken<string>('API URL', {
  providedIn: 'root',
  factory: () => environment.apiUrl
});
```

---

## 11. HTTP & Data Fetching (MEDIUM)

### 11.1 Use Interceptors for Cross-Cutting Concerns

```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(AuthService).getToken();
  if (token) {
    req = req.clone({ setHeaders: { Authorization: `Bearer ${token}` } });
  }
  return next(req);
};
```

### 11.2 Type HTTP Responses

```typescript
getUsers(): Observable<UserListResponse> {
  return this.http.get<UserListResponse>('/api/users');
}
```

### 11.3 Centralize Error Handling

Use interceptors and a centralized error service for consistent handling.

### 11.4 Implement HTTP Caching Strategies

Use `resource()` API, `shareReplay()`, or custom caching for repeated requests.

---

## 12. Style & Conventions (LOW)

### 12.1 Use Kebab-Case for File Names

```
user-profile.component.ts
auth.service.ts
date-format.pipe.ts
```

### 12.2 Apply Consistent Selector Prefixes

```typescript
@Component({ selector: 'app-user-card' })
@Directive({ selector: '[appTooltip]' })
```

### 12.3 Group and Organize Imports

1. Angular core imports
2. Third-party library imports
3. Application imports (core/shared)
4. Feature-specific imports

### 12.4 Implement Lifecycle Interfaces

```typescript
export class UserComponent implements OnInit, OnDestroy {
  ngOnInit(): void { }
  ngOnDestroy(): void { }
}
```

---

## References

- [Angular Documentation](https://angular.dev)
- [Angular Style Guide](https://angular.dev/style-guide)
- [Angular Security](https://angular.dev/best-practices/security)
- [Angular Signals](https://angular.dev/guide/signals)
- [Angular v21 Release](https://angular.love/angular-21-whats-new)
- [NgRx ComponentStore](https://ngrx.io/guide/component-store)
- [Angular Vietnam - Router Guards](https://github.com/angular-vietnam/100-days-of-angular)
- [NgRx ComponentStore Best Practices](https://sagarsnath.medium.com/ngrx-component-store-from-basics-to-best-practices-1234bc8d70a1)
