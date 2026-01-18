---
title: Prefer Standalone Components
impact: HIGH
impactDescription: Simpler architecture, better tree-shaking
tags: components, standalone, architecture, modern-angular
---

## Prefer Standalone Components

**Impact: HIGH (simpler architecture, better tree-shaking)**

Standalone components are the default in Angular v19+. They provide a simpler mental model and enable better optimization by the compiler.

**Incorrect (legacy NgModule pattern):**

```typescript
// shared.module.ts
@NgModule({
  declarations: [ButtonComponent, CardComponent, ModalComponent],
  imports: [CommonModule],
  exports: [ButtonComponent, CardComponent, ModalComponent]
})
export class SharedModule {}

// feature.module.ts
@NgModule({
  declarations: [FeatureComponent],
  imports: [SharedModule] // Imports everything, even unused
})
export class FeatureModule {}
```

**Correct (standalone with direct imports):**

```typescript
// button.component.ts
@Component({
  selector: 'app-button',
  standalone: true,
  template: `<button [type]="type()"><ng-content /></button>`
})
export class ButtonComponent {
  type = input<'button' | 'submit'>('button');
}

// feature.component.ts
@Component({
  selector: 'app-feature',
  standalone: true,
  imports: [ButtonComponent], // Only what's needed
  template: `
    <app-button type="submit">Save</app-button>
  `
})
export class FeatureComponent {}
```

**Bootstrapping standalone:**

```typescript
// main.ts
import { bootstrapApplication } from '@angular/platform-browser';
import { AppComponent } from './app/app.component';
import { appConfig } from './app/app.config';

bootstrapApplication(AppComponent, appConfig);

// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient()
  ]
};
```

Reference: [Angular Standalone Components](https://angular.dev/guide/components)
