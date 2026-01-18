---
title: Use Native Control Flow Syntax
impact: HIGH
impactDescription: Better performance, smaller bundles, improved type inference
tags: control-flow, templates, if, for, switch
---

## Use Native Control Flow Syntax

**Impact: HIGH (better performance, smaller bundles, improved type inference)**

Angular's native control flow (@if, @for, @switch) replaces NgIf, NgFor, and NgSwitch with better performance and type checking.

**Incorrect (legacy structural directives):**

```typescript
@Component({
  imports: [NgIf, NgFor, NgSwitch, NgSwitchCase],
  template: `
    <div *ngIf="user; else noUser">
      <h2>{{ user.name }}</h2>
    </div>
    <ng-template #noUser>
      <p>No user found</p>
    </ng-template>

    <ul>
      <li *ngFor="let item of items; trackBy: trackById">
        {{ item.name }}
      </li>
    </ul>

    <div [ngSwitch]="status">
      <p *ngSwitchCase="'active'">Active</p>
      <p *ngSwitchCase="'inactive'">Inactive</p>
      <p *ngSwitchDefault>Unknown</p>
    </div>
  `
})
export class LegacyComponent {
  trackById(index: number, item: Item) { return item.id; }
}
```

**Correct (native control flow):**

```typescript
@Component({
  template: `
    @if (user; as u) {
      <h2>{{ u.name }}</h2>
    } @else {
      <p>No user found</p>
    }

    <ul>
      @for (item of items; track item.id) {
        <li>{{ item.name }}</li>
      } @empty {
        <li>No items available</li>
      }
    </ul>

    @switch (status) {
      @case ('active') {
        <p>Active</p>
      }
      @case ('inactive') {
        <p>Inactive</p>
      }
      @default {
        <p>Unknown</p>
      }
    }
  `
})
export class ModernComponent {}
```

**Migration:** Use Angular CLI schematic:
```bash
ng generate @angular/core:control-flow
```

Reference: [Angular Control Flow](https://angular.dev/guide/templates/control-flow)
