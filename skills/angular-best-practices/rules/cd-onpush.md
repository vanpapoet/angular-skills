---
title: Use OnPush Change Detection Strategy
impact: CRITICAL
impactDescription: 2-10x fewer change detection cycles
tags: change-detection, performance, onpush
---

## Use OnPush Change Detection Strategy

**Impact: CRITICAL (2-10x fewer change detection cycles)**

OnPush instructs Angular to run change detection for a component subtree only when it receives new inputs or handles events. This dramatically reduces unnecessary DOM checks in large applications.

**Incorrect (default change detection, checks entire tree):**

```typescript
@Component({
  selector: 'app-user-list',
  template: `<div *ngFor="let user of users">{{ user.name }}</div>`
})
export class UserListComponent {
  @Input() users: User[];
}
```

**Correct (OnPush, skips subtree when inputs unchanged):**

```typescript
import { ChangeDetectionStrategy, Component, Input } from '@angular/core';

@Component({
  selector: 'app-user-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `@for (user of users; track user.id) {
    <div>{{ user.name }}</div>
  }`
})
export class UserListComponent {
  @Input() users: User[];
}
```

**When to use markForCheck():**

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class MyComponent {
  private cdr = inject(ChangeDetectorRef);

  // When mutating @Input objects directly via @ViewChild
  updateFromParent() {
    this.cdr.markForCheck();
  }
}
```

Reference: [Angular OnPush Documentation](https://angular.dev/best-practices/skipping-subtrees)
