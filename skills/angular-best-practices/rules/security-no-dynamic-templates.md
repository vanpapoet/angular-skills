---
title: Never Generate Templates Dynamically
impact: HIGH
impactDescription: Prevents template injection vulnerabilities
tags: security, templates, xss, aot
---

## Never Generate Templates Dynamically

**Impact: HIGH (prevents template injection vulnerabilities)**

Generating Angular templates with user data bypasses all built-in protections. Always use static templates with data binding.

**Incorrect (dynamic template generation):**

```typescript
// DANGEROUS: Server-side template generation
@Controller()
export class PageController {
  @Get('profile/:id')
  async getProfile(@Param('id') id: string) {
    const user = await this.userService.findOne(id);
    // Template injection vulnerability!
    return `
      <app-root>
        <h1>${user.name}</h1>
        <p>${user.bio}</p>
      </app-root>
    `;
  }
}

// DANGEROUS: Runtime template compilation
@Component({
  template: ''
})
export class DynamicComponent {
  constructor(private compiler: Compiler) {}

  createTemplate(userInput: string) {
    // Never do this!
    const template = `<div>${userInput}</div>`;
    const component = Component({ template })(class {});
    // ...
  }
}
```

**Correct (static templates with data binding):**

```typescript
// Safe: Static template with data binding
@Component({
  template: `
    <h1>{{ user().name }}</h1>
    <p>{{ user().bio }}</p>

    <!-- Safe HTML rendering (auto-sanitized) -->
    <div [innerHTML]="user().description"></div>

    <!-- Conditional content -->
    @if (user().isAdmin) {
      <admin-panel />
    }

    <!-- Dynamic components via NgComponentOutlet -->
    <ng-container *ngComponentOutlet="dynamicComponent" />
  `
})
export class ProfileComponent {
  user = input.required<User>();
  dynamicComponent = AdminPanelComponent; // Predefined component
}
```

**For truly dynamic content needs:**

```typescript
// Use predefined component mapping
@Component({
  template: `
    <ng-container *ngComponentOutlet="getComponent()" />
  `
})
export class DynamicContentComponent {
  contentType = input.required<string>();

  private componentMap: Record<string, Type<any>> = {
    'article': ArticleComponent,
    'gallery': GalleryComponent,
    'video': VideoComponent
  };

  getComponent(): Type<any> {
    return this.componentMap[this.contentType()] ?? FallbackComponent;
  }
}
```

**Key principles:**
- Use AOT compilation (default) - prevents runtime template compilation
- Never construct templates from user input
- Use `[innerHTML]` for user HTML (sanitized by Angular)
- Use `*ngComponentOutlet` for dynamic components
- Validate and whitelist dynamic component types

Reference: [Angular Security - AOT](https://angular.dev/best-practices/security)
