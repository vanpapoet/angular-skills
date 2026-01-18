---
title: Implement Custom Preloading Strategies
impact: HIGH
impactDescription: Balances initial load time with navigation speed
tags: routing, lazy-loading, preloading, performance
---

## Implement Custom Preloading Strategies

**Impact: HIGH (balances initial load time with navigation speed)**

Preloading strategies download lazy modules in the background after the app loads, improving subsequent navigation speed without increasing initial load time.

**Incorrect (no preloading - slow navigation):**

```typescript
// Lazy routes only load on navigation - user waits
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes) // No preloading
  ]
};
```

**Correct (strategic preloading):**

```typescript
// Option 1: Preload all modules (simple apps)
import { PreloadAllModules } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withPreloading(PreloadAllModules))
  ]
};

// Option 2: Custom selective preloading
@Injectable({ providedIn: 'root' })
export class SelectivePreloadStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    // Check route data for preload flag
    if (route.data?.['preload']) {
      return load();
    }
    return of(null);
  }
}

// Routes with preload data
export const routes: Routes = [
  {
    path: 'dashboard',
    loadComponent: () => import('./dashboard/dashboard.component'),
    data: { preload: true } // Will preload
  },
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes'),
    data: { preload: false } // Won't preload
  },
  {
    path: 'settings',
    loadComponent: () => import('./settings/settings.component')
    // No data - won't preload
  }
];

// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withPreloading(SelectivePreloadStrategy))
  ]
};
```

**Network-aware preloading:**

```typescript
@Injectable({ providedIn: 'root' })
export class NetworkAwarePreloadStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    // Check network conditions
    const connection = (navigator as any).connection;

    if (connection) {
      // Don't preload on slow connections
      if (connection.saveData || connection.effectiveType === '2g') {
        return of(null);
      }
    }

    // Preload if route is marked for preloading
    if (route.data?.['preload']) {
      // Optional: delay preloading to not compete with critical resources
      return timer(2000).pipe(switchMap(() => load()));
    }

    return of(null);
  }
}
```

**Role-based preloading:**

```typescript
@Injectable({ providedIn: 'root' })
export class RoleBasedPreloadStrategy implements PreloadingStrategy {
  private authService = inject(AuthService);

  preload(route: Route, load: () => Observable<any>): Observable<any> {
    const requiredRole = route.data?.['role'];

    // Only preload routes user has access to
    if (!requiredRole || this.authService.hasRole(requiredRole)) {
      return route.data?.['preload'] ? load() : of(null);
    }

    return of(null);
  }
}
```

**Preloading strategies comparison:**

| Strategy | Use Case |
|----------|----------|
| `PreloadAllModules` | Small apps, fast networks |
| `NoPreloading` | Mobile-first, data-saving |
| Custom selective | Large apps, prioritized features |
| Network-aware | Adaptive to user conditions |

Reference: [Angular Lazy Loading](https://github.com/angular-vietnam/100-days-of-angular/blob/master/Day029-router-lazy-load.md)
