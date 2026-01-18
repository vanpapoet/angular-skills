---
title: Adopt Zoneless Change Detection
impact: CRITICAL
impactDescription: Eliminates Zone.js overhead, smaller bundles
tags: change-detection, zoneless, performance, angular-21
---

## Adopt Zoneless Change Detection

**Impact: CRITICAL (eliminates Zone.js overhead, smaller bundles)**

Angular v21 introduces zoneless as the default for new applications. Zoneless change detection removes the Zone.js dependency, reducing bundle size and eliminating monkey-patching of browser APIs.

**Incorrect (legacy Zone.js-based detection):**

```typescript
// main.ts - old approach with Zone.js
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent);
```

**Correct (zoneless with explicit change detection):**

```typescript
// main.ts - Angular v21+ zoneless
import { bootstrapApplication } from '@angular/platform-browser';
import { provideExperimentalZonelessChangeDetection } from '@angular/core';
import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [
    provideExperimentalZonelessChangeDetection()
  ]
});
```

**Component with signals (automatic change detection):**

```typescript
@Component({
  selector: 'app-counter',
  template: `<button (click)="increment()">Count: {{ count() }}</button>`
})
export class CounterComponent {
  count = signal(0);

  increment() {
    this.count.update(c => c + 1); // Automatically triggers change detection
  }
}
```

**Migration note:** Use Angular CLI schematics for existing projects:
```bash
ng update @angular/core --migrate-only zoneless
```

Reference: [Angular v21 Zoneless](https://angular.love/angular-21-whats-new)
