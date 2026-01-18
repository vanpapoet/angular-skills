---
title: Use Effects for Async Operations in ComponentStore
impact: HIGH
impactDescription: Handles side effects, prevents observable leaks
tags: ngrx, component-store, effects, async, side-effects
---

## Use Effects for Async Operations in ComponentStore

**Impact: HIGH (handles side effects, prevents observable leaks)**

Effects handle async operations like API calls. Use `tapResponse` to handle success/error without killing the effect stream.

**Incorrect (async in updaters or manual subscriptions):**

```typescript
@Injectable()
export class UserStore extends ComponentStore<UserState> {
  // Bad: async logic in updater
  loadUsers = this.updater((state) => {
    this.http.get<User[]>('/api/users').subscribe(users => {
      this.patchState({ users }); // Race conditions possible
    });
    return { ...state, loading: true };
  });

  // Bad: manual subscription management
  fetchUser(id: string) {
    this.patchState({ loading: true });
    this.http.get<User>(`/api/users/${id}`).subscribe({
      next: user => this.patchState({ currentUser: user, loading: false }),
      error: err => this.patchState({ error: err.message, loading: false })
    });
    // Subscription leak if component destroys
  }
}
```

**Correct (effects with tapResponse):**

```typescript
import { tapResponse } from '@ngrx/operators';

@Injectable()
export class UserStore extends ComponentStore<UserState> {
  private http = inject(HttpClient);

  constructor() {
    super({ users: [], currentUser: null, loading: false, error: null });
  }

  // Effect without parameters
  readonly loadUsers = this.effect<void>(trigger$ =>
    trigger$.pipe(
      tap(() => this.patchState({ loading: true, error: null })),
      switchMap(() => this.http.get<User[]>('/api/users').pipe(
        tapResponse(
          users => this.patchState({ users, loading: false }),
          error => this.patchState({ error: error.message, loading: false })
        )
      ))
    )
  );

  // Effect with parameter
  readonly loadUser = this.effect<string>(userId$ =>
    userId$.pipe(
      tap(() => this.patchState({ loading: true })),
      switchMap(id => this.http.get<User>(`/api/users/${id}`).pipe(
        tapResponse(
          user => this.patchState({ currentUser: user, loading: false }),
          error => this.patchState({ error: error.message, loading: false })
        )
      ))
    )
  );

  // Effect with observable parameter
  readonly loadUserFromRoute = this.effect<Observable<string>>(userId$ =>
    userId$.pipe(
      switchMap(idObs => idObs),
      distinctUntilChanged(),
      tap(id => this.patchState({ loading: true })),
      switchMap(id => this.http.get<User>(`/api/users/${id}`).pipe(
        tapResponse(
          user => this.patchState({ currentUser: user, loading: false }),
          error => this.patchState({ error: error.message, loading: false })
        )
      ))
    )
  );

  // Effect with debounce for search
  readonly searchUsers = this.effect<string>(query$ =>
    query$.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      tap(() => this.patchState({ loading: true })),
      switchMap(query => this.http.get<User[]>(`/api/users?q=${query}`).pipe(
        tapResponse(
          users => this.patchState({ users, loading: false }),
          error => this.patchState({ error: error.message, loading: false })
        )
      ))
    )
  );

  // Effect that dispatches to global store
  readonly saveAndNotify = this.effect<User>(user$ =>
    user$.pipe(
      switchMap(user => this.http.post<User>('/api/users', user).pipe(
        tapResponse(
          saved => {
            this.patchState({ currentUser: saved });
            this.globalStore.dispatch(showNotification({ message: 'Saved!' }));
          },
          error => this.patchState({ error: error.message })
        )
      ))
    )
  );
}

// Component usage
@Component({
  providers: [UserStore],
  template: `...`
})
export class UserComponent {
  store = inject(UserStore);
  route = inject(ActivatedRoute);

  ngOnInit() {
    // Trigger effect without params
    this.store.loadUsers();

    // Trigger effect with value
    this.store.loadUser('user-123');

    // Pass observable to effect
    this.store.loadUserFromRoute(
      this.route.params.pipe(map(p => p['id']))
    );
  }
}
```

**Key points:**
- Use `tapResponse` - handles errors without killing effect
- Use `switchMap` to cancel previous requests
- Effects auto-cleanup when store is destroyed
- Pass observables for reactive parameters

Reference: [NgRx ComponentStore Effects](https://ngrx.io/guide/component-store/effect)
