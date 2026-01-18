---
title: Use InjectionToken for Non-Class Dependencies
impact: LOW
impactDescription: Type-safe injection of primitives, objects, and functions
tags: dependency-injection, injection-token, configuration
---

## Use InjectionToken for Non-Class Dependencies

**Impact: LOW (type-safe injection of primitives, objects, and functions)**

Use `InjectionToken` when injecting values that aren't class instances, such as configuration objects, primitive values, or functions.

**Incorrect (using strings or arbitrary tokens):**

```typescript
// Unclear, not type-safe
providers: [
  { provide: 'API_URL', useValue: 'https://api.example.com' },
  { provide: 'CONFIG', useValue: { timeout: 5000 } }
]

// Component
constructor(@Inject('API_URL') private apiUrl: any) {} // any type!
```

**Correct (typed InjectionToken):**

```typescript
// tokens/api.tokens.ts
export const API_URL = new InjectionToken<string>('API URL', {
  providedIn: 'root',
  factory: () => environment.apiUrl
});

export interface ApiConfig {
  timeout: number;
  retries: number;
  baseHeaders: Record<string, string>;
}

export const API_CONFIG = new InjectionToken<ApiConfig>('API Configuration', {
  providedIn: 'root',
  factory: () => ({
    timeout: 5000,
    retries: 3,
    baseHeaders: { 'X-App-Version': '1.0.0' }
  })
});

// Usage with full type safety
@Injectable({ providedIn: 'root' })
export class ApiService {
  private apiUrl = inject(API_URL);
  private config = inject(API_CONFIG);

  request(endpoint: string) {
    return this.http.get(`${this.apiUrl}${endpoint}`, {
      headers: this.config.baseHeaders // Typed!
    }).pipe(
      timeout(this.config.timeout),
      retry(this.config.retries)
    );
  }
}
```

**Function injection:**

```typescript
export type LoggerFn = (message: string, ...args: any[]) => void;

export const LOGGER = new InjectionToken<LoggerFn>('Logger function', {
  providedIn: 'root',
  factory: () => {
    const isProduction = inject(IS_PRODUCTION);
    return isProduction
      ? () => {} // No-op in production
      : console.log.bind(console);
  }
});

// Usage
@Component({ ... })
export class DebugComponent {
  private log = inject(LOGGER);

  ngOnInit() {
    this.log('Component initialized', { timestamp: Date.now() });
  }
}
```

**Override in tests:**

```typescript
TestBed.configureTestingModule({
  providers: [
    { provide: API_URL, useValue: 'http://localhost:3000' },
    { provide: LOGGER, useValue: jasmine.createSpy('logger') }
  ]
});
```

Reference: [Angular Dependency Injection](https://angular.dev/guide/di)
