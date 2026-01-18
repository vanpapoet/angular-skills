---
title: Use Kebab-Case for File Names
impact: LOW
impactDescription: Consistent, cross-platform compatible naming
tags: style, naming, file-organization
---

## Use Kebab-Case for File Names

**Impact: LOW (consistent, cross-platform compatible naming)**

Use lowercase with hyphens for all file names. This ensures consistency and avoids case-sensitivity issues across different operating systems.

**Incorrect (inconsistent naming):**

```
src/
├── UserProfile.component.ts
├── userCard.component.ts
├── User_Service.ts
├── auth.Module.ts
└── API_CONSTANTS.ts
```

**Correct (kebab-case):**

```
src/
├── users/
│   ├── user-profile.component.ts
│   ├── user-profile.component.html
│   ├── user-profile.component.css
│   ├── user-profile.component.spec.ts
│   ├── user-card.component.ts
│   └── user.service.ts
├── auth/
│   ├── auth.guard.ts
│   ├── auth.interceptor.ts
│   └── auth.service.ts
├── shared/
│   ├── pipes/
│   │   └── date-format.pipe.ts
│   └── directives/
│       └── tooltip.directive.ts
└── constants/
    └── api.constants.ts
```

**File naming patterns:**

| Type | Pattern | Example |
|------|---------|---------|
| Component | `feature-name.component.ts` | `user-profile.component.ts` |
| Service | `feature-name.service.ts` | `auth.service.ts` |
| Directive | `directive-name.directive.ts` | `tooltip.directive.ts` |
| Pipe | `pipe-name.pipe.ts` | `date-format.pipe.ts` |
| Guard | `guard-name.guard.ts` | `auth.guard.ts` |
| Interceptor | `interceptor-name.interceptor.ts` | `error.interceptor.ts` |
| Model/Interface | `model-name.model.ts` | `user.model.ts` |
| Test | `*.spec.ts` | `user.service.spec.ts` |
| Routes | `feature.routes.ts` | `admin.routes.ts` |

**Match filename to exported identifier:**

```typescript
// user-profile.component.ts
@Component({ selector: 'app-user-profile' })
export class UserProfileComponent {}

// auth.service.ts
@Injectable({ providedIn: 'root' })
export class AuthService {}

// date-format.pipe.ts
@Pipe({ name: 'dateFormat' })
export class DateFormatPipe {}
```

Reference: [Angular Style Guide](https://angular.dev/style-guide)
