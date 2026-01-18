---
title: One Component Per File
impact: MEDIUM
impactDescription: Improves maintainability, testability, and discoverability
tags: components, organization, single-responsibility
---

## One Component Per File

**Impact: MEDIUM (improves maintainability, testability, and discoverability)**

Each file should contain one component, directive, pipe, or service. This makes code easier to find, test, and maintain.

**Incorrect (multiple components in one file):**

```typescript
// user.component.ts
@Component({
  selector: 'app-user-avatar',
  template: `<img [src]="user().avatar" />`
})
export class UserAvatarComponent {
  user = input.required<User>();
}

@Component({
  selector: 'app-user-name',
  template: `<span>{{ user().name }}</span>`
})
export class UserNameComponent {
  user = input.required<User>();
}

@Component({
  selector: 'app-user-card',
  imports: [UserAvatarComponent, UserNameComponent],
  template: `
    <app-user-avatar [user]="user()" />
    <app-user-name [user]="user()" />
  `
})
export class UserCardComponent {
  user = input.required<User>();
}
```

**Correct (one component per file):**

```typescript
// user-avatar.component.ts
@Component({
  selector: 'app-user-avatar',
  template: `<img [src]="user().avatar" [alt]="user().name" />`
})
export class UserAvatarComponent {
  user = input.required<User>();
}

// user-name.component.ts
@Component({
  selector: 'app-user-name',
  template: `<span>{{ user().name }}</span>`
})
export class UserNameComponent {
  user = input.required<User>();
}

// user-card.component.ts
@Component({
  selector: 'app-user-card',
  imports: [UserAvatarComponent, UserNameComponent],
  template: `
    <app-user-avatar [user]="user()" />
    <app-user-name [user]="user()" />
  `
})
export class UserCardComponent {
  user = input.required<User>();
}
```

**File organization:**

```
users/
├── user-card.component.ts
├── user-card.component.html      # Optional: external template
├── user-card.component.css       # Optional: external styles
├── user-card.component.spec.ts
├── user-avatar.component.ts
├── user-name.component.ts
└── user.model.ts
```

Reference: [Angular Style Guide](https://angular.dev/style-guide)
