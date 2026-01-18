---
title: Use Standalone Components
impact: HIGH
impactDescription: Better tree-shaking, reduced boilerplate
tags: standalone, components, bundle-size, modern-angular
---

## Use Standalone Components

**Impact: HIGH (better tree-shaking, reduced boilerplate)**

Standalone components are the default in Angular v19+. They eliminate NgModule boilerplate and enable more effective tree-shaking.

**Incorrect (legacy NgModule-based component):**

```typescript
// user.component.ts
@Component({
  selector: 'app-user',
  templateUrl: './user.component.html'
})
export class UserComponent {}

// user.module.ts
@NgModule({
  declarations: [UserComponent],
  imports: [CommonModule, SharedModule],
  exports: [UserComponent]
})
export class UserModule {}
```

**Correct (standalone component):**

```typescript
// user.component.ts
@Component({
  selector: 'app-user',
  standalone: true, // default in v19+, explicit for clarity
  imports: [DatePipe, RouterLink],
  template: `
    <h2>{{ user().name }}</h2>
    <p>Joined: {{ user().joinDate | date }}</p>
    <a routerLink="/profile">View Profile</a>
  `
})
export class UserComponent {
  user = input.required<User>();
}
```

**Import standalone components directly:**

```typescript
@Component({
  selector: 'app-dashboard',
  imports: [UserComponent, StatsComponent, ChartComponent],
  template: `
    <app-user [user]="currentUser()" />
    <app-stats [data]="stats()" />
    <app-chart [series]="chartData()" />
  `
})
export class DashboardComponent {}
```

**Benefits:**
- No NgModule declarations needed
- Direct imports enable better tree-shaking
- Simpler mental model
- Easier lazy loading with `loadComponent`

Reference: [Angular Standalone Components](https://angular.dev/guide/components)
