---
title: Prefer inject() Over Constructor Injection
impact: MEDIUM
impactDescription: Cleaner syntax, better type inference, works in field initializers
tags: dependency-injection, inject, services
---

## Prefer inject() Over Constructor Injection

**Impact: MEDIUM (cleaner syntax, better type inference, works in field initializers)**

The `inject()` function provides a more concise and flexible way to inject dependencies compared to constructor parameters.

**Incorrect (verbose constructor injection):**

```typescript
@Component({ ... })
export class UserComponent {
  private userService: UserService;
  private router: Router;
  private activatedRoute: ActivatedRoute;
  private http: HttpClient;

  constructor(
    userService: UserService,
    router: Router,
    activatedRoute: ActivatedRoute,
    http: HttpClient
  ) {
    this.userService = userService;
    this.router = router;
    this.activatedRoute = activatedRoute;
    this.http = http;
  }

  // Or with parameter properties (still verbose)
  constructor(
    private userService: UserService,
    private router: Router,
    private activatedRoute: ActivatedRoute,
    private http: HttpClient
  ) {}
}
```

**Correct (inject function):**

```typescript
import { inject } from '@angular/core';

@Component({ ... })
export class UserComponent {
  private userService = inject(UserService);
  private router = inject(Router);
  private route = inject(ActivatedRoute);
  private http = inject(HttpClient);

  // Can use in field initializers
  userId = toSignal(
    this.route.params.pipe(map(p => p['id']))
  );

  user = toSignal(
    this.userService.getUser(this.userId()!)
  );
}
```

**inject() in services:**

```typescript
@Injectable({ providedIn: 'root' })
export class AuthService {
  private http = inject(HttpClient);
  private router = inject(Router);
  private storage = inject(STORAGE_TOKEN);

  login(credentials: Credentials) {
    return this.http.post<AuthResponse>('/api/login', credentials).pipe(
      tap(response => {
        this.storage.setItem('token', response.token);
        this.router.navigate(['/dashboard']);
      })
    );
  }
}
```

**inject() with options:**

```typescript
@Component({ ... })
export class OptionalDepComponent {
  // Optional injection
  private analytics = inject(AnalyticsService, { optional: true });

  // Skip self (look in parent injectors only)
  private parentConfig = inject(CONFIG_TOKEN, { skipSelf: true });

  // Self only (don't look in parent injectors)
  private localService = inject(LocalService, { self: true });
}
```

**Benefits of inject():**
- Works in field initializers (before constructor runs)
- Cleaner, more readable code
- Better TypeScript type inference
- Easier to refactor (no constructor parameter reordering)

Reference: [Angular Dependency Injection](https://angular.dev/guide/di)
