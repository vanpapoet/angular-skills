---
title: Prefetch Deferred Content Strategically
impact: HIGH
impactDescription: Improves perceived performance by loading ahead of user action
tags: prefetch, defer, performance, ux
---

## Prefetch Deferred Content Strategically

**Impact: HIGH (improves perceived performance by loading ahead of user action)**

Separate prefetch triggers from render triggers to load code before the user needs it, eliminating loading delays when they interact.

**Incorrect (no prefetching, delay on interaction):**

```typescript
@Component({
  template: `
    @defer (on interaction) {
      <app-modal />
    }
  `
})
export class PageComponent {}
// User clicks → loading starts → user waits
```

**Correct (prefetch on idle, show on interaction):**

```typescript
@Component({
  template: `
    @defer (on interaction; prefetch on idle) {
      <app-modal />
    } @placeholder {
      <button>Open Modal</button>
    }
  `
})
export class PageComponent {}
// Code loads during idle → user clicks → instant display
```

**Prefetch on hover for navigation:**

```typescript
@Component({
  template: `
    <nav>
      @defer (on viewport; prefetch on hover) {
        <a routerLink="/dashboard">Dashboard</a>
      }
    </nav>

    @defer (when showDetails; prefetch on timer(2s)) {
      <app-details [data]="selectedItem" />
    }
  `
})
export class NavComponent {}
```

**Prefetch strategies:**

| Strategy | Use Case |
|----------|----------|
| `prefetch on idle` | Background loading when browser is idle |
| `prefetch on hover` | Navigation links, buttons |
| `prefetch on timer(Xms)` | Predictable user flows |
| `prefetch on immediate` | High-priority deferred content |

**Avoid nested @defer cascades:**
```typescript
// Bad: nested defers with same trigger cause cascading loads
@defer (on viewport) {
  @defer (on viewport) { ... }  // Both trigger simultaneously
}

// Good: different triggers prevent cascade
@defer (on viewport) {
  @defer (on interaction) { ... }
}
```

Reference: [Angular @defer Prefetching](https://angular.dev/guide/templates/defer)
