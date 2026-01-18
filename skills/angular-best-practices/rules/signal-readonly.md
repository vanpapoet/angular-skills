---
title: Expose Signals as Readonly
impact: MEDIUM
impactDescription: Encapsulates state, prevents external mutation
tags: signals, readonly, encapsulation, services
---

## Expose Signals as Readonly

**Impact: MEDIUM (encapsulates state, prevents external mutation)**

Use `asReadonly()` to expose signal values without allowing external modification. This maintains single-source-of-truth for state management.

**Incorrect (exposing writable signals):**

```typescript
// user.service.ts
@Injectable({ providedIn: 'root' })
export class UserService {
  currentUser = signal<User | null>(null);

  login(credentials: Credentials) {
    return this.http.post<User>('/api/login', credentials).pipe(
      tap(user => this.currentUser.set(user))
    );
  }
}

// some.component.ts
@Component({ ... })
export class SomeComponent {
  private userService = inject(UserService);

  ngOnInit() {
    // Dangerous: external code can modify service state
    this.userService.currentUser.set(null);
  }
}
```

**Correct (readonly exposure):**

```typescript
// user.service.ts
@Injectable({ providedIn: 'root' })
export class UserService {
  private _currentUser = signal<User | null>(null);

  // Public readonly view
  readonly currentUser = this._currentUser.asReadonly();

  // Derived state
  readonly isLoggedIn = computed(() => this._currentUser() !== null);
  readonly userName = computed(() => this._currentUser()?.name ?? 'Guest');

  login(credentials: Credentials) {
    return this.http.post<User>('/api/login', credentials).pipe(
      tap(user => this._currentUser.set(user))
    );
  }

  logout() {
    this._currentUser.set(null);
  }
}

// some.component.ts
@Component({
  template: `
    @if (userService.isLoggedIn()) {
      <span>Welcome, {{ userService.userName() }}</span>
    }
  `
})
export class SomeComponent {
  userService = inject(UserService);

  // This would cause TypeScript error:
  // this.userService.currentUser.set(null); // Error!
}
```

**Pattern for state stores:**

```typescript
@Injectable({ providedIn: 'root' })
export class CartStore {
  private _items = signal<CartItem[]>([]);
  private _loading = signal(false);

  // Public readonly views
  readonly items = this._items.asReadonly();
  readonly loading = this._loading.asReadonly();
  readonly itemCount = computed(() => this._items().length);
  readonly total = computed(() =>
    this._items().reduce((sum, i) => sum + i.price, 0)
  );

  // Mutations only through methods
  addItem(item: CartItem) {
    this._items.update(items => [...items, item]);
  }
}
```

Reference: [Angular Signals Documentation](https://angular.dev/guide/signals)
