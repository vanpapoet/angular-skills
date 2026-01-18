---
title: Lazy Load Route Modules
impact: CRITICAL
impactDescription: Reduces initial bundle by loading routes on demand
tags: routing, lazy-loading, bundle-size, code-splitting
---

## Lazy Load Route Modules

**Impact: CRITICAL (reduces initial bundle by loading routes on demand)**

Route-based code splitting loads feature modules only when users navigate to them, significantly reducing initial load time.

**Incorrect (eagerly imported routes):**

```typescript
// app.routes.ts
import { AdminComponent } from './admin/admin.component';
import { ReportsComponent } from './reports/reports.component';

export const routes: Routes = [
  { path: 'admin', component: AdminComponent },
  { path: 'reports', component: ReportsComponent }
];
```

**Correct (lazy loaded routes):**

```typescript
// app.routes.ts
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

**With preloading strategy:**

```typescript
// app.config.ts
import { PreloadAllModules } from '@angular/router';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes, withPreloading(PreloadAllModules))
  ]
};
```

**Custom preloading (selective):**

```typescript
// custom-preload.strategy.ts
@Injectable({ providedIn: 'root' })
export class SelectivePreloadStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    return route.data?.['preload'] ? load() : of(null);
  }
}

// routes
{ path: 'dashboard', loadComponent: ..., data: { preload: true } }
```

Reference: [Angular Routing Documentation](https://angular.dev/guide/routing)
