---
title: Use Selectors to Read ComponentStore State
impact: MEDIUM
impactDescription: Memoized, composable state access
tags: ngrx, component-store, selectors, state-management
---

## Use Selectors to Read ComponentStore State

**Impact: MEDIUM (memoized, composable state access)**

Selectors in ComponentStore extract specific slices of state as observables. Combine selectors for derived state and create view model selectors for templates.

**Incorrect (direct state access):**

```typescript
@Injectable()
export class UserStore extends ComponentStore<UserState> {
  // Exposing raw state - no memoization
  getUsers() {
    return this.state$.pipe(map(s => s.users));
  }

  getActiveUsers() {
    return this.state$.pipe(
      map(s => s.users.filter(u => u.active))
    );
  }
}
```

**Correct (proper selectors):**

```typescript
@Injectable()
export class UserStore extends ComponentStore<UserState> {
  constructor() {
    super({ users: [], filter: '', loading: false });
  }

  // Basic selectors
  readonly users$ = this.select(state => state.users);
  readonly filter$ = this.select(state => state.filter);
  readonly loading$ = this.select(state => state.loading);

  // Derived selectors (combined from other selectors)
  readonly activeUsers$ = this.select(
    this.users$,
    users => users.filter(u => u.active)
  );

  readonly filteredUsers$ = this.select(
    this.users$,
    this.filter$,
    (users, filter) => users.filter(u =>
      u.name.toLowerCase().includes(filter.toLowerCase())
    )
  );

  readonly userCount$ = this.select(
    this.filteredUsers$,
    users => users.length
  );

  // View model selector - all data for template
  readonly vm$ = this.select({
    users: this.filteredUsers$,
    count: this.userCount$,
    loading: this.loading$,
    hasUsers: this.select(this.filteredUsers$, users => users.length > 0)
  });

  // Selector with external observable
  readonly userWithPermissions$ = this.select(
    this.users$,
    this.permissionService.permissions$, // External observable
    (users, permissions) => users.map(u => ({
      ...u,
      canEdit: permissions.includes(`edit:${u.id}`)
    }))
  );

  // Synchronous state access (use sparingly)
  get currentFilter(): string {
    return this.get(state => state.filter);
  }
}

// Component usage
@Component({
  template: `
    @if (store.vm$ | async; as vm) {
      <p>Showing {{ vm.count }} users</p>
      @if (vm.loading) {
        <app-spinner />
      } @else if (vm.hasUsers) {
        @for (user of vm.users; track user.id) {
          <app-user-card [user]="user" />
        }
      } @else {
        <p>No users found</p>
      }
    }
  `
})
export class UserListComponent {
  store = inject(UserStore);
}
```

**Selector best practices:**

1. **Create atomic selectors** for each state property
2. **Combine selectors** for derived state (memoized automatically)
3. **Use vm$ selector** to bundle template data
4. **Avoid `get()`** for synchronous access except when necessary

Reference: [NgRx ComponentStore Selectors](https://ngrx.io/guide/component-store/read)
