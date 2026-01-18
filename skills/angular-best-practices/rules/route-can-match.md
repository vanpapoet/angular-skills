---
title: Use CanMatch to Control Lazy Loading
impact: MEDIUM
impactDescription: Prevents unauthorized module downloads
tags: routing, guards, canMatch, lazy-loading, security
---

## Use CanMatch to Control Lazy Loading

**Impact: MEDIUM (prevents unauthorized module downloads)**

CanMatch (replacement for deprecated CanLoad) prevents lazy-loaded modules from downloading for unauthorized users. Unlike CanActivate, it runs before the module is fetched.

**Incorrect (CanActivate for lazy routes - downloads first):**

```typescript
// Module downloads before guard runs - wastes bandwidth
{
  path: 'admin',
  loadChildren: () => import('./admin/admin.routes'),
  canActivate: [adminGuard] // Too late - module already downloaded
}
```

**Correct (CanMatch - prevents download):**

```typescript
// guards/can-match-admin.guard.ts
export const canMatchAdmin: CanMatchFn = (route, segments) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  // Synchronous check preferred for CanMatch
  const user = authService.currentUserSnapshot;

  if (user?.isAdmin) {
    return true;
  }

  // Redirect or return false
  return router.createUrlTree(['/login']);
};

// guards/can-match-feature.guard.ts
export const canMatchFeature = (featureFlag: string): CanMatchFn => {
  return (route, segments) => {
    const featureService = inject(FeatureFlagService);
    return featureService.isEnabled(featureFlag);
  };
};

// app.routes.ts
export const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes'),
    canMatch: [canMatchAdmin] // Checked BEFORE download
  },
  {
    path: 'beta-feature',
    loadComponent: () => import('./beta/beta.component'),
    canMatch: [canMatchFeature('beta_v2')]
  },
  // Fallback for non-admin users
  {
    path: 'admin',
    redirectTo: '/forbidden'
  }
];
```

**Role-based route matching:**

```typescript
// Multiple routes with same path, different guards
export const routes: Routes = [
  {
    path: 'dashboard',
    loadComponent: () => import('./admin-dashboard/admin-dashboard.component'),
    canMatch: [canMatchAdmin]
  },
  {
    path: 'dashboard',
    loadComponent: () => import('./user-dashboard/user-dashboard.component'),
    canMatch: [canMatchUser]
  },
  {
    path: 'dashboard',
    redirectTo: '/login' // Fallback if no match
  }
];
```

**CanMatch vs CanActivate:**

| Guard | When Runs | Use Case |
|-------|-----------|----------|
| `canMatch` | Before route matching | Prevent lazy module download |
| `canActivate` | After route matched | Protect already-loaded routes |

**Note:** `CanLoad` is deprecated in Angular 15.1+. Use `canMatch` instead.

Reference: [Angular Router Guards](https://angular.dev/guide/routing)
