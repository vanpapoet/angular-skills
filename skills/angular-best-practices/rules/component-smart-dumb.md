---
title: Separate Smart and Presentational Components
impact: MEDIUM
impactDescription: Improves reusability, testability, and separation of concerns
tags: components, architecture, smart-dumb, container-presentational
---

## Separate Smart and Presentational Components

**Impact: MEDIUM (improves reusability, testability, and separation of concerns)**

Smart (container) components handle data fetching and state management. Presentational (dumb) components focus on UI rendering with inputs and outputs.

**Incorrect (mixed responsibilities):**

```typescript
@Component({
  selector: 'app-user-list',
  template: `
    @for (user of users(); track user.id) {
      <div class="user-card" (click)="selectUser(user)">
        <img [src]="user.avatar" />
        <h3>{{ user.name }}</h3>
        <button (click)="deleteUser(user.id)">Delete</button>
      </div>
    }
  `
})
export class UserListComponent {
  private userService = inject(UserService);
  users = signal<User[]>([]);

  constructor() {
    this.userService.getUsers().subscribe(u => this.users.set(u));
  }

  selectUser(user: User) { /* navigation logic */ }
  deleteUser(id: string) { /* API call */ }
}
```

**Correct (separated concerns):**

```typescript
// Presentational: user-card.component.ts
@Component({
  selector: 'app-user-card',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="user-card" (click)="selected.emit()">
      <img [src]="user().avatar" />
      <h3>{{ user().name }}</h3>
      <button (click)="deleted.emit(); $event.stopPropagation()">
        Delete
      </button>
    </div>
  `
})
export class UserCardComponent {
  user = input.required<User>();
  selected = output<void>();
  deleted = output<void>();
}

// Smart: user-list.component.ts
@Component({
  selector: 'app-user-list',
  imports: [UserCardComponent],
  template: `
    @for (user of users(); track user.id) {
      <app-user-card
        [user]="user"
        (selected)="onSelect(user)"
        (deleted)="onDelete(user.id)"
      />
    }
  `
})
export class UserListComponent {
  private userService = inject(UserService);
  private router = inject(Router);

  users = toSignal(this.userService.getUsers(), { initialValue: [] });

  onSelect(user: User) {
    this.router.navigate(['/users', user.id]);
  }

  onDelete(id: string) {
    this.userService.delete(id).subscribe();
  }
}
```

**Benefits:**
- Presentational components are easy to test (no DI needed)
- Reusable across different contexts
- Clear data flow (inputs down, outputs up)

Reference: [Angular Style Guide](https://angular.dev/style-guide)
