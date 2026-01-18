---
title: Use Pure Pipes for Transformations
impact: MEDIUM
impactDescription: Automatic memoization, runs only when inputs change
tags: pipes, pure, performance, memoization
---

## Use Pure Pipes for Transformations

**Impact: MEDIUM (automatic memoization, runs only when inputs change)**

Pure pipes (default) are memoized - they only re-execute when their input reference changes. Use them for data transformations in templates.

**Incorrect (impure pipe for simple transformation):**

```typescript
@Pipe({ name: 'filter', pure: false }) // Runs every CD cycle!
export class FilterPipe implements PipeTransform {
  transform(items: Item[], searchTerm: string): Item[] {
    return items.filter(i => i.name.includes(searchTerm));
  }
}
```

**Correct (pure pipe with immutable updates):**

```typescript
@Pipe({ name: 'filter', pure: true }) // Default, explicit for clarity
export class FilterPipe implements PipeTransform {
  transform(items: Item[], searchTerm: string): Item[] {
    if (!searchTerm) return items;
    return items.filter(i =>
      i.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }
}

// Component must update array reference for pipe to re-run
@Component({
  imports: [FilterPipe],
  template: `
    @for (item of items() | filter:searchTerm(); track item.id) {
      <div>{{ item.name }}</div>
    }
  `
})
export class ListComponent {
  items = signal<Item[]>([]);
  searchTerm = signal('');

  updateItems(newItems: Item[]) {
    this.items.set([...newItems]); // New reference triggers pipe
  }
}
```

**Common pure pipe use cases:**

```typescript
// Currency formatting
{{ price | currency:'USD' }}

// Date formatting
{{ createdAt | date:'mediumDate' }}

// Async observable unwrapping
{{ user$ | async }}

// JSON debugging
{{ debugData | json }}

// Custom transformations
{{ text | truncate:50 }}
```

**When to use impure pipes:**
- Filtering/sorting arrays that mutate (prefer signals + computed instead)
- Real-time data that changes frequently without reference change

Reference: [Angular Pipes Documentation](https://angular.dev/guide/pipes)
