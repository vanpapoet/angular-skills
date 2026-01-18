---
title: Use @defer for Lazy Loading Components
impact: CRITICAL
impactDescription: 50-90% reduction in initial bundle size
tags: defer, lazy-loading, bundle-size, performance
---

## Use @defer for Lazy Loading Components

**Impact: CRITICAL (50-90% reduction in initial bundle size)**

The `@defer` block enables lazy loading of components, directives, and pipes by deferring their loading until a specified trigger condition is met.

**Incorrect (eagerly loaded heavy component):**

```typescript
@Component({
  selector: 'app-dashboard',
  imports: [HeavyChartComponent, DataGridComponent],
  template: `
    <app-heavy-chart [data]="chartData" />
    <app-data-grid [rows]="rows" />
  `
})
export class DashboardComponent {}
```

**Correct (deferred loading with triggers):**

```typescript
@Component({
  selector: 'app-dashboard',
  template: `
    @defer (on viewport) {
      <app-heavy-chart [data]="chartData" />
    } @placeholder {
      <div class="chart-skeleton">Loading chart...</div>
    } @loading (minimum 200ms) {
      <app-spinner />
    } @error {
      <p>Failed to load chart. <button (click)="reload()">Retry</button></p>
    }

    @defer (on idle; prefetch on hover) {
      <app-data-grid [rows]="rows" />
    } @placeholder {
      <div class="grid-placeholder">Data grid</div>
    }
  `
})
export class DashboardComponent {}
```

**Available triggers:**

| Trigger | Description |
|---------|-------------|
| `on idle` | When browser reaches idle state (default) |
| `on viewport` | When content enters visible viewport |
| `on interaction` | On user click or keydown |
| `on hover` | On mouseover or focusin |
| `on immediate` | After non-deferred content renders |
| `on timer(500ms)` | After specified duration |
| `when condition` | When boolean expression is true |

**Best Practices:**
- Avoid deferring above-the-fold content (increases CLS)
- Use `prefetch` for perceived performance: `@defer (on interaction; prefetch on idle)`
- Add `@placeholder` with minimum height to prevent layout shift
- Only standalone components/directives/pipes can be deferred

Reference: [Angular @defer Documentation](https://angular.dev/guide/templates/defer)
