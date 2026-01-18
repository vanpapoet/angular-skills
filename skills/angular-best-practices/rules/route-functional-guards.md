---
title: Use Functional Guards for Route Protection
impact: HIGH
impactDescription: Simpler syntax, tree-shakable, testable
tags: routing, guards, canActivate, functional
---

## Use Functional Guards for Route Protection

**Impact: HIGH (simpler syntax, tree-shakable, testable)**

Functional guards are the modern approach in Angular. They're simpler than class-based guards and can use `inject()` for dependencies.

**Incorrect (class-based guard - verbose):**

```typescript
@Injectable({ providedIn: 'root' })
export class AuthGuard implements CanActivate {
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): Observable<boolean | UrlTree> {
    return this.authService.isLoggedIn$.pipe(
      map(isLoggedIn => isLoggedIn || this.router.createUrlTree(['/login']))
    );
  }
}
```

**Correct (functional guard):**

```typescript
// guards/auth.guard.ts
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  return authService.isLoggedIn$.pipe(
    map(isLoggedIn => isLoggedIn || router.createUrlTree(['/login']))
  );
};

// guards/role.guard.ts
export const roleGuard = (allowedRoles: string[]): CanActivateFn => {
  return (route, state) => {
    const authService = inject(AuthService);
    const router = inject(Router);

    return authService.currentUser$.pipe(
      map(user => {
        if (allowedRoles.includes(user?.role)) {
          return true;
        }
        return router.createUrlTree(['/forbidden']);
      })
    );
  };
};

// guards/permission.guard.ts
export const canEditGuard: CanActivateFn = (route, state) => {
  const userService = inject(UserService);
  const articleService = inject(ArticleService);

  const slug = route.paramMap.get('slug');
  return articleService.getBySlug(slug).pipe(
    map(article => article.author === userService.currentUser?.username)
  );
};

// app.routes.ts
export const routes: Routes = [
  {
    path: 'dashboard',
    component: DashboardComponent,
    canActivate: [authGuard]
  },
  {
    path: 'admin',
    loadComponent: () => import('./admin/admin.component'),
    canActivate: [authGuard, roleGuard(['admin'])]
  },
  {
    path: 'articles/:slug/edit',
    component: ArticleEditComponent,
    canActivate: [authGuard, canEditGuard]
  }
];
```

**CanActivateChild for child routes:**

```typescript
export const adminChildGuard: CanActivateChildFn = (childRoute, state) => {
  const authService = inject(AuthService);
  return authService.currentUser$.pipe(
    map(user => user?.isAdmin ?? false)
  );
};

// routes
{
  path: 'admin',
  canActivateChild: [adminChildGuard],
  children: [
    { path: 'users', component: UsersComponent },
    { path: 'settings', component: SettingsComponent }
  ]
}
```

**Guard return values:**
- `true` - navigation proceeds
- `false` - navigation cancelled
- `UrlTree` - redirect to another route

Reference: [Angular Router Guards](https://angular.dev/guide/routing)
