---
title: Trust Angular's Automatic Sanitization
impact: MEDIUM
impactDescription: Prevents XSS attacks by default
tags: security, xss, sanitization
---

## Trust Angular's Automatic Sanitization

**Impact: MEDIUM (prevents XSS attacks by default)**

Angular automatically sanitizes untrusted values before inserting them into the DOM. Trust this mechanism and avoid bypassing it unless absolutely necessary.

**Incorrect (attempting manual sanitization):**

```typescript
@Component({
  template: `<div [innerHTML]="sanitizedHtml"></div>`
})
export class ContentComponent {
  userContent = '<script>alert("xss")</script><p>Hello</p>';

  // Unnecessary - Angular already sanitizes
  sanitizedHtml = this.userContent
    .replace(/<script>/g, '')
    .replace(/<\/script>/g, '');
}
```

**Correct (let Angular handle sanitization):**

```typescript
@Component({
  template: `
    <!-- Angular automatically sanitizes innerHTML -->
    <div [innerHTML]="userContent"></div>

    <!-- Interpolation is always escaped -->
    <p>{{ userContent }}</p>

    <!-- URL sanitization -->
    <a [href]="userUrl">Link</a>

    <!-- Style sanitization -->
    <div [style.background]="userBackground">Styled</div>
  `
})
export class ContentComponent {
  // Angular removes script tags and dangerous content
  userContent = '<script>alert("xss")</script><p>Hello</p>';
  // Result: <p>Hello</p>

  // Angular blocks javascript: URLs
  userUrl = 'javascript:alert("xss")';
  // Result: unsafe:javascript:alert("xss")

  userBackground = 'url(javascript:alert("xss"))';
  // Result: sanitized
}
```

**Security contexts Angular recognizes:**

| Context | Example | Sanitization |
|---------|---------|--------------|
| HTML | `[innerHTML]` | Removes scripts, event handlers |
| Style | `[style]` | Blocks dangerous CSS |
| URL | `[href]`, `[src]` | Blocks javascript: URLs |
| Resource URL | `<iframe [src]>` | Requires explicit trust |

**Safe practices:**
- Use interpolation `{{ }}` for user text (always escaped)
- Let `[innerHTML]` be sanitized automatically
- Avoid generating templates with user data
- Use AOT compilation (default)

Reference: [Angular Security Guide](https://angular.dev/best-practices/security)
