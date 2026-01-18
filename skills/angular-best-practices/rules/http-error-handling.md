---
title: Centralize Error Handling
impact: MEDIUM
impactDescription: Consistent error handling, better user experience
tags: http, errors, error-handling, interceptors
---

## Centralize Error Handling

**Impact: MEDIUM (consistent error handling, better user experience)**

Handle HTTP errors consistently using interceptors and a centralized error service.

**Incorrect (scattered error handling):**

```typescript
@Component({ ... })
export class UserComponent {
  loadUser(id: string) {
    this.http.get(`/api/users/${id}`).subscribe({
      next: user => this.user.set(user),
      error: err => {
        if (err.status === 404) {
          alert('User not found');
        } else if (err.status === 500) {
          alert('Server error');
        } else {
          alert('Unknown error');
        }
      }
    });
  }
}
// Same error handling copy-pasted everywhere...
```

**Correct (centralized error handling):**

```typescript
// services/error.service.ts
export interface AppError {
  code: string;
  message: string;
  details?: Record<string, any>;
}

@Injectable({ providedIn: 'root' })
export class ErrorService {
  private _error = signal<AppError | null>(null);
  readonly error = this._error.asReadonly();

  handleHttpError(error: HttpErrorResponse): AppError {
    const appError = this.mapHttpError(error);
    this._error.set(appError);
    return appError;
  }

  private mapHttpError(error: HttpErrorResponse): AppError {
    switch (error.status) {
      case 400:
        return { code: 'BAD_REQUEST', message: 'Invalid request', details: error.error };
      case 401:
        return { code: 'UNAUTHORIZED', message: 'Please log in to continue' };
      case 403:
        return { code: 'FORBIDDEN', message: 'You do not have permission' };
      case 404:
        return { code: 'NOT_FOUND', message: 'Resource not found' };
      case 422:
        return { code: 'VALIDATION', message: 'Validation failed', details: error.error };
      case 500:
        return { code: 'SERVER_ERROR', message: 'Something went wrong. Please try again.' };
      default:
        return { code: 'UNKNOWN', message: 'An unexpected error occurred' };
    }
  }

  clearError() {
    this._error.set(null);
  }
}

// interceptors/error.interceptor.ts
export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const errorService = inject(ErrorService);
  const router = inject(Router);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      const appError = errorService.handleHttpError(error);

      // Handle specific cases
      if (error.status === 401) {
        router.navigate(['/login']);
      }

      return throwError(() => appError);
    })
  );
};

// components/error-toast.component.ts
@Component({
  selector: 'app-error-toast',
  template: `
    @if (errorService.error(); as error) {
      <div class="error-toast" role="alert">
        <strong>{{ error.code }}</strong>
        <p>{{ error.message }}</p>
        <button (click)="errorService.clearError()">Dismiss</button>
      </div>
    }
  `
})
export class ErrorToastComponent {
  errorService = inject(ErrorService);
}

// Component usage - clean error handling
@Component({ ... })
export class UserComponent {
  private userService = inject(UserService);
  user = signal<User | null>(null);
  loading = signal(false);

  loadUser(id: string) {
    this.loading.set(true);
    this.userService.getUser(id).pipe(
      finalize(() => this.loading.set(false))
    ).subscribe({
      next: user => this.user.set(user)
      // Error already handled by interceptor
    });
  }
}
```

Reference: [Angular HTTP Client](https://angular.dev/guide/http)
