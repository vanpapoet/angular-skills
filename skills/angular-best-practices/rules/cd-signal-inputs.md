---
title: Use Signal-Based Inputs
impact: CRITICAL
impactDescription: Fine-grained reactivity, automatic change detection
tags: signals, inputs, change-detection, angular-21
---

## Use Signal-Based Inputs

**Impact: CRITICAL (fine-grained reactivity, automatic change detection)**

Signal-based inputs provide automatic dependency tracking in OnPush components. When read in templates, Angular tracks them as dependencies and updates only when values change.

**Incorrect (traditional @Input decorator):**

```typescript
@Component({
  selector: 'app-user-card',
  template: `<h2>{{ user.name }}</h2>`
})
export class UserCardComponent {
  @Input() user!: User;
}
```

**Correct (signal input with automatic tracking):**

```typescript
import { Component, input } from '@angular/core';

@Component({
  selector: 'app-user-card',
  template: `<h2>{{ user().name }}</h2>`
})
export class UserCardComponent {
  user = input.required<User>();

  // Computed values derived from input
  displayName = computed(() => `${this.user().firstName} ${this.user().lastName}`);
}
```

**With default value:**

```typescript
@Component({
  selector: 'app-settings',
  template: `<div [class.dark]="darkMode()">...</div>`
})
export class SettingsComponent {
  darkMode = input(false); // Default value
  theme = input<'light' | 'dark'>('light'); // Typed with default
}
```

**Transform input values:**

```typescript
@Component({
  selector: 'app-item',
  template: `<span>{{ price() | currency }}</span>`
})
export class ItemComponent {
  price = input(0, { transform: numberAttribute }); // Coerce to number
}
```

Reference: [Angular Signals Documentation](https://angular.dev/guide/signals)
