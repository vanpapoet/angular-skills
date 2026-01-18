---
title: Implement Lifecycle Interfaces
impact: LOW
impactDescription: Ensures correct method names, improves documentation
tags: style, lifecycle, interfaces, typescript
---

## Implement Lifecycle Interfaces

**Impact: LOW (ensures correct method names, improves documentation)**

Explicitly implement lifecycle interfaces to ensure correct method signatures and improve code clarity.

**Incorrect (no interface, typo risk):**

```typescript
@Component({ ... })
export class UserComponent {
  // Typo: 'ngOnint' instead of 'ngOnInit' - no error!
  ngOnint() {
    this.loadUser();
  }

  // Wrong signature - no compile-time error
  ngOnChanges() {
    console.log('changed');
  }

  ngOnDestroy() {
    // cleanup
  }
}
```

**Correct (explicit interfaces):**

```typescript
import {
  Component,
  OnInit,
  OnChanges,
  OnDestroy,
  SimpleChanges,
  input,
  inject
} from '@angular/core';

@Component({ ... })
export class UserComponent implements OnInit, OnChanges, OnDestroy {
  userId = input.required<string>();
  private userService = inject(UserService);
  private destroyRef = inject(DestroyRef);

  // TypeScript enforces correct signature
  ngOnInit(): void {
    this.loadUser();
  }

  // TypeScript enforces SimpleChanges parameter
  ngOnChanges(changes: SimpleChanges): void {
    if (changes['userId']) {
      this.loadUser();
    }
  }

  ngOnDestroy(): void {
    // TypeScript ensures this exists when OnDestroy is implemented
    console.log('Component destroyed');
  }

  private loadUser() {
    this.userService.getUser(this.userId())
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe();
  }
}
```

**All lifecycle interfaces:**

| Interface | Method | When Called |
|-----------|--------|-------------|
| `OnInit` | `ngOnInit()` | After first `ngOnChanges` |
| `OnChanges` | `ngOnChanges(changes)` | When inputs change |
| `OnDestroy` | `ngOnDestroy()` | Before component destroyed |
| `AfterViewInit` | `ngAfterViewInit()` | After view initialized |
| `AfterViewChecked` | `ngAfterViewChecked()` | After every view check |
| `AfterContentInit` | `ngAfterContentInit()` | After content projected |
| `AfterContentChecked` | `ngAfterContentChecked()` | After every content check |
| `DoCheck` | `ngDoCheck()` | During every CD cycle |

**Modern alternative with DestroyRef:**

```typescript
@Component({ ... })
export class ModernComponent {
  private destroyRef = inject(DestroyRef);

  constructor() {
    // Register cleanup without OnDestroy interface
    this.destroyRef.onDestroy(() => {
      console.log('Cleanup');
    });
  }
}
```

Reference: [Angular Style Guide](https://angular.dev/style-guide)
