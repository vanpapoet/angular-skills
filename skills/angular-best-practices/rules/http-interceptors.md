---
title: Use Interceptors for Cross-Cutting Concerns
impact: MEDIUM
impactDescription: Centralizes auth, logging, error handling across all requests
tags: http, interceptors, authentication, error-handling
---

## Use Interceptors for Cross-Cutting Concerns

**Impact: MEDIUM (centralizes auth, logging, error handling across all requests)**

Interceptors process requests and responses globally, avoiding repetitive code in individual services.

**Incorrect (repeated logic in every service):**

```typescript
@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);
  private auth = inject(AuthService);

  getUsers() {
    return this.http.get<User[]>('/api/users', {
      headers: { Authorization: `Bearer ${this.auth.getToken()}` }
    }).pipe(
      catchError(err => {
        if (err.status === 401) this.auth.logout();
        throw err;
      })
    );
  }

  // Same auth/error logic repeated in every method...
}
```

**Correct (functional interceptors):**

```typescript
// interceptors/auth.interceptor.ts
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const auth = inject(AuthService);
  const token = auth.getToken();

  if (token) {
    req = req.clone({
      setHeaders: { Authorization: `Bearer ${token}` }
    });
  }

  return next(req);
};

// interceptors/error.interceptor.ts
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const auth = inject(AuthService);
  const router = inject(Router);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        auth.logout();
        router.navigate(['/login']);
      }
      if (error.status === 403) {
        router.navigate(['/forbidden']);
      }
      return throwError(() => error);
    })
  );
};

// interceptors/logging.interceptor.ts
export const loggingInterceptor: HttpInterceptorFn = (req, next) => {
  const started = Date.now();

  return next(req).pipe(
    tap({
      next: () => {
        console.log(`${req.method} ${req.url} - ${Date.now() - started}ms`);
      },
      error: (err) => {
        console.error(`${req.method} ${req.url} - FAILED`, err);
      }
    })
  );
};

// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([
        authInterceptor,
        errorInterceptor,
        loggingInterceptor
      ])
    )
  ]
};
```

**Retry interceptor:**

```typescript
export const retryInterceptor: HttpInterceptorFn = (req, next) => {
  return next(req).pipe(
    retry({
      count: 3,
      delay: (error, retryCount) => {
        if (error.status >= 500) {
          return timer(retryCount * 1000); // Exponential backoff
        }
        throw error; // Don't retry client errors
      }
    })
  );
};
```

Reference: [Angular HTTP Client](https://angular.dev/guide/http)
