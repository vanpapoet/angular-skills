---
title: Apply Consistent Selector Prefixes
impact: LOW
impactDescription: Avoids naming conflicts, identifies app-specific elements
tags: style, selectors, naming, components
---

## Apply Consistent Selector Prefixes

**Impact: LOW (avoids naming conflicts, identifies app-specific elements)**

Use a consistent prefix for all component and directive selectors to distinguish your application's elements from third-party libraries.

**Incorrect (no prefix or inconsistent):**

```typescript
@Component({ selector: 'user-card' }) // No prefix
export class UserCardComponent {}

@Component({ selector: 'app-header' }) // app- prefix
export class HeaderComponent {}

@Component({ selector: 'my-footer' }) // my- prefix
export class FooterComponent {}

@Directive({ selector: '[tooltip]' }) // No prefix
export class TooltipDirective {}
```

**Correct (consistent prefix):**

```typescript
// Use app- for main application (or company/project abbreviation)
@Component({ selector: 'app-user-card' })
export class UserCardComponent {}

@Component({ selector: 'app-header' })
export class HeaderComponent {}

@Component({ selector: 'app-footer' })
export class FooterComponent {}

// Directives use camelCase attribute selectors with prefix
@Directive({ selector: '[appTooltip]' })
export class TooltipDirective {}

@Directive({ selector: '[appHighlight]' })
export class HighlightDirective {}
```

**Library/shared module prefixes:**

```typescript
// UI library components might use: ui-
@Component({ selector: 'ui-button' })
export class ButtonComponent {}

@Component({ selector: 'ui-modal' })
export class ModalComponent {}

// Admin feature might use: admin-
@Component({ selector: 'admin-dashboard' })
export class AdminDashboardComponent {}
```

**Configure in angular.json:**

```json
{
  "projects": {
    "my-app": {
      "prefix": "app",
      "schematics": {
        "@schematics/angular:component": {
          "prefix": "app"
        },
        "@schematics/angular:directive": {
          "prefix": "app"
        }
      }
    }
  }
}
```

**ESLint rule to enforce:**

```json
{
  "rules": {
    "@angular-eslint/component-selector": [
      "error",
      { "type": "element", "prefix": "app", "style": "kebab-case" }
    ],
    "@angular-eslint/directive-selector": [
      "error",
      { "type": "attribute", "prefix": "app", "style": "camelCase" }
    ]
  }
}
```

Reference: [Angular Style Guide](https://angular.dev/style-guide)
