---
title: Use untracked() to Prevent Dependency Tracking
impact: MEDIUM
impactDescription: Prevents unwanted re-computations in computed and effects
tags: signals, untracked, dependencies, optimization
---

## Use untracked() to Prevent Dependency Tracking

**Impact: MEDIUM (prevents unwanted re-computations in computed and effects)**

Wrap signal reads in `untracked()` when you need the value but don't want changes to trigger re-computation.

**Incorrect (unnecessary dependency tracking):**

```typescript
@Component({ ... })
export class SearchComponent {
  searchTerm = signal('');
  config = signal({ debounceMs: 300, minLength: 3 });
  results = signal<Result[]>([]);

  constructor() {
    effect(() => {
      const term = this.searchTerm();
      const cfg = this.config(); // Also tracked - effect runs on config changes

      if (term.length >= cfg.minLength) {
        this.performSearch(term, cfg);
      }
    });
  }
}
```

**Correct (selective tracking with untracked):**

```typescript
import { untracked } from '@angular/core';

@Component({ ... })
export class SearchComponent {
  searchTerm = signal('');
  config = signal({ debounceMs: 300, minLength: 3 });
  results = signal<Result[]>([]);

  constructor() {
    effect(() => {
      const term = this.searchTerm(); // Tracked - effect runs on term change

      // Not tracked - config changes won't trigger effect
      const cfg = untracked(() => this.config());

      if (term.length >= cfg.minLength) {
        this.performSearch(term, cfg);
      }
    });
  }
}
```

**In computed signals:**

```typescript
@Component({ ... })
export class DashboardComponent {
  userId = signal('user-123');
  timestamp = signal(Date.now());
  userData = signal<UserData | null>(null);

  // Only recompute when userData changes, not on every timestamp tick
  lastUpdated = computed(() => {
    const data = this.userData();
    if (!data) return null;

    // Get timestamp without tracking it
    const ts = untracked(() => this.timestamp());
    return `${data.name} - Updated at ${new Date(ts).toLocaleString()}`;
  });
}
```

**Common use cases:**
- Reading configuration that rarely changes
- Accessing services that return signals
- Preventing cascading updates
- Reading "snapshot" values in effects

Reference: [Angular Signals Documentation](https://angular.dev/guide/signals)
