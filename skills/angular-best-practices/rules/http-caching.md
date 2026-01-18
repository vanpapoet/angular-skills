---
title: Implement HTTP Caching Strategies
impact: MEDIUM
impactDescription: Reduces network requests, improves response time
tags: http, caching, performance, rxjs
---

## Implement HTTP Caching Strategies

**Impact: MEDIUM (reduces network requests, improves response time)**

Cache HTTP responses appropriately to reduce server load and improve application responsiveness.

**Incorrect (no caching, repeated requests):**

```typescript
@Injectable({ providedIn: 'root' })
export class ConfigService {
  private http = inject(HttpClient);

  // Called multiple times = multiple requests
  getConfig() {
    return this.http.get<Config>('/api/config');
  }
}
```

**Correct (signal-based caching with resource):**

```typescript
import { resource } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class ConfigService {
  private http = inject(HttpClient);

  // Cached with resource API
  config = resource({
    loader: () => this.http.get<Config>('/api/config')
  });
}

// Component usage
@Component({
  template: `
    @if (configService.config.isLoading()) {
      <app-spinner />
    } @else if (configService.config.value(); as config) {
      <div>{{ config.appName }}</div>
    }
  `
})
export class AppComponent {
  configService = inject(ConfigService);
}
```

**RxJS-based caching with shareReplay:**

```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);
  private cache = new Map<string, Observable<User>>();

  getUser(id: string): Observable<User> {
    if (!this.cache.has(id)) {
      this.cache.set(id,
        this.http.get<User>(`/api/users/${id}`).pipe(
          shareReplay({ bufferSize: 1, refCount: true })
        )
      );
    }
    return this.cache.get(id)!;
  }

  invalidateUser(id: string) {
    this.cache.delete(id);
  }

  invalidateAll() {
    this.cache.clear();
  }
}
```

**Time-based cache invalidation:**

```typescript
@Injectable({ providedIn: 'root' })
export class CachedApiService {
  private http = inject(HttpClient);
  private cache = new Map<string, { data: any; timestamp: number }>();
  private ttl = 5 * 60 * 1000; // 5 minutes

  get<T>(url: string): Observable<T> {
    const cached = this.cache.get(url);
    const now = Date.now();

    if (cached && now - cached.timestamp < this.ttl) {
      return of(cached.data as T);
    }

    return this.http.get<T>(url).pipe(
      tap(data => this.cache.set(url, { data, timestamp: now }))
    );
  }
}
```

**Interceptor-based caching:**

```typescript
const cacheInterceptor: HttpInterceptorFn = (req, next) => {
  // Only cache GET requests
  if (req.method !== 'GET') return next(req);

  const cacheKey = req.urlWithParams;
  const cached = requestCache.get(cacheKey);

  if (cached) return of(cached.clone());

  return next(req).pipe(
    tap(response => {
      if (response instanceof HttpResponse) {
        requestCache.set(cacheKey, response.clone());
      }
    })
  );
};
```

Reference: [Angular HTTP Client](https://angular.dev/guide/http)
