---
title: Clean Up Form Subscriptions
impact: MEDIUM
impactDescription: Prevents memory leaks from valueChanges subscriptions
tags: forms, subscriptions, memory-leaks, cleanup
---

## Clean Up Form Subscriptions

**Impact: MEDIUM (prevents memory leaks from valueChanges subscriptions)**

Form observables like `valueChanges` and `statusChanges` don't complete automatically. Always clean up subscriptions to prevent memory leaks.

**Incorrect (subscription leak):**

```typescript
@Component({ ... })
export class SearchComponent {
  form = new FormGroup({
    query: new FormControl('')
  });

  ngOnInit() {
    // Memory leak: subscription never cleaned up
    this.form.controls.query.valueChanges.subscribe(value => {
      this.search(value);
    });
  }
}
```

**Correct (using takeUntilDestroyed):**

```typescript
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

@Component({ ... })
export class SearchComponent {
  private destroyRef = inject(DestroyRef);

  form = new FormGroup({
    query: new FormControl('')
  });

  ngOnInit() {
    this.form.controls.query.valueChanges.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      takeUntilDestroyed(this.destroyRef)
    ).subscribe(value => {
      this.search(value);
    });
  }
}
```

**Using toSignal for reactive forms:**

```typescript
import { toSignal } from '@angular/core/rxjs-interop';

@Component({
  template: `
    <input [formControl]="searchControl" />
    <div>Searching: {{ searchTerm() }}</div>
  `
})
export class SearchComponent {
  searchControl = new FormControl('', { nonNullable: true });

  // Automatically unsubscribes when component is destroyed
  searchTerm = toSignal(
    this.searchControl.valueChanges.pipe(
      debounceTime(300),
      distinctUntilChanged()
    ),
    { initialValue: '' }
  );

  results = computed(() => {
    const term = this.searchTerm();
    // Derive results reactively
  });
}
```

**Alternative patterns:**

```typescript
// Using async pipe (auto-unsubscribe)
@Component({
  template: `
    @if (formStatus$ | async; as status) {
      <span>{{ status }}</span>
    }
  `
})
export class FormComponent {
  formStatus$ = this.form.statusChanges;
}

// Using effect with signals
@Component({ ... })
export class FormComponent {
  searchControl = new FormControl('', { nonNullable: true });
  searchValue = toSignal(this.searchControl.valueChanges, { initialValue: '' });

  constructor() {
    effect(() => {
      const value = this.searchValue();
      // React to form changes
    });
  }
}
```

Reference: [Angular RxJS Interop](https://angular.dev/guide/signals)
