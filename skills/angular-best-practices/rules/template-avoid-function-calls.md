---
title: Avoid Function Calls in Templates
impact: HIGH
impactDescription: Prevents repeated expensive computations on every change detection
tags: templates, performance, computed, functions
---

## Avoid Function Calls in Templates

**Impact: HIGH (prevents repeated expensive computations on every change detection)**

Functions called in templates execute on every change detection cycle. Use computed signals or pure pipes for derived values instead.

**Incorrect (function call in template):**

```typescript
@Component({
  template: `
    <div>{{ getFullName() }}</div>
    <div>{{ calculateTotal() }}</div>
    <div>{{ formatDate(user.createdAt) }}</div>
    <div [class.highlight]="isHighlighted(item)">{{ item.name }}</div>
  `
})
export class UserComponent {
  user = input.required<User>();

  getFullName() {
    console.log('Called!'); // Logs on every CD cycle
    return `${this.user().firstName} ${this.user().lastName}`;
  }

  calculateTotal() {
    return this.items.reduce((sum, i) => sum + i.price, 0);
  }
}
```

**Correct (computed signals for derived state):**

```typescript
@Component({
  template: `
    <div>{{ fullName() }}</div>
    <div>{{ total() }}</div>
    <div>{{ user().createdAt | date }}</div>
    <div [class.highlight]="highlightedIds().has(item.id)">{{ item.name }}</div>
  `
})
export class UserComponent {
  user = input.required<User>();
  items = input.required<Item[]>();

  fullName = computed(() =>
    `${this.user().firstName} ${this.user().lastName}`
  );

  total = computed(() =>
    this.items().reduce((sum, i) => sum + i.price, 0)
  );

  highlightedIds = computed(() =>
    new Set(this.items().filter(i => i.highlighted).map(i => i.id))
  );
}
```

**Use pure pipes for formatting:**

```typescript
// date-format.pipe.ts
@Pipe({ name: 'dateFormat', pure: true })
export class DateFormatPipe implements PipeTransform {
  transform(value: Date, format: string): string {
    return formatDate(value, format);
  }
}

// Usage: {{ user.createdAt | dateFormat:'short' }}
```

Reference: [Angular Performance](https://angular.dev/best-practices/runtime-performance)
