---
title: Always Use track in @for Loops
impact: HIGH
impactDescription: Prevents unnecessary DOM recreation, 2-10x faster list updates
tags: for, track, performance, lists
---

## Always Use track in @for Loops

**Impact: HIGH (prevents unnecessary DOM recreation, 2-10x faster list updates)**

The `track` expression maintains relationships between data items and DOM nodes, enabling Angular to reuse existing elements instead of recreating them.

**Incorrect (tracking by index only):**

```typescript
@Component({
  template: `
    @for (user of users; track $index) {
      <app-user-card [user]="user" />
    }
  `
})
export class UserListComponent {}
// When list is sorted/filtered, all cards are destroyed and recreated
```

**Correct (tracking by unique identifier):**

```typescript
@Component({
  template: `
    @for (user of users; track user.id) {
      <app-user-card [user]="user" />
    }
  `
})
export class UserListComponent {}
// Cards are reused and moved, not recreated
```

**Track expression priority:**

1. **Unique ID (best):** `track item.id`
2. **Composite key:** `track item.categoryId + '-' + item.name`
3. **Index (only for static lists):** `track $index`
4. **Identity (avoid):** `track identity` - slowest option

**Access contextual variables:**

```typescript
@for (item of items; track item.id; let i = $index, first = $first, last = $last) {
  <div [class.first]="first" [class.last]="last">
    {{ i + 1 }}. {{ item.name }}
  </div>
}
```

**Available variables:**
- `$index` - current position
- `$first` / `$last` - boundary flags
- `$even` / `$odd` - parity flags
- `$count` - total items

Reference: [Angular @for Documentation](https://angular.dev/guide/templates/control-flow)
