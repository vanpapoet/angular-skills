---
title: Use providedIn 'root' for Singleton Services
impact: MEDIUM
impactDescription: Tree-shakable, no module registration needed
tags: dependency-injection, services, singleton, tree-shaking
---

## Use providedIn 'root' for Singleton Services

**Impact: MEDIUM (tree-shakable, no module registration needed)**

Services with `providedIn: 'root'` are tree-shakable singletons that don't require explicit module registration.

**Incorrect (manual provider registration):**

```typescript
// user.service.ts
@Injectable()
export class UserService {
  // ...
}

// app.module.ts or app.config.ts
@NgModule({
  providers: [UserService] // Must register manually
})
export class AppModule {}

// Or in standalone:
bootstrapApplication(AppComponent, {
  providers: [UserService] // Still manual
});
```

**Correct (providedIn: 'root'):**

```typescript
// user.service.ts
@Injectable({ providedIn: 'root' })
export class UserService {
  private http = inject(HttpClient);
  private currentUser = signal<User | null>(null);

  readonly user = this.currentUser.asReadonly();

  getUser(id: string) {
    return this.http.get<User>(`/api/users/${id}`);
  }
}

// No registration needed - Angular provides automatically
// If service is never injected, it's tree-shaken from bundle
```

**When NOT to use providedIn: 'root':**

```typescript
// Scoped to a specific feature (multiple instances needed)
@Injectable()
export class FormStateService {
  // Each component gets its own instance
}

@Component({
  providers: [FormStateService] // Scoped to this component
})
export class FormComponent {
  private formState = inject(FormStateService);
}

// Scoped to a lazy-loaded route
@Injectable({ providedIn: 'any' })
export class FeatureService {
  // New instance per lazy chunk
}
```

**InjectionToken for non-class dependencies:**

```typescript
// config.token.ts
export interface AppConfig {
  apiUrl: string;
  features: string[];
}

export const APP_CONFIG = new InjectionToken<AppConfig>('app.config', {
  providedIn: 'root',
  factory: () => ({
    apiUrl: environment.apiUrl,
    features: ['feature1', 'feature2']
  })
});

// Usage
@Component({ ... })
export class ApiComponent {
  private config = inject(APP_CONFIG);
  private apiUrl = this.config.apiUrl;
}
```

Reference: [Angular Dependency Injection](https://angular.dev/guide/di)
