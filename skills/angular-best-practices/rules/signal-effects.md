---
title: Use effect() Sparingly for Side Effects
impact: MEDIUM
impactDescription: Handles side effects reactively, but can cause unexpected behavior
tags: signals, effects, side-effects, reactivity
---

## Use effect() Sparingly for Side Effects

**Impact: MEDIUM (handles side effects reactively, but can cause unexpected behavior)**

Effects run when their signal dependencies change. Use them for side effects like logging, syncing with external systems, or localStorage. Avoid for state updates.

**Incorrect (using effect for state updates):**

```typescript
@Component({ ... })
export class CartComponent {
  items = signal<CartItem[]>([]);
  total = signal(0);

  constructor() {
    // Bad: using effect to update state
    effect(() => {
      const sum = this.items().reduce((s, i) => s + i.price, 0);
      this.total.set(sum); // Causes additional effect runs
    });
  }
}
```

**Correct (computed for derived state, effect for side effects):**

```typescript
@Component({ ... })
export class CartComponent {
  items = signal<CartItem[]>([]);

  // Use computed for derived state
  total = computed(() =>
    this.items().reduce((s, i) => s + i.price, 0)
  );

  constructor() {
    // Effect for side effects only
    effect(() => {
      // Sync to localStorage
      localStorage.setItem('cart', JSON.stringify(this.items()));
    });

    effect(() => {
      // Analytics tracking
      this.analytics.track('cart_total_changed', { total: this.total() });
    });
  }
}
```

**Effect cleanup:**

```typescript
constructor() {
  effect((onCleanup) => {
    const subscription = this.websocket.connect(this.userId());

    onCleanup(() => {
      subscription.unsubscribe();
    });
  });
}
```

**Use untracked() to prevent unwanted tracking:**

```typescript
effect(() => {
  // Only react to searchTerm changes
  const term = this.searchTerm();

  // Don't track this read
  const config = untracked(() => this.config());

  this.search(term, config);
});
```

**When to use effect:**
- Logging/analytics
- localStorage/sessionStorage sync
- External API calls (with debouncing)
- DOM manipulation outside Angular

**Avoid effect for:**
- Deriving state (use `computed`)
- Setting other signals (creates chains)
- Complex business logic (use services)

Reference: [Angular Signals Documentation](https://angular.dev/guide/signals)
