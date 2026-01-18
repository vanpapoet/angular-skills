---
title: Use computed() for Derived State
impact: HIGH
impactDescription: Automatic memoization, lazy evaluation, dependency tracking
tags: signals, computed, derived-state, memoization
---

## Use computed() for Derived State

**Impact: HIGH (automatic memoization, lazy evaluation, dependency tracking)**

Computed signals automatically track dependencies and only recalculate when those dependencies change. They're lazily evaluated and memoized.

**Incorrect (manual derivation in getter):**

```typescript
@Component({
  template: `<div>{{ fullName }}</div><div>{{ isAdmin }}</div>`
})
export class UserComponent {
  firstName = signal('John');
  lastName = signal('Doe');
  role = signal('admin');

  // Called on every change detection cycle
  get fullName() {
    console.log('Computing full name'); // Logs repeatedly
    return `${this.firstName()} ${this.lastName()}`;
  }

  get isAdmin() {
    return this.role() === 'admin';
  }
}
```

**Correct (computed signals):**

```typescript
@Component({
  template: `<div>{{ fullName() }}</div><div>{{ isAdmin() }}</div>`
})
export class UserComponent {
  firstName = signal('John');
  lastName = signal('Doe');
  role = signal('admin');

  // Only recalculates when firstName or lastName changes
  fullName = computed(() => {
    console.log('Computing full name'); // Logs only when deps change
    return `${this.firstName()} ${this.lastName()}`;
  });

  isAdmin = computed(() => this.role() === 'admin');

  // Derived from other computed signals
  displayName = computed(() =>
    this.isAdmin() ? `Admin: ${this.fullName()}` : this.fullName()
  );
}
```

**Complex computed with multiple dependencies:**

```typescript
@Component({
  template: `
    <div>Total: {{ orderTotal() | currency }}</div>
    <div>Items: {{ itemCount() }}</div>
  `
})
export class CartComponent {
  items = signal<CartItem[]>([]);
  discount = signal(0);
  taxRate = signal(0.1);

  subtotal = computed(() =>
    this.items().reduce((sum, item) => sum + item.price * item.quantity, 0)
  );

  itemCount = computed(() =>
    this.items().reduce((count, item) => count + item.quantity, 0)
  );

  orderTotal = computed(() => {
    const sub = this.subtotal();
    const discounted = sub * (1 - this.discount());
    return discounted * (1 + this.taxRate());
  });
}
```

Reference: [Angular Signals Documentation](https://angular.dev/guide/signals)
